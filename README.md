# Sistema de Controle de Antibióticos

Sistema web de controle de uso de antibióticos por paciente e por setor, com controle de estoque e relatórios, usado pelos farmacêuticos da Farmácia.

**Endereço oficial:** https://controledeantibioticos.pages.dev (Cloudflare Pages)

## Estrutura

| Caminho | Descrição |
|---|---|
| `public/index.html` | O sistema completo (HTML + CSS + JavaScript em um único arquivo, sem build). **Só a pasta `public/` é publicada.** |
| `public/_headers` | Cabeçalhos de segurança e regra de cache (o `index.html` é sempre revalidado). |
| `redirect-enderecos-antigos/` | Arquivos que redirecionam os endereços antigos (Netlify e Workers) para o oficial. Não são publicados pelo Pages. |
| `docs/RESUMO_DO_PROJETO.md` | Regras de negócio, histórico de alterações, mapa do código e pendências. |
| `planilha/controle_antibioticos.xlsx` | Versão em Excel (agosto/2026). Desatualizada em relação ao sistema web. |

## Arquitetura

- **Front-end:** `public/index.html` estático.
- **Banco de dados e login:** Firebase, projeto `controle-de-antibiotico`. Coleções do Firestore: `records` (registros), `stock` (estoque) e `config/system` (configurações). Autenticação por e-mail e senha. Nenhum dado fica na hospedagem nem neste repositório.
- **Hospedagem:** Cloudflare Pages ligado a este repositório. Cada push na branch `main` publica automaticamente.

> A configuração web do Firebase no `index.html` é pública por design. Quem protege os dados são as **regras de segurança do Firestore**, que devem exigir usuário autenticado. **Nunca** faça commit de backups `.json`, porque eles contêm dados de pacientes.

## Configuração do Cloudflare Pages (feita uma vez)

1. Cloudflare → **Workers e Pages** → **Criar aplicação** → **Pages** → **Conectar ao Git**.
2. Autorize o GitHub e escolha `Alisson0112/Controle-De-Antibi-ticos`.
3. Nome do projeto: `controledeantibioticos` · Branch de produção: `main`.
4. Predefinição de framework: **Nenhuma** · Comando de build: **(vazio)** · Diretório de saída do build: **`public`**.
5. **Salvar e implantar.**
6. Firebase → Authentication → Configurações → **Domínios autorizados** → adicionar `controledeantibioticos.pages.dev`.

## Como publicar uma nova versão

Altere `public/index.html`, faça commit e push na `main`. O Pages publica em cerca de 1 minuto. Para testar antes, use outra branch: o Pages gera uma URL de pré-visualização.

## Desativar os endereços antigos

1. Confirme que o Pages funciona com login e dados.
2. Envie a pasta `redirect-enderecos-antigos/` como nova implantação no **Netlify** e no **Worker** `controledeantibioticos`.
3. Depois de duas a quatro semanas, exclua o site do Netlify e o Worker, e remova os domínios antigos dos domínios autorizados do Firebase.

## Backup

Configurações → Backup gera um `.json` com os dados. Faça semanalmente e guarde fora do navegador.
