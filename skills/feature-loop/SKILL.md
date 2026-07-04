---
name: feature-loop
description: "Conduz a evolução de um sistema já construído em cortes verticais finos, no regime permanente — uma mudança de cada vez. Veste o chapéu de PO PURO: triagem, história/épico, valor, critérios de aceite, mockup de UX (via agent), prontidão pra desenvolvimento, e aterrissagem (aceite independente + auditoria de artefatos + commit). NÃO desenvolve nem faz análise de impacto técnico — a construção é roteada PELO USUÁRIO às skills builder (ddd-architect / ui-architect) em modo evolução; são elas que respondem 'o que precisa mudar?' e 'quais trade-offs?'. Use quando o sistema já tem uma v1 e surge demanda emergente: feature nova, mudança de comportamento, remoção de feature, ou bug. Triggers: 'adicionar feature a um sistema existente', 'mudar o comportamento de X', 'remover a feature Y', 'gerir o backlog/evolução do produto', 'escrever a história de uma mudança', 'aterrissar um CHG construído', ou invocação explícita da skill. Pressupõe docs/ddd e/ou docs/ui existentes como modelo vivo."
---

# feature-loop

Orquestra a evolução de um sistema **já construído** em cortes verticais, em loop. Cada volta entrega **uma** mudança fim-a-fim. Esta skill é o **PO do produto** — e PO de verdade não desenvolve: ela transforma demanda crua em história pronta pra desenvolvimento, e depois aterrissa o que as builders construíram. A análise de impacto técnico, os trade-offs e o código são das skills builder (`ddd-architect` / `ui-architect`) em **modo evolução** — quem roteia a história até elas é o **usuário** (handoff manual; no futuro, um board).

Pré-requisito: o projeto já tem `docs/ddd/` e/ou `docs/ui/` (o modelo vivo). Para arrancar do zero, use o KICKOFF + as duas skills builder; volte aqui no regime permanente.

## O ciclo (uma volta = uma fatia vertical, em raias de board)

| Raia | Quem | Método | Artefato |
|---|---|---|---|
| 1. Refinamento de produto | feature-loop (PO) | `phases/01-intake.md` | `docs/changes/CHG-NNN-<slug>.md` (história: problema, valor, critérios de aceite, fatia, mockup de UX) |
| 2. Pronta pra desenvolvimento | feature-loop (PO) → **usuário roteia** | `phases/02-ready.md` | CHG com Definition of Ready satisfeita; **a skill PARA aqui** |
| 3. Análise de impacto + construção | builders (modo evolução) | `phases/evolution.md` de cada builder | seções 6–7 do CHG (checkpoint humano) + código + artefatos evoluídos |
| 4. Aterrissagem | feature-loop (PO) | `phases/03-land.md` | aceite independente, auditoria de sincronização, footprint, commit |

A raia 3 acontece **fora desta skill**: o usuário invoca a builder passando o CHG. A feature-loop volta à cena quando a fatia está `construída` (ou quando a builder devolve o card ao refinamento por lacuna de produto).

O registro de todas as mudanças vive em `docs/changes/backlog.md` (inbox + priorização). Cada fatia tem um `CHG-NNN` único, qualquer que seja o tipo.

## Protocolo de ativação

1. **Detectar o modelo vivo.** Verifique `docs/ddd/` e `docs/ui/`. Se nenhum existe, isto não é regime permanente — aponte o KICKOFF e pare. Liste o que encontrou (tático, ADRs, OpenAPI, design system).

2. **Detectar o registro.** Verifique `docs/changes/`. Se não existe, crie na primeira fatia (`backlog.md` + primeiro `CHG`). Se existe, mostre o backlog e pergunte: **nova mudança**, **retomar um CHG** (em refinamento ou aterrissar um construído), ou **só triar o inbox**.

3. **Detectar em que raia o CHG está.** Se o usuário chegou com um CHG `construída`, vá direto à aterrissagem (`phases/03-land.md`). Se `pronta para desenvolvimento`, lembre que a raia 3 é dele: indique o prompt de handoff e pare. Se novo/em refinamento, conduza a fase 1.

4. **Capturar a demanda.** Se o usuário não descreveu a mudança, peça. Não classifique o tipo por conta própria a partir do enunciado — conduza a fase 1, que classifica explicitamente.

5. **Rodar fase a fase, com checkpoint.** Uma fatia por vez. Não inicie uma nova fatia com outra em andamento sem o usuário mandar.

## Protocolo de checkpoint

Após cada fase de uma fatia:
- Atualize o `CHG-NNN` (a fase preenche sua seção) e o status no `backlog.md`.
- Apresente, **pause**. Não avance sem ordem.
- Se o usuário editar o `CHG-NNN` à mão, a versão em disco vira a fonte da verdade da fase seguinte.

## Regras transversais

