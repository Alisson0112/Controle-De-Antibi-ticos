# Sistema de Controle de Antibióticos — Resumo do projeto

Atualizado em 23/10/2026. Este arquivo resume tudo o que foi feito na conversa, as regras do sistema e o que ainda está pendente. Serve para guardar, para passar a outra pessoa ou para retomar o trabalho em outra conversa.

---

## 1. O que é o sistema

- Sistema web de **controle de uso de antibióticos** por paciente e por setor, com **controle de estoque** e **relatórios**. Usado pelos **4 farmacêuticos** da Farmácia.
- É **um único arquivo** (`index.html`, com HTML, CSS e JavaScript), sem etapa de build.
- Os dados ficam no **Firebase** (projeto `controle-de-antibiotico`): Firestore (coleções de registros, estoque e configurações) e Authentication (e-mail e senha). **A hospedagem só guarda o arquivo; nenhum dado fica nela.**
- Abas: **Registros**, **Estoque**, **Protocolo de Infusão**, **Relatórios**, **Configurações** (nome do sistema, backup e restauração, padronização de medicamentos).
- Nome padrão no código: **AC - Sistemas e Soluções**. O nome exibido no cabeçalho vem do banco de dados: para mudar, use Configurações → Nome do Hospital → Salvar Configurações.

## 2. Arquivos deste pacote

| Arquivo | O que é |
|---|---|
| `index.html` | Versão atual do sistema web (23/10/2026). É o que deve ser publicado. |
| `controle_antibioticos.xlsx` | Versão em Excel (Registro, Estoque, Relatórios, Listas), feita em agosto. **Está desatualizada em relação ao sistema web** (veja a seção 7). |
| `RESUMO_DO_PROJETO.md` | Este resumo. |

## 3. Como publicar e atualizar

**Hospedagem atual: Cloudflare (Workers/Pages), gratuita e sem sistema de créditos.** Antes estava no Netlify, que passou a cobrar créditos por publicação.

1. Em dash.cloudflare.com → **Workers e Pages** → **Criar** → envio direto de arquivos (arrastar e soltar).
2. Nome do projeto usado antes: `controledeantibioticoshne`. Endereço antigo: `controledeantibioticoshne.planilhahne.workers.dev`.
3. Arraste o `index.html` e confirme. Para atualizar depois: **Nova implantação** e arrastar o arquivo novo.
4. Se o navegador mostrar a versão antiga, use **Ctrl+F5**.
5. **Se o endereço mudar**, cadastre o novo no Firebase: console do Firebase → Authentication → Configurações → **Domínios autorizados** → Adicionar domínio (sem `https://`). Sem isso o login dá erro.

> O projeto do Cloudflare foi **excluído por engano** e precisa ser publicado de novo. Nenhum dado foi perdido. O site antigo no Netlify (`controledeantibioticoshne.netlify.app`) pode continuar no ar enquanto houver créditos.

**Guarde sempre uma cópia do `index.html` em uma pasta sua (Drive, OneDrive).** Faça também o backup dos dados em Configurações → Backup (arquivo .json), toda semana.

## 4. Regras de negócio importantes

**Campo Medicamento**
- Lista fixa de 32 antibióticos (`MEDICAMENTOS_LISTA`), em campo pesquisável que **bloqueia texto livre**. Isso evita nomes escritos de formas diferentes.
- Registros antigos com nomes fora da lista continuam aparecendo ao editar. A ferramenta **Configurações → Padronização de Medicamentos** troca os nomes antigos pelos da lista em lote (depois de revisão manual, em blocos de até 400).

**Casamento de nomes com a aba Estoque**
- **Desde 30/10/2026:** cada item da lista fixa tem um vínculo direto com o código da linha do Estoque (`MED_STOCK_CODE`). A comparação por nome abaixo só vale como reserva para registros antigos fora da lista. Ao incluir um medicamento novo na lista, inclua também o código dele em `MED_STOCK_CODE`.
- `normalizeMedName` tira acentos, passa para maiúsculas e remove espaços e pontuação. A comparação é por "contém", nos dois sentidos.
- Nomes sem correspondência, ou com mais de uma, ficam de fora da soma e aparecem no aviso amarelo "Não reconhecidos no Estoque".

**Estoque: "Tratamento em Andamento" e "Disponível"**
- Soma, para cada registro com status **Ativo**, a **quantidade que ainda falta usar** = dose diária × dias restantes até o término.
  - Término = início + duração.
  - Tratamento que ainda não começou reserva a duração inteira; tratamento vencido conta 0.
- **Disponível = Estoque − Tratamento em Andamento.**
- Recalcula a cada atualização do banco e a cada 15 minutos.
- Evolução da regra: primeiro somava a dose diária; depois o total do tratamento; por fim a quantidade restante. Um erro de "1 dia a mais" foi corrigido e conferido contra os relatórios reais da farmácia.

**Finalização automática**
- Registros "Ativo" com término já passado viram "Finalizado" **quando alguém abre o sistema** (não há rotina no servidor). Por isso os farmacêuticos devem mudar o status manualmente quando o tratamento acabar ou for suspenso.

