---
description: Etapas 1 e 2 — registra o objetivo do projeto e conduz a entrevista (grill-me)
argument-hint: [objetivo do projeto em uma ou duas frases]
---

Objetivo informado pelo Vinicius: $ARGUMENTS

Execute as etapas 1 e 2 do fluxo definido no CLAUDE.md:

## Etapa 1 — Objetivo
1. Leia `CLAUDE.md` e `docs/00-PRD.md` por completo.
2. Grave `docs/01-objetivo.md` contendo:
   - o objetivo informado acima, com as palavras do Vinicius;
   - o objetivo de negócio do PRD (seção 1);
   - o que está fora de escopo (seção 14 do PRD).
3. Mostre o arquivo e peça confirmação. **Não siga sem o "ok".**

## Etapa 2 — Entrevista
4. Use a skill `grill-me` para entrevistar o Vinicius.
5. Ao final, a skill grava `docs/02-briefing.md` (somente após a confirmação dele).

## Encerramento
6. Crie (ou atualize) `docs/05-status.md` registrando: data, etapas 1 e 2 concluídas, pendências do briefing.
7. Faça commit com a mensagem "Etapas 1 e 2: objetivo e briefing". **Não faça push.**
8. **PARE.** Informe que o próximo passo é revisar o briefing e depois rodar `/planejar`.
