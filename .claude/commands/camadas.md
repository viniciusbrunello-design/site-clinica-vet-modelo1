---
description: Etapa 4 — divide o plano aprovado em camadas de trabalho (docs/04-camadas.md) com o agente arquiteto-camadas
argument-hint: [opcional: correções a aplicar na divisão de camadas]
---

Correções pedidas pelo Vinicius (vazio = gerar divisão nova): $ARGUMENTS

Execute a etapa 4 do fluxo definido no CLAUDE.md:

## Pré-requisitos
1. Confirme que `docs/03-plano.md` existe e está com status **APROVADO**. Se não estiver, **PARE** e diga para rodar `/planejar` primeiro.
2. Se `docs/04-camadas.md` já estiver APROVADO e não houver correções acima, **PARE** e avise.

## Execução
3. Chame o subagente `arquiteto-camadas`:
   - sem correções → gerar `docs/04-camadas.md` do zero;
   - com correções → repassar exatamente as correções acima.
4. Faça você mesmo uma checagem rápida do arquivo gerado e aponte ao Vinicius se encontrar:
   - arquivo com mais de uma camada dona;
   - item da matriz de rastreabilidade sem camada;
   - critério de aceite subjetivo (não verificável);
   - camada atribuída a um agente que não existe em `.claude/agents/`.
5. Mostre ao Vinicius, em linguagem simples:
   - tabela das camadas (camada · agente · onda);
   - as ondas de execução e os **portões de aprovação**;
   - problemas encontrados no passo 4 e itens em aberto.
6. Pergunte: **aprova a divisão ou quer correções?**
   - Correções → chame o `arquiteto-camadas` de novo com elas e volte ao passo 4.
   - Aprovado → mude o status em `docs/04-camadas.md` para `APROVADO em <data>`.

## Encerramento (somente após aprovação)
7. Atualize `docs/05-status.md`: etapa 4 concluída, data, pendências.
8. Commit com a mensagem "Etapa 4: camadas aprovadas". **Não faça push.**
9. **PARE.** Informe que o próximo passo é rodar `/executar`.