**Aba Relatórios**
- Período **Diário, Mensal ou Anual**, com setas para avançar e voltar.
- Filtros de **Setor** e de **Antibióticos** com seleção múltipla (busca e "Selecionar tudo").
- **Contar consumo por**:
  - *Início do tratamento* (padrão, critério original): o total do tratamento conta na data de início.
  - *Dias de uso*: a dose diária conta em cada dia de uso, até hoje (tratamentos suspensos contam até o término previsto).
- Mostra 5 indicadores, a tabela por antibiótico, a matriz **antibiótico × setor** (com totais, % e pacientes), um gráfico de evolução e o comparativo entre meses (que também segue os filtros).
- Impressão e exportação (CSV) respeitam os filtros.

**Impressão**
- Os seletores de impressão de Registros, Estoque e Relatórios têm a opção **"Escolher ao imprimir"**, que faz o Chrome mostrar Retrato/Paisagem na própria janela. Nos Relatórios ela vem selecionada por padrão.

## 5. Histórico do que foi feito (em ordem)

1. Campo Medicamento virou lista fixa pesquisável, com bloqueio de texto livre.
2. Ferramenta de padronização em lote dos registros antigos (Configurações).
3. Correção da comparação de nomes (espaços, pontuação), que fazia itens como Benzilpenicilina aparecerem como "não reconhecidos".
4. Migração do Netlify (créditos) para o Cloudflare.
5. Versão em Excel (Registro, Estoque, Relatórios), depois redesenhada com visual de sistema.
6. Estoque: da dose diária para o total e depois para a quantidade restante, com correção do "+1 dia".
7. Filtro de colunas da tabela de Registros: duas correções (o painel fechava ao rolar e ao arrastar a barra de rolagem).
8. Aba Relatórios reformulada (período, setor, antibióticos, matriz por setor, gráfico, impressão e Excel).
9. Troca do nome "Hospital Nova Esperança" por "AC - Sistemas e Soluções" no código.
10. Impressão: opção "Escolher ao imprimir".
11. Projeto do Cloudflare excluído por engano; é preciso republicar.

## 6. Mapa do código (`index.html`, linhas aproximadas)

| Assunto | Função |
|---|---|
| Filtros de coluna dos Registros | `setupColumnFilters` (~1243) |
| Quantidade restante de um tratamento | `calculateRemainingQuantity` (~1540) |
| Finalização automática | `autoFinalizeExpiredTreatments` (~1562) |
| Regra de página da impressão | `buildPageRule` (~1835) |
| Campo Medicamento (lista fixa) | `setupMedicamentoCombobox` (~2068) |
| Padronização em lote | `setupMedMigration` (~2179) |
| Normalização de nomes | `normalizeMedName` (~2294) |
| Cálculo do Estoque | `recomputeStockTratamento` (~2311) |
| Matriz antibiótico × setor | `buildSectorMatrix` (~2930) |
| Listas de seleção múltipla | `createMultiSelect` (~3001) |
| Impressão e exportação dos Relatórios | `printReport` (~3368), `exportReport` (~3502) |

## 7. Pontos de atenção e pendências

1. **Republicar o site** (seção 3) e conferir os domínios autorizados no Firebase.
2. **Endereço desejado `controledeantibioticos.workers.dev`:** no Cloudflare Workers o endereço sempre tem o formato `nome-do-projeto.subdominio-da-conta.workers.dev`, então um endereço só com `nome.workers.dev` não é possível. Opções:
   - escolher o nome do projeto e trocar o subdomínio da conta (Workers e Pages → "Your subdomain" → Change; **afeta todos os projetos da conta**);
   - usar o Cloudflare Pages (`nome.pages.dev`; o envio direto ainda existe, mas a Cloudflare recomenda Workers para projetos novos);
   - comprar um **domínio próprio** (pago) e ligá-lo ao projeto.
3. **Planilha Excel desatualizada:** ela soma a **dose diária** dos registros ativos no Estoque e não tem os relatórios novos nem a quantidade restante. Se for usada, precisa ser atualizada.
4. **Datas digitadas erradas** distorcem o Estoque (já houve um registro com o mês de início trocado). Vale conferir as datas de início dos registros.
5. **Rodapé** do sistema: "Sistema de Controle de Antibióticos - AC - Sistemas e Soluções © 2025" (hífen duplo e ano antigo). Fácil de ajustar.
6. **Nomes parecidos** (por exemplo Levofloxacino 500 MG e 500 MG/100 ML) podem casar com mais de uma linha do Estoque. Quando isso ocorre, o sistema avisa em vez de somar errado.
7. **Testes:** a lógica foi testada por simulação (cálculos, filtros, impressão e exportação). A aparência foi conferida pelos prints enviados. Depois de cada publicação, vale olhar a tela.

## 8. Texto para retomar em outra conversa

> Tenho um sistema web de controle de antibióticos (arquivo único `index.html`, com Firebase Firestore/Auth, hospedado no Cloudflare). Tem abas de Registros, Estoque, Protocolo de Infusão, Relatórios e Configurações. O campo Medicamento usa uma lista fixa de 32 itens. O "Tratamento em Andamento" do Estoque soma a quantidade que ainda falta usar (dose diária × dias restantes). Os Relatórios têm período diário/mensal/anual, filtros de setor e antibióticos, matriz antibiótico × setor, impressão e CSV. Segue o `index.html` atual e o resumo do projeto. Quero continuar de onde paramos.
