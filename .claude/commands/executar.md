---
description: Etapas 5 e 6 — executa as camadas com os subagentes e supervisiona cada entrega até a aprovação
argument-hint: [opcional: ID de camada ou "onda N" para rodar só uma parte]
---

Escopo pedido pelo Vinicius (vazio = continuar de onde parou): $ARGUMENTS

Você é o **orquestrador**. Você não escreve código do site: você distribui o trabalho, repassa pareceres, roda o build, commita e mantém o Vinicius informado.

## Pré-requisitos
1. `docs/04-camadas.md` precisa estar **APROVADO**. Se não estiver, **PARE** e diga para rodar `/camadas`.
2. Leia `CLAUDE.md`, `docs/04-camadas.md` e `docs/05-status.md`.
3. **Retomada:** camadas já registradas como "APROVADA" em `05-status.md` não são refeitas. Continue da primeira camada pendente (ou do escopo pedido acima).

## Para cada onda (na ordem de `04-camadas.md`)
4. Anuncie ao Vinicius em 1 linha: qual onda, quais camadas, quais agentes.
5. Chame o agente de cada camada (em paralelo **apenas** se a onda permitir), passando:
   - o **ID da camada**;
   - a instrução: "Leia sua especificação em docs/04-camadas.md e siga as regras do seu arquivo de agente."
   - para o `executor-camada`, reforce o ID.
6. **Pedidos de mudança de contrato** nos relatórios:
   - se afetar só arquivos de outra camada → chame o agente dono para aplicar, depois retome;
   - se mudar requisito do PRD/briefing ou for ambíguo → **PARE e pergunte ao Vinicius**.
7. Ao fim da onda, rode `npm run build` (se a fundação já existir). Erro de build → identifique a camada dona do arquivo e devolva a ela.

## Supervisão (para cada camada da onda)
8. Chame o `supervisor` no **Modo 1**, passando o ID da camada e o relatório do agente.
9. **REPROVADO** → chame o agente da camada de novo com o **parecer completo** e a instrução "corrija somente os itens listados". Depois, volte ao passo 8.
   - Máximo de **3 ciclos** por camada. No 3º REPROVADO, **PARE**, mostre os pareceres ao Vinicius e peça uma decisão.
10. **APROVADO** →
    - registre em `docs/05-status.md`: camada, data, nº de ciclos, observações do supervisor;
    - commit: `Camada <ID> aprovada pelo supervisor`. **Não faça push.**

## Portão do design (obrigatório)
11. Quando a camada de design for aprovada pelo supervisor, **PARE** e diga ao Vinicius:
    - como ver a pré-visualização (`npm run dev` e o endereço da página de preview);
    - pergunte: **aprova o design ou quer ajustes?**
    - Ajustes → devolva ao `camada-design` com os pedidos e repita a supervisão.
    - Aprovado → marque `docs/design-system.md` como `APROVADO em <data>`, commit, e só então siga para as próximas ondas.

## Encerramento
12. Com todas as camadas aprovadas, chame o `supervisor` no **Modo 2 (revisão final)**. REPROVADO → devolva cada correção à camada dona, com o mesmo limite de 3 ciclos.
13. Aprovado na revisão final:
    - atualize `docs/05-status.md` (projeto concluído);
    - commit: `Projeto aprovado na revisão final`. **Não faça push.**
14. Entregue ao Vinicius um resumo curto:
    - o que foi construído;
    - **como testar localmente** (comando + roteiro de teste das ferramentas interativas);
    - lista de conteúdos marcados `VALIDAR-VET`;
    - pendências e observações do supervisor.
15. **PARE.** Informe que o próximo passo é revisar o site e, se aprovado, fazer o push e conectar a Vercel.

## Regras do orquestrador
- Mantenha o Vinicius informado com mensagens curtas a cada camada iniciada, reprovada ou aprovada.
- Nunca edite você mesmo arquivos do site para "ajeitar rápido": devolva à camada dona.
- Nunca pule o supervisor nem o portão do design.
