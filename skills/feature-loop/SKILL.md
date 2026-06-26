---
name: feature-loop
description: "Conduz a evolução de um sistema já construído em cortes verticais finos, no regime permanente — uma mudança de cada vez, do 'por que isso vale a pena' até 'entregue, verde, artefatos sincronizados, commitado'. Veste o chapéu de PM/PO (triagem, valor, critério de aceite, fatiamento) + arquiteto (análise de impacto contra o modelo vivo) e DELEGA a construção às skills builder (ddd-architect / ui-architect) em modo incremental — não reimplementa. Use quando o sistema já tem uma v1 e surge demanda emergente: feature nova, mudança de comportamento, remoção de feature, ou bug. Triggers: 'adicionar feature a um sistema existente', 'mudar o comportamento de X', 'remover a feature Y', 'gerir o backlog/evolução do produto', 'corte vertical de uma mudança', ou invocação explícita da skill. Pressupõe docs/ddd e/ou docs/ui existentes como modelo vivo."
---

# feature-loop

Orquestra a evolução de um sistema **já construído** em cortes verticais, em loop. Cada volta entrega **uma** mudança fim-a-fim. Não é uma skill de construção greenfield — ela **delega** a construção a `ddd-architect` / `ui-architect` em modo incremental e cuida do que essas skills não cobrem: a triagem PM/PO, a análise de impacto contra o modelo vivo, e a aterrissagem com os artefatos sincronizados.

Pré-requisito: o projeto já tem `docs/ddd/` e/ou `docs/ui/` (o modelo vivo). Para arrancar do zero, use o KICKOFF + as duas skills builder; volte aqui no regime permanente.

## O ciclo (uma volta = uma fatia vertical)

| Fase | Chapéu | Método | Artefato |
|---|---|---|---|
| 1. Intake & triagem | PM/PO | `phases/01-intake.md` | `docs/changes/CHG-NNN-<slug>.md` (problema, valor, critério de aceite, fatia) |
| 2. Análise de impacto | Arquiteto | `phases/02-impact.md` | seção de impacto no mesmo `CHG-NNN` |
| 3. Construção | Builder | `phases/03-build.md` | código + testes (delega às skills builder) |
| 4. Aterrissagem | PM/PO + eng | `phases/04-land.md` | artefatos vivos sincronizados + commit + verificação |

O registro de todas as mudanças vive em `docs/changes/backlog.md` (inbox + priorização). Cada fatia tem um `CHG-NNN` único, qualquer que seja o tipo.

## Protocolo de ativação

1. **Detectar o modelo vivo.** Verifique `docs/ddd/` e `docs/ui/`. Se nenhum existe, isto não é regime permanente — aponte o KICKOFF e pare. Liste o que encontrou (tático, ADRs, OpenAPI, design system) — é a base da análise de impacto.

2. **Detectar o registro.** Verifique `docs/changes/`. Se não existe, crie na primeira fatia (`backlog.md` + primeiro `CHG`). Se existe, mostre o backlog e pergunte: **nova mudança**, **retomar um CHG em andamento**, ou **só triar o inbox**.

3. **Capturar a demanda.** Se o usuário não descreveu a mudança, peça. Não classifique o tipo por conta própria a partir do enunciado — conduza a fase 1, que classifica explicitamente.

4. **Rodar fase a fase, com checkpoint.** Uma fatia por vez. Não inicie uma nova fatia com outra em andamento sem o usuário mandar.

## Protocolo de checkpoint

Após cada fase de uma fatia:
- Atualize o `CHG-NNN` (a fase preenche sua seção) e o status no `backlog.md`.
- Na fase 3+, build/test **verde** antes de propor commit (regra global). Invoque as revisões pré-commit aplicáveis (`review-go` em Go; `code-review` em mudança significativa).
- Apresente, **pause**. Não avance sem ordem.
- Se o usuário editar o `CHG-NNN` à mão, a versão em disco vira a fonte da verdade da fase seguinte.

## Regras transversais

