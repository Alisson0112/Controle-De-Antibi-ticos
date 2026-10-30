# Sistema de Controle de Antibióticos

Sistema web de controle de uso de antibióticos por paciente e por setor, com controle de estoque e relatórios, usado pelos farmacêuticos da Farmácia.

## Estrutura

| Arquivo | Descrição |
|---|---|
| `index.html` | O sistema completo (HTML + CSS + JavaScript em um único arquivo, sem etapa de build). **É o arquivo publicado.** |
| `docs/RESUMO_DO_PROJETO.md` | Regras de negócio, histórico de alterações, mapa do código e pendências. |
| `planilha/controle_antibioticos.xlsx` | Versão em Excel (agosto/2026). Desatualizada em relação ao sistema web. |

## Arquitetura

- **Front-end:** `index.html` estático, sem dependências de build.
- **Banco de dados e login:** Firebase, projeto `controle-de-antibiotico` (Firestore + Authentication por e-mail e senha). Nenhum dado fica na hospedagem nem neste repositório.
- **Hospedagem:**
  - Cloudflare Workers: `controledeantibioticos.planilhahne.workers.dev`
  - Netlify: `controledeantibioticos.netlify.app`

> A configuração web do Firebase no `index.html` é pública por design. Quem protege os dados são as **regras de segurança do Firestore**, que devem exigir usuário autenticado.

## Abas

Registros · Estoque · Protocolo de Infusão · Relatórios · Configurações

## Como publicar uma nova versão

1. Altere o `index.html` e faça commit neste repositório.
2. **Cloudflare:** Workers e Pages → `controledeantibioticos` → Nova implantação → enviar o `index.html`.
3. **Netlify:** projeto `controledeantibioticos` → Deploys → arrastar o arquivo.
4. Atualize **os dois** endereços, porque ambos gravam no mesmo banco.
5. Se um endereço novo for criado, cadastre-o no Firebase em Authentication → Configurações → Domínios autorizados.
6. Use Ctrl+F5 para conferir a versão nova.

## Backup

Configurações → Backup gera um `.json` com os dados. Faça semanalmente e guarde fora do navegador.