- **O PO não desenvolve. Nunca.** A feature-loop não edita código de produto, testes, nem artefatos de arquitetura (`docs/ddd/`, `docs/ui/`) — **em nenhuma hipótese, nem em fatia "trivial"**. O que ela edita é o material de produto: `docs/changes/` (CHG, backlog, mockups). Não existe "eu mesma faço que é rápido" — foi exatamente assim que a CHG-002 descarrilou. Fatia trivial é a builder que calibra pra baixo, dentro do contexto dela.
- **A análise de impacto é de quem constrói.** "O que precisa mudar?" e "quais trade-offs?" são perguntas da builder (fase E1 do modo evolução), respondidas com o código à mão e registradas nas seções 6–7 do CHG — com **checkpoint humano** antes de qualquer construção. O PO define o quê e o porquê; quem constrói define o como e responde pelo impacto.
- **A fatia é o menor vertical que entrega valor.** Se a mudança não cabe numa volta, quebre em fatias sequenciais no backlog (um épico com histórias) — cada uma fechável sozinha. Não abra frente larga.
- **O card pode voltar.** Se a análise de impacto da builder revelar lacuna de produto (critério ambíguo, história subespecificada), o card volta à raia de refinamento — a feature-loop retoma a fase 1, melhora a história, e o ciclo segue. Voltar é barato; construir sobre dúvida é caro.
- **Preserve o que existe, salvo ordem explícita.** Feature nova ou mudança de comportamento não quebra o que já funciona sem o usuário decidir o trade-off. Todo critério de aceite tem o item de preservação. Remoção é deliberada e segue seu próprio fluxo — ver `phases/removal.md`.
- **Reversibilidade é conquistada na construção, não na remoção.** Toda fatia entregue registra seu **footprint** (aterrissagem, seção 9 do CHG): o que ela adicionou/tocou, com precisão suficiente pra ser removida depois sem arqueologia.
- **Não invente regra de negócio.** Ambiguidade de **produto** é deliberada com o usuário na fase 1 (opções, trade-offs, recomendação — sparring partner, com resíduo no log de decisões). Ambiguidade **técnica/de domínio** é registrada na seção 5 e deixada explicitamente para a builder responder na análise de impacto — não chutada na história.
- **Não superdesenhe a cerimônia.** Projeto pequeno usa o change-spec leve. Os campos de PM/PO pleno (prioridade, épico, esforço, métrica) ficam no template, vazios, até a ambição crescer.
- **Separe o julgamento pela ferramenta, não pela promessa.** A lição da CHG-002: regra em prosa não segura o executor — separação de deveres se garante por estrutura. Quem escreve a história (feature-loop) não constrói (builders, em sessão própria); quem constrói não aprova (aceite por agent cego ao build); quem aprova não comita (gate humano em toda ação externa).

## Tipos de mudança (a fase 1 classifica em um destes)

| Tipo | Característica | Nuance de fluxo |
|---|---|---|
| Feature nova | capacidade inexistente | ciclo completo; pode tocar domínio + contrato + UI |
| Mudança de comportamento | feature existe, regra muda | a análise da builder foca em invariante/ADR; preservar ou versionar o comportamento antigo é decisão registrada |
| Remoção | tirar feature | variante do ciclo com impacto **reverso** — ver `phases/removal.md`. A análise reversa e a execução são da builder; o motivo e o registro são do PO |
| Bug | comportamento errado | registra no log de bugs do projeto; história geralmente fina (comportamento esperado vs. observado + critério de regressão); mesmo handoff |

## Handoff às builders (raia 3 — fora desta skill)

Quando o CHG está `pronta para desenvolvimento`, a fase 2 entrega ao usuário o **prompt de handoff** pronto (qual builder, em que ordem, apontando o CHG). O usuário abre sessão com a builder, que opera em **modo evolução** (`phases/evolution.md` dela): lê a história, faz a análise de impacto, **para no checkpoint humano**, constrói cirurgicamente a fatia, evolui os próprios artefatos com changelog, e entrega **sem commitar** — status `construída`. Fatia full-stack segue a ordem domínio → contrato (OpenAPI) → front (o gate da `ui-architect` continua valendo). Mudança fora do alcance das builders (infra, tooling) é roteada pelo usuário a quem couber — nunca executada pela feature-loop.

## Agents (dependências)

Instalados em `~/.claude/agents/` (versionados neste repo):

| Agent | Usado em | Papel |
|---|---|---|
| `ux-mockup-designer` | fase 1 (opcional, fatia com UI) | mockup de UX product-side, ancorado no design system existente, estruturado pra ser **promovido a spec de componente** pela `ui-architect` |
| `feature-loop-acceptance-verifier` | fase 3 de aterrissagem (aceite) | verificação de aceite **independente e read-only** — cego ao build, adversarial, sem poder de editar. Garante "quem decide ≠ quem aprova" pela própria ferramenta |

A revisão de **defeito de código** roda nos checkpoints das builders (`review-go`, `code-review`) — a aterrissagem confere que rodou e pode pedir passada extra; achados que exigem código voltam pra builder.

## Arquivos

- `phases/01-intake.md` — refinamento de produto: história/épico, tipo, JTBD, valor, critérios de aceite, fatiamento, ambiguidades, mockup de UX
- `phases/02-ready.md` — Definition of Ready + handoff manual (a skill para aqui)
- `phases/03-land.md` — aterrissagem: aceite independente, auditoria de sincronização dos artefatos, footprint, commit com ID, verificação ao vivo
- `phases/removal.md` — protocolo de remoção (tipo remoção): motivo, análise reversa (na builder), registro no próprio CHG
- `templates/change-spec.md.tmpl` — o `CHG-NNN` (contrato entre raias; leve com campos de PM/PO reservados)
- `templates/backlog.md.tmpl` — o registro/inbox de mudanças
