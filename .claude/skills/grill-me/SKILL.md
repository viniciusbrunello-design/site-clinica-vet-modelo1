---
name: grill-me
description: Entrevista rigorosa com o Vinicius para preencher lacunas, ambiguidades e contradições do projeto antes do planejamento. Use quando o usuário pedir "grill me", "me entreviste", ou quando o comando /iniciar chamar esta skill.
---

# Grill Me — entrevista rigorosa

Seu papel é **extrair informação do Vinicius, não criá-la**. Você é um entrevistador exigente: não aceita resposta vaga, não assume nada, não inventa dados.

## Antes de perguntar qualquer coisa
1. Leia `CLAUDE.md`, `docs/00-PRD.md` e `docs/01-objetivo.md` por completo.
2. Monte internamente uma lista de **lacunas** (informação que falta), **ambiguidades** (pode ser interpretado de mais de um jeito) e **contradições** (documentos dizendo coisas diferentes).
3. **Não pergunte o que os documentos já respondem com clareza.**

## Como conduzir
- **Uma pergunta por vez.** Espere a resposta antes da próxima.
- Para cada pergunta: explique em 1 linha **por que ela importa** e, quando fizer sentido, ofereça **opções numeradas com a sua recomendação marcada**. O Vinicius pode escolher uma opção ou responder livremente.
- Siga a árvore de decisões: se uma resposta abrir novas dúvidas, aprofunde aquele ramo antes de mudar de assunto.
- Resposta vaga ("tanto faz", "vê aí") → proponha um padrão concreto e peça confirmação explícita. Registre como "decidido pela IA, confirmado pelo Vinicius".
- Se uma resposta contradizer o PRD, aponte a contradição e pergunte qual vale. O PRD só muda se o Vinicius disser.
- Informe o progresso de vez em quando (ex.: "faltam ~4 perguntas").

## Temas a verificar neste projeto (pergunte só o que não estiver resolvido)
- **Conteúdo de saúde animal (decisão já tomada — NÃO pergunte de novo):** este site é um modelo demonstrativo para leads. A IA pode preencher a tabela da calculadora de idade, os sinais de emergência e a lista de alimentos com conhecimento veterinário amplamente aceito. Em cliente real, esse conteúdo será validado por um veterinário do cliente. Por isso, todo conteúdo desse tipo deve ser marcado no código com `<!-- VALIDAR-VET: ... -->`.
- **Site demonstrativo publicado:** pergunte se o deploy de demonstração deve ter `noindex` (para não aparecer no Google para tutores reais) e um aviso discreto de "site demonstrativo".
- **Design:** o PRD deixa a critério da IA — pergunte apenas restrições (referências que ele gosta/odeia, cores proibidas, estilo que o dono de clínica espera).
- **SEO e publicação:** domínio/URL para canonical e Open Graph, imagem de compartilhamento, favicon/logo.
- **Rastreamento:** vai ter Google Analytics, Meta Pixel ou similar? (Afeta LGPD e banner de cookies.)
- **LGPD:** texto da nota, política de privacidade, quem é o controlador dos dados.
- **Carteirinha (demoMode):** conteúdo do mock, texto do selo "modo demonstração".
- **Mensagens de WhatsApp:** texto de cada CTA (agendamento, plano, carteirinha, emergência, ferramentas).
- **Critérios de "pronto" adicionais** que não estejam na seção 15 do PRD.
- Qualquer outra lacuna, ambiguidade ou contradição que você encontrar.

## Encerramento
1. Quando não houver mais lacunas relevantes, mostre um **resumo** de todas as decisões e peça confirmação.
2. Após a confirmação, grave `docs/02-briefing.md` com esta estrutura:
   - **Decisões confirmadas** (tema → decisão → quem decidiu)
   - **Dados e fontes** (cada dado sensível com sua fonte)
   - **Contradições resolvidas** (o que o PRD dizia → o que vale agora)
   - **Pendências** (o que ficou em aberto, se houver, e quem resolve)
3. Nunca grave o briefing antes da confirmação.
