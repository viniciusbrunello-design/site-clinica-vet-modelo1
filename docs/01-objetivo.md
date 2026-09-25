# 01 — Objetivo

## Objetivo informado pelo Vinicius
> Construir o Site 1 (arquétipo clínico) de referência para clínicas veterinárias, usando a Clínica VetSaúde como cliente demonstrativo. O site será usado para mostrar a leads como ficaria o site deles e depois virará um template clonável por cliente.

## Objetivo de negócio (PRD, seção 1)
O site tem **um único objetivo de negócio**: transformar visitante em **lead no WhatsApp** da clínica (agendamento ou dúvida). Todo o resto é suporte a isso.

Este é o primeiro de dois sites de referência para clínicas veterinárias. Depois de pronto e refinado, ele vira um template padronizado que será clonado e personalizado por cliente. Portanto: construir como um site único e excelente para o cliente descrito (Clínica VetSaúde), mas mantendo os dados do cliente isolados (`src/data/cliente.json`) para facilitar a extração de template mais tarde.

Este site representa o **arquétipo clínico** (medicina veterinária séria: consultas, vacinas, cirurgia, emergência 24h). Ele **não** tem galeria de antes/depois de banho e tosa — isso é do Site 2 (arquétipo estética). Manter essa fronteira limpa é proposital.

## Fora de escopo (PRD, seção 14)
Não construir agora:

- Galeria/slider de antes e depois de banho e tosa (isso é o **Site 2**).
- Login, área do tutor, prontuário, qualquer backend, banco de dados ou funções serverless.
- Pagamento real / cobrança.
- Múltiplas páginas ou blog (pode ser add-on de SEO numa fase futura; agora é single page).
- Integração real de IA na carteirinha (fica em `demoMode`; o endpoint é plugado depois).