- **A fatia é o menor vertical que entrega valor.** Se a mudança não cabe numa volta, quebre em fatias sequenciais no backlog — cada uma fechável sozinha. Não abra frente larga.
- **O modelo vivo é a fonte da verdade — sincronize, não duplique.** A fase 2 lê `docs/ddd`/`docs/ui`; a fase 4 os **atualiza** (tático, ADR, design system, build-log). Mudança que diverge do que está escrito é sinal: conserte o artefato.
- **Preserve o que existe, salvo ordem explícita.** Feature nova ou mudança de comportamento não quebra o que já funciona sem o usuário decidir o trade-off. Remoção é deliberada e segue seu próprio fluxo — ver `phases/removal.md`.
- **Reversibilidade é conquistada na construção, não na remoção.** Toda fatia entregue registra seu **footprint** (fase 4, seção 9 do CHG): o que ela adicionou/tocou, com precisão suficiente pra ser removida depois sem arqueologia. É isso que torna *"remover a feature CHG-XXX"* uma operação barata e honesta — o protocolo de remoção lê o footprint pra decidir entre revert limpo e remoção cirúrgica.
- **Não invente regra de negócio.** Ambiguidade de domínio que aparece na fatia (ex.: "o que entra no cálculo quando filtro X?") é registrada na fase 1 e **decidida com o usuário** na fase 2 — não chutada no código.
- **Delibere como sparring partner, registre o resíduo.** Dúvidas surgem no discovery/desenho e isso é o normal, não exceção. O debate é conversa livre — mas você **traz opções, expõe tradeoffs e recomenda**, não só anota o que o usuário decide. Cada dúvida resolvida vira uma entrada no **log de decisões** do `CHG-NNN` (pergunta → opções pesadas → decisão → porquê). É um mini-ADR por mudança: daqui a meses, o *porquê* da escolha ainda está lá. Dúvida de produto ("é esse o corte certo?") se delibera na fase 1; dúvida de impacto ("dado o modelo, como X interage?") na fase 2 — ambas deixam registro.
- **Não superdesenhe a cerimônia.** Projeto pequeno usa o change-spec leve. Os campos de PM/PO pleno (prioridade, épico, esforço, métrica) ficam no template, vazios, até a ambição crescer.
- **Delegue a construção, não a reescreva.** A fase 3 chama as skills builder em modo incremental sobre a fatia. Esta skill não gera domínio/UI por conta própria.
- **Separe o julgamento, não os dedos. O humano é a autoridade de merge.** Que o agente comite/edite não é o problema — em time humano o que separa PM de eng não é "quem digita", é **separação de deveres: quem decide não é quem aprova**. Honre isso com *verificação independente* (fase 4: aceite por agent fresco, cego ao build) + o *gate humano* sobre toda ação externa (commit/push/PR). Escala com a autonomia: quanto menos humano no loop, mais o aceite independente vira veto formal.

## Tipos de mudança (a fase 1 classifica em um destes)

| Tipo | Característica | Nuance de fluxo |
|---|---|---|
| Feature nova | capacidade inexistente | ciclo completo A→D; pode tocar domínio + contrato + UI |
| Mudança de comportamento | feature existe, regra muda | foco na fase 2 — costuma tocar invariante/ADR; preservar ou versionar o comportamento antigo é decisão |
| Remoção | tirar feature | variante do ciclo com impacto **reverso** — ver `phases/removal.md`. Lê o footprint do CHG, classifica limpo (revert) vs. entrelaçado (cirúrgico), registra o motivo e a remoção **no próprio CHG** |
| Bug | comportamento errado | registra no log de bugs do projeto; fatia geralmente fina; foco no teste de regressão |

## Delegação às skills builder (modo incremental)

A fase 3 invoca `ddd-architect` e/ou `ui-architect` passando o `CHG-NNN` como entrada. As builder operam **só sobre a fatia**: tocam o agregado/use-case/componente afetado, rodam os agents da fase 4 delas só nesses arquivos, e **editam** (não reescrevem) os artefatos. Esse gancho "modo incremental" nas builder é a costura entre o orquestrador e elas — `phases/03-build.md` define o contrato de entrada/saída.

## Agents (dependências)

A fase 4 despacha um agent dedicado pra **separação de deveres** (instalado em `~/.claude/agents/`):

| Agent | Usado em | Papel |
|---|---|---|
| `feature-loop-acceptance-verifier` | fase 4 (aceite) | verificação de aceite **independente e read-only** — cego ao build, adversarial, sem poder de editar. Garante "quem decide ≠ quem aprova" pela própria ferramenta. |

A revisão de **defeito de código** não tem agent próprio — delega às skills `review-go` (Go) e `code-review` (diff), já existentes. A **construção** (fase 3) delega aos agents das skills builder (`ddd-*`, `ui-component-*`), não a agents desta skill.

## Arquivos

- `phases/01-intake.md` — método de intake & triagem (classificar tipo, JTBD, valor, critério de aceite, fatiamento, ambiguidades)
- `phases/02-impact.md` — método de análise de impacto contra o modelo vivo (camadas, invariantes/ADR, contrato, DEM)
- `phases/03-build.md` — orquestração da construção: contrato de delegação às skills builder em modo incremental
- `phases/04-land.md` — aterrissagem: sincronizar artefatos vivos, registrar footprint, review, commit com ID, verificação ao vivo
- `phases/removal.md` — protocolo de remoção (tipo remoção): impacto reverso, classificação limpo/entrelaçado, registro no próprio CHG
- `templates/change-spec.md.tmpl` — o `CHG-NNN` (contrato entre as fases da fatia; leve com campos de PM/PO reservados)
- `templates/backlog.md.tmpl` — o registro/inbox de mudanças
