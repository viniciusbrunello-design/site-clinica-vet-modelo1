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

### Próximo passo
Revisar o `docs/02-briefing.md` e, quando aprovado, rodar `/planejar` (Etapa 3).
