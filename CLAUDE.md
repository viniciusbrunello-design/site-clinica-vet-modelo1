# CLAUDE.md — Site Clínica Vet (Modelo 1)

## O projeto
Site single page para clínicas veterinárias (arquétipo clínico), construído em **Astro** (páginas estáticas + uma única função serverless para a leitura de carteirinha por IA) e publicado na **Vercel via GitHub**. Depois de pronto, vira template clonável por cliente. Objetivo de negócio único: transformar visitante em **lead no WhatsApp**.

## Fontes da verdade (nesta ordem de prioridade)
1. `docs/00-PRD.md` — requisitos do produto. **Só o Vinicius edita este arquivo.**
2. `docs/02-briefing.md` — resultado da entrevista (complementa o PRD).
3. `docs/03-plano.md` — plano aprovado.
4. `docs/04-camadas.md` — divisão em camadas, escopo de arquivos e critérios de aceite.
5. `docs/design-system.md` — referência visual aprovada.

Se duas fontes se contradisserem, **pare e pergunte ao Vinicius**. Nunca resolva contradição por conta própria.

## Fluxo de trabalho obrigatório (não pule etapas)
1. **Objetivo** → `docs/01-objetivo.md`
2. **Entrevista** (skill grill-me) → `docs/02-briefing.md` → PARAR para aprovação
3. **Planejamento** → `docs/03-plano.md` → PARAR para aprovação
4. **Camadas** → `docs/04-camadas.md` → PARAR para aprovação
5. **Execução pelos subagentes**, um por camada
6. **Supervisão** → avaliação contra os critérios de aceite → correções (máx. 3 ciclos por camada; depois disso, PARAR e consultar o Vinicius)

Nenhum código do site é escrito antes de `03-plano.md` e `04-camadas.md` estarem aprovados.
Registre o andamento em `docs/05-status.md` (o que foi feito, por qual agente, pareceres do supervisor).

## Regras de execução
- **Posse de arquivos:** cada subagente só edita os arquivos/pastas atribuídos à sua camada em `04-camadas.md`. Precisa mexer fora? Pare e reporte ao orquestrador.
- **Dados do cliente** só em `src/data/cliente.json`. Nada hardcoded nos componentes.
- **Nunca invente dados** (números, estatísticas, contatos). Se faltar, pergunte.
- **Nunca** use `localStorage`/`sessionStorage`.
- **Backend:** a **única** exceção permitida é a função `src/pages/api/carteirinha.ts` (PRD seção 7). Nenhum outro endpoint, função serverless ou banco de dados. O Upstash Redis é usado **somente** para os contadores de limite de uso.
- **Segredos:** chaves e tokens **só** em variáveis de ambiente (`.env` local, fora do Git, e painel da Vercel). Nunca no código, no front-end, em logs, em mensagens de erro, em `docs/` ou no chat. Você **não** lê o `.env`: use o `.env.example` (sem valores) como referência.
- **IA real da carteirinha:** a IA só lê (nomes e datas); quem classifica "em dia/verificar" é o código com `src/data/regras-vacinas.json`. Sem chave **ou** sem limitador configurado → não chama a IA (falha fechada, cai no modo demonstração). Limite de uso por IP e teto global diário são obrigatórios. Imagem nunca é armazenada.
- **WhatsApp** só via link `wa.me`. Emergência: ação principal "Ligar agora" (`tel:`).
- **Carteirinha:** nunca afirmar "está tudo em dia / seu pet está protegido" como conclusão geral. Status sempre por vacina, triagem + disclaimer + WhatsApp.
- **"Meu pet pode comer isso?"** usa lista fixa curada, nunca geração livre de IA.
- Todo texto do site e da documentação em **português do Brasil**.

## Ambiente e comandos
- Sistema: Windows, terminal PowerShell.
- `npm run dev` — servidor local de desenvolvimento
- `npm run build` — gera o site (tem que passar sem erros antes de qualquer entrega)
- Testar a IA real localmente exige que o **Vinicius** preencha o `.env`. Sem isso, teste em modo demonstração e com testes automatizados que simulam a resposta da IA (sem chamar a API de verdade).

## Git
- Commit ao final de cada etapa aprovada, com mensagem clara em português.
- **Nunca faça `git push` sem autorização explícita do Vinicius** (o push para `main` publica o site na Vercel).
- Nunca use comandos destrutivos (`git reset --hard`, `git push --force`, apagar pastas) sem autorização.
