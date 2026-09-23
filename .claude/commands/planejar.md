---
description: Etapa 3 — gera ou corrige o plano técnico (docs/03-plano.md) com o agente planejador
argument-hint: [opcional: correções a aplicar no plano]
---

Correções pedidas pelo Vinicius (vazio = gerar plano novo): $ARGUMENTS

Execute a etapa 3 do fluxo definido no CLAUDE.md:

## Pré-requisitos
1. Confirme que `docs/01-objetivo.md` e `docs/02-briefing.md` existem. Se algum faltar, **PARE** e diga para rodar `/iniciar` primeiro.
2. Se `docs/03-plano.md` já estiver com status APROVADO e não houver correções acima, **PARE** e avise que o plano já foi aprovado.

## Execução
3. Chame o subagente `planejador`:
   - sem correções → gerar `docs/03-plano.md` do zero;
   - com correções → repassar exatamente as correções acima para ele aplicar.
4. Mostre ao Vinicius:
   - o resumo devolvido pelo planejador;
   - as **perguntas em aberto** e os **riscos**, em destaque.
5. Pergunte: **aprova o plano ou quer correções?**
   - Correções → chame o `planejador` de novo com elas e volte ao passo 4.
   - Aprovado → mude o status em `docs/03-plano.md` para `APROVADO em <data>`.

## Encerramento (somente após aprovação)
6. Atualize `docs/05-status.md`: etapa 3 concluída, data, pendências.
7. Commit com a mensagem "Etapa 3: plano aprovado". **Não faça push.**
8. **PARE.** Informe que o próximo passo é a etapa 4 (camadas).
