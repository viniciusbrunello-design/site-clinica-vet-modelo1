# 02 — Briefing (resultado da entrevista)

> Complementa o `docs/00-PRD.md`. Se houver contradição entre os dois, vale o PRD até o Vinicius decidir o contrário. Entrevista conduzida pela skill `grill-me` em 25/09/2026.

## Decisões confirmadas

| Tema | Decisão | Quem decidiu |
|------|---------|--------------|
| **Deploy de demonstração** | O site demo leva `<meta name="robots" content="noindex,nofollow">` e um aviso discreto de "site demonstrativo — modelo de exemplo, não é uma clínica real". Ambos são removidos quando virar cliente real. Controlar por flag em `cliente.json` (ex.: `demoSite: true`). | Vinicius |
| **URL para canonical/OG** | Usar a URL provisória da Vercel por enquanto, exposta como campo `site` configurável (Astro config + `cliente.json`). O Vinicius passará o nome exato do projeto na Vercel depois. | Vinicius |
| **Design — paleta** | Confiança na IA: paleta vet clássica, séria, limpa e acolhedora (tendência azul/teal confiável + acento quente para CTA/emergência), sem infantilizar. O `docs/design-system.md` é gerado e **aprovado pelo Vinicius antes de qualquer seção**. | Vinicius (delegado à IA) |
| **Design — o que evitar** | Fugir dos "tells" de site genérico feito por IA: gradiente roxo padrão, emojis usados como ícones, herói centralizado genérico, cards todos idênticos, stock-art infantil. A camada de design lista explicitamente o que evitar no `design-system.md`. | Vinicius |
| **Rastreamento** | Nenhum agora (sem Google Analytics, sem Meta Pixel). Logo, **sem banner de cookies**. Deixar um ponto comentado no código para plugar analytics no futuro (exigirá revisar a LGPD). | Vinicius |
| **LGPD** | A nota de rodapé fala em nome da **Clínica VetSaúde** como controladora dos dados, com e-mail de contato (`contato@vetsaude.com.br`). Sem página de política separada — mantém single page. No cliente real, troca-se por nome/CNPJ dele. | Vinicius |
| **Carteirinha (demoMode)** | Mock realista: espécie cão, algumas vacinas "em dia" e **pelo menos uma "verificar/pendente"** (ex.: V10 03/2024 em dia · Antirrábica 01/2024 verificar · Gripe canina — verificar). Selo visível "Modo demonstração". Status é sempre **por vacina**; nunca um "tudo em dia" global (regra de segurança do PRD seção 7). Disclaimer + botão WhatsApp sempre presentes. | Vinicius |
| **Mensagens de WhatsApp** | As 6 mensagens abaixo aprovadas como redigidas; guardadas em `cliente.json` (as `{}` são preenchidas em runtime). | Vinicius |
| **Logo / favicon / OG** | Logo "pata + Clínica VetSaúde" em SVG inline (usa variáveis CSS); favicon derivado em SVG/PNG; imagem de compartilhamento (Open Graph) gerada como bloco com gradiente da marca + nome + "Emergência 24h", sem stock-art. | Vinicius |

### Mensagens de WhatsApp aprovadas
1. **Agendamento (Herói 2):** "Olá! Quero agendar *{serviço}* para o *{pet}*. Dia preferido: {data} ({período}). Vim pelo site."
2. **Plano VetSaúde:** "Olá! Vim pelo site e quero saber mais sobre o Plano VetSaúde (mensalidade, o que inclui e como assinar)."
3. **Carteirinha (após triagem):** "Olá! Fiz a triagem da carteirinha no site e apareceram vacinas para verificar: {vacinas}. Gostaria de confirmar a situação do meu pet com a equipe."
4. **Emergência 24h:** "Olá! É uma *emergência* com meu pet e preciso de atendimento agora."
5. **CTA genérico (nav / botão flutuante):** "Olá! Vim pelo site da Clínica VetSaúde e gostaria de tirar uma dúvida."
6. **Ferramentas (Pacote Captação):** "Olá! Usei as ferramentas do site e gostaria de agendar um check-up / tirar uma dúvida sobre meu pet."

## Decisões já fixadas pelo processo (não reperguntadas)
- **Conteúdo de saúde animal** (calculadora de idade, sinais de emergência, lista "pode comer isso?") pode ser preenchido pela IA com conhecimento veterinário amplamente aceito, por ser site demonstrativo. Todo esse conteúdo deve ser marcado no código com `<!-- VALIDAR-VET: ... -->`. Em cliente real, será validado por um veterinário do cliente.
- **Dados do cliente** exclusivamente em `src/data/cliente.json`; nada hardcoded nos componentes.
- **Sem** backend, banco de dados, funções serverless, `localStorage`/`sessionStorage`.
- **WhatsApp** só via link `wa.me`; nunca a API oficial.
- **Chave de API** nunca no front-end; caminho de produção da carteirinha passa por endpoint do dono.

## Dados e fontes
- **Dados do cliente** (nome, endereço, equipe/CRMV, horários, contato, prova social, serviços, plano): PRD seção 3. São fictícios mas tratados como reais.
- **Estatística da faixa-âncora da carteirinha** ("cinomose e parvovirose matam até 90% dos cães não vacinados"): PRD seção 7 — usar **exatamente** a fonte citada no PRD (letalidade de cinomose 50–90% em não vacinados; parvovirose de alta mortalidade em filhotes). **Nunca** inventar número novo.
- **Depoimentos (3):** de demonstração, redigidos pela IA (PRD seção 9), incluindo um sobre emergência noturna e um sobre lembrete de vacina. Marcados como demo — em cliente real, substituir por avaliações reais.
- **Conteúdo de saúde animal:** conhecimento vet amplamente aceito, marcado `<!-- VALIDAR-VET: ... -->`.

## Contradições resolvidas
- Nenhuma contradição entre PRD e briefing. As decisões acima **complementam** o PRD (pontos que o PRD deixou em aberto: comportamento do deploy demo, URL, rastreamento, controlador LGPD, formato de logo/OG, texto das mensagens).

## Pendências
| Pendência | Quem resolve | Quando |
|-----------|--------------|--------|
| Nome exato do projeto/URL na Vercel para preencher `site` (canonical/OG). | Vinicius | Antes do deploy; usar placeholder até lá. |
| **Imagem Open Graph precisa ser PNG/JPG 1200×630 em URL absoluta** — SVG não renderiza como prévia no WhatsApp/redes. Definir no plano como gerar (Astro build ou export único). | Camada SEO/Fundação (definir no `/planejar`) | Etapa de planejamento. |
| `endpointCarteirinha` real (webhook N8N/backend do dono) para produção. Fica vazio/comentado; site opera em `demoMode`. | Vinicius (fase futura) | Pós-template. |
| Aprovação do `docs/design-system.md` antes de construir seções. | Vinicius | Etapa de execução (camada design). |
