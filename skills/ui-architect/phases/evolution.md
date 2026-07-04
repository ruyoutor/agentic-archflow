# Modo evolução — uma fatia de UI sobre o sistema existente (entrada: CHG-NNN)

Brownfield. A `feature-loop` (PO) escreveu uma história em `docs/changes/CHG-NNN-<slug>.md` —
possivelmente com um **mockup de UX** apensado (do agent `ux-mockup-designer`) — e o usuário roteou
o card pra cá. Este modo responde, com o design system e o código à mão, **o que precisa mudar na
interface para que o pedido funcione** e **quais trade-offs considerar** — e só então constrói,
cirurgicamente, dentro da fatia.

As fases greenfield (1–4) não rodam aqui; são o repertório. Contratos do modo:
- **Editar, não reescrever.** Componente/tela existente se edita; reescrita só quando edição não resolve — e dito explicitamente.
- **Escopo = a fatia.** Os componentes/telas que ela toca. Redesign fora da fatia é demanda nova, não carona.
- **Sem commit.** A entrega é o diff + artefatos evoluídos + relatório; quem aterrissa é a `feature-loop`.
- **Suíte inteira + build, sempre.** A superfície testável existente é contrato — preservação é critério de aceite.

## E0 — Ler tudo antes de opinar

1. **A história:** o `CHG-NNN` completo — JTBD, contexto, critérios de aceite (incluindo preservação),
   ambiguidades 5b, e o **mockup de UX** (rotulado construível/aspiracional).
2. **O contrato de backend (gate duro — continua valendo):** OpenAPI real (+ ADRs/tático), inventário
   de capacidades e **não-capacidades**. Se a fatia é full-stack e o backend ainda não aterrissou o
   contrato novo, **pare** — o front nunca precede o contrato real.
3. **O modelo vivo de UI:** `docs/ui/03-design-system.md` (tokens, catálogo, specs), `02-ia-flows.md`,
   build-log.
4. **O código na vizinhança:** os componentes/telas/hooks que a fatia toca, e seus testes — inclusive
   os `data-testid`/copy/roles que testes existentes assertam (é superfície contratada).
5. **A stack:** `templates/stacks/<stack>.md` (default `vite-react-ts`).

## E1 — Análise de impacto & promoção do mockup

1. **Adjudicar o mockup contra o contrato.** Cada elemento do mockup rastreia a uma capacidade real?
   - **Construível** → segue pra promoção.
   - **Aspiracional** (excede o backend) → o excesso vira **demanda registrada** (backlog de
     `docs/ddd/` / novo CHG), nunca gambiarra nem renderização fingida. Diga explicitamente o que
     ficou de fora desta fatia.
2. **Promover o mockup a spec de componente** (`templates/component-spec.md.tmpl`): o mockup do
   `ux-mockup-designer` já traz estados, copy, proveniência e superfície testável **sugerida** — a
   promoção valida contra o contrato, decide reuso vs. componente novo, e **pina** props, hooks,
   `data-testid`/roles/copy. Spec novo em `docs/ui/specs/`, ou edição do spec existente quando a
   fatia altera componente que já tem um.
3. **O que precisa mudar** — escrito na seção 6 do CHG (parte de UI):
   - Telas/componentes: **novos** vs. **editados** (e o que a edição preserva — estados e superfície
     testável que continuam valendo).
   - Design system: reusa tokens/átomos? Estende o catálogo? Variação de componente existente ou
     componente novo (e por quê)?
   - Dados: hooks novos/alterados em `api.ts`, tipos da OpenAPI regenerados?
   - **O que NÃO muda** — proteção do critério de preservação.
4. **Trade-offs** — deliberação como sparring partner (opções, trade-offs, recomendação); resíduo no
   log de decisões do CHG. Ambiguidades 5b de UI todas endereçadas.
5. **Plano de build (seção 7):** calibragem (ver E2), ordem, footprint previsto.
6. **CHECKPOINT DURO — apresente e PARE.** Status → `em análise de impacto`, checkpoint `pendente`.
   "Segue" explícito → E2. Lacuna de **produto/UX** (história subespecificada, mockup inancorável) →
   anote no CHG, status → `em refinamento`, devolva o card à `feature-loop` e pare de vez.

## E2 — Construção cirúrgica (após o "segue"; status → `em construção`)

**Calibre o mecanismo pelo peso da fatia** — a calibragem mora aqui, não no PO:

| Peso | Caminho |
|---|---|
| Trivial (copy, estilo, ajuste de layout) | edite direto + rode a suíte. |
| Componente/tela **novo** | fluxo padrão da fase 4.1..N escopado: hooks em `api.ts` → stubs com `data-testid` do spec → `ui-component-code-generator` ∥ `ui-component-test-generator` (cegos, convergem pelo spec promovido). |
| Componente **existente alterado** | protocolo cirúrgico: atualize o **spec primeiro** (o que muda e o que fica na superfície testável) → despache os agents em modo edição (passe os arquivos existentes de código E teste, com a instrução de preservar o que o spec manteve) — ou edite direto se trivial. |

Registre a calibragem no build-log — é a lição da próxima fatia.

- **Integração com componentes existentes:** filho novo montado num pai já testado → atualize o
  `vi.mock` do teste do pai para o novo hook (tarefa do orquestrador, não dos agents — lição do dogfood 4.4).
- **Reconciliação:** `npm run build` + `vitest run` na suíte **completa**. Teste antigo vermelho é
  sinal de preservação violada; teste vs. código do corte divergindo é spec ambíguo — conserte a
  **causa**, nunca afrouxe o teste. Mudar asserção de teste existente é decisão explícita, registrada.
- **Proveniência:** cada elemento novo rastreia a um campo do contrato; nada renderiza sem fonte.

## E3 — Evoluir os artefatos + relatório de entrega

1. **Editar** (não regenerar): `docs/ui/03-design-system.md` (catálogo + tokens se tocados), specs em
   `docs/ui/specs/`, e `02-ia-flows.md` se a navegação mudou.
2. **Histórico de revisões:** entrada `{{data}} · CHG-NNN · o que mudou · por quê` em cada artefato
   tocado (crie a seção no rodapé se não existir).
3. **Build-log:** seção da fatia (modo evolução), com calibragem, proveniência, resultados.
4. **Relatório de entrega:** diff resumido, footprint real vs. previsto, suíte + build, screenshot ao
   vivo quando o backend estiver no ar. Status do CHG → `construída`.
5. **PARE. Sem commit.** Aterrissagem é da `feature-loop`.

## Regras

- Mockup é intenção; o spec promovido é o contrato. Não construa direto do mockup sem promover.
- Aspiracional nunca vira gambiarra — excesso é demanda registrada, dito com todas as letras.
- Superfície testável existente (testids/copy/roles assertados) só muda com decisão registrada.
- Escopo é a fatia. Redesign do vizinho vira demanda, não carona.
- Artefato dessincronizado no fim da E3 é entrega incompleta — a aterrissagem devolve.
