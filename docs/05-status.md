# 05 — Status do projeto

Registro do andamento (o que foi feito, por qual agente, pareceres do supervisor).

## 2026-09-25

### Etapa 1 — Objetivo ✅ concluída
- Gravado `docs/01-objetivo.md` com o objetivo do Vinicius, o objetivo de negócio (PRD §1) e o fora de escopo (PRD §14).
- Aprovado pelo Vinicius.

### Etapa 2 — Entrevista (grill-me) ✅ concluída
- Entrevista conduzida (8 perguntas). Gravado `docs/02-briefing.md` após confirmação do Vinicius.
- Decisões-chave: deploy demo com `noindex` + aviso discreto; URL da Vercel (campo `site` configurável); design delegado à IA com design-system aprovado antes das seções; sem rastreamento; LGPD em nome da Clínica VetSaúde; mock realista da carteirinha com ≥1 vacina "verificar"; 6 mensagens de WhatsApp aprovadas; logo/favicon/OG em SVG/CSS.

### Pendências abertas do briefing
- Nome/URL exata do projeto na Vercel (Vinicius, antes do deploy).
- Imagem Open Graph precisa ser PNG 1200×630 em URL absoluta — resolver no `/planejar`.
- `endpointCarteirinha` real (fase futura; site opera em `demoMode`).
- Aprovação do `docs/design-system.md` antes das seções.

## 2026-09-29

### Etapa 3 — Plano técnico ✅ concluída
- Gerado `docs/03-plano.md` pelo subagente `planejador` a partir do PRD e do briefing.
- Uma rodada de correções aplicada (8 itens pedidos pelo Vinicius): flags duplicadas removidas do `cliente.json` (`demoMode` único; `flags.temCarteirinha`/`flags.especies` como fontes únicas); verificação de Origin por mesma origem (não pelo campo `site`); robots.txt/sitemap como rotas geradas do `cliente.json` com `noindex` como único bloqueio em demo; Vitest + zod nos testes; variáveis CSS no passo 2 (design system); perguntas 1 e 2 resolvidas (URL provisória e OG via `sharp`); agendamento sem datas passadas (dd/mm/aaaa); matriz e riscos atualizados.
- **APROVADO pelo Vinicius em 2026-09-29.**

### Pendências abertas (não bloqueiam)
- Provisionamento da IA real: chave Anthropic (com teto de gasto), Upstash e variáveis de ambiente (Vinicius). Até lá, o site opera em modo demonstração (falha fechada).
- Aprovação do `docs/design-system.md` antes das seções (etapa de execução).
- Confirmação/troca da URL da Vercel no `cliente.json` na hora do deploy (provisória: `https://site-clinica-vet-modelo1.vercel.app`).

### Próximo passo
Rodar `/camadas` (Etapa 4): dividir o plano aprovado em camadas de trabalho (`docs/04-camadas.md`).
