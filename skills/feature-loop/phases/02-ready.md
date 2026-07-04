# Fase 2 — Prontidão & handoff (chapéu PO)

Verifica que a história está **pronta pra desenvolvimento** (Definition of Ready), muda a raia, e
entrega ao usuário o prompt de handoff. **A skill PARA aqui** — quem leva o card até a builder é o
usuário. A feature-loop só volta à cena na aterrissagem (ou se o card voltar ao refinamento).

## Definition of Ready (checklist)

Confira contra o `CHG-NNN` em disco (o usuário pode tê-lo editado — o disco é a fonte da verdade):

- [ ] Tipo classificado e confirmado com o usuário.
- [ ] JTBD com o *para* claro (valor articulado).
- [ ] Contexto de produto e objetivo escritos — a builder entende o porquê sem adivinhar.
- [ ] Critérios de aceite testáveis, **incluindo o de preservação**.
- [ ] Fatia mínima definida; se épico, histórias sequenciais no backlog e esta marcada.
- [ ] Ambiguidades de **produto** todas decididas (log de decisões preenchido).
- [ ] Ambiguidades **técnicas** listadas explicitamente para a builder (seção 5) — nenhuma decidida no chute.
- [ ] Fatia com UI → mockup de UX apensado, rotulado construível/aspiracional, revisado pelo usuário.

Item faltando → volte à fase 1 e feche-o. Não empurre história incompleta pra frente — o retorno do
card depois custa mais que fechar agora.

## Handoff

1. **Mudar a raia.** Status do CHG e do `backlog.md` → `pronta para desenvolvimento`.

2. **Indicar o roteiro** — quais builders e em que ordem, pelo que a história pede (sem fazer a
   análise por elas): toca regra/dado → `ddd-architect` primeiro; toca interface → `ui-architect`
   depois do contrato (o gate dela continua valendo); fatia full-stack → domínio → contrato → front.

3. **Entregar o prompt de handoff pronto** pro usuário copiar, um por builder. Modelo:

   > Vamos trabalhar a fatia `docs/changes/CHG-NNN-<slug>.md` pela skill `<builder>` em **modo
   > evolução**. Comece pela análise de impacto (fase E1) e pare no checkpoint pra eu revisar
   > antes de construir.

4. **Parar.** Diga explicitamente que a raia agora é do usuário e o que esperar de volta: a builder
   preenche as seções 6–7 (análise de impacto + plano), **para no checkpoint humano**, e após o
   "segue" constrói e entrega a fatia `construída` — sem commit. Aí a feature-loop aterrissa.

## O card pode voltar

Se a builder, na análise de impacto, encontrar lacuna de **produto** (critério ambíguo, história
subespecificada, mockup impossível de ancorar no contrato), ela devolve o card: status →
`em refinamento`, com a lacuna anotada no CHG. Ao retomar, a feature-loop lê a anotação, conduz a
fase 1 no que faltou, e refaz esta prontidão. Voltar raia é o mecanismo funcionando — não é falha.

## Saída

- `CHG-NNN` e `backlog.md` com status `pronta para desenvolvimento`.
- Prompt(s) de handoff entregues ao usuário. **Skill parada.**

## Regras

- A feature-loop **não invoca as builders** — o handoff é do usuário (é a raia dele no board).
- Não adiante análise técnica "pra ajudar a builder" — é exatamente o trabalho que pertence a ela.
- DoR incompleta bloqueia o handoff. Sem exceção "é só uma coisinha".
