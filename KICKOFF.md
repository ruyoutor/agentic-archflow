# Kickoff — começar um projeto novo com as skills (design-first)

Guia pra arrancar um projeto **do zero**, num chat zerado de contexto, tendo um **mini-mundo**
em mãos. Usa as skills `ddd-architect` (backend) e `ui-architect` (frontend), que convergem
backend → contrato → frontend.

> **Onde isto fica no ciclo de vida:** o kickoff vai *uma vez*, do mini-mundo até a v1. Depois,
> o projeto não acaba — entra em **evolução contínua**, conduzida pela skill `feature-loop` (ver
> [seção 6](#6-depois-da-v1--regime-permanente-feature-loop)). Três skills, duas fases do ciclo:
> **arranque** (ddd + ui) → **regime permanente** (feature-loop).

> A `CLAUDE.md` global + a memória carregam sozinhas no chat novo — não precisa reexplicar
> convenções (commit atômico, build/testes antes de commitar, idioma, etc.).

## 1. Antes de abrir o chat (prep)

- [ ] **Mini-mundo** num `.md` acessível (raiz do projeto novo ou `~/Downloads/`).
- [ ] **Repo git** inicializado no projeto (`git init`) — pra commitar fase a fase.
- [ ] **Stack** decidida — default Go (backend) + `vite-react-ts` (frontend); senão, prepare o override.
- [ ] Agents instalados (`~/.claude/agents` → `build-skills/agents`, symlink do diretório): `ddd-*`, `ui-component-*` e `feature-loop-acceptance-verifier`. Nada a fazer.

## 2. Sequência recomendada (full-stack)

**Backend primeiro, frontend depois** — o gate do `ui-architect` exige o contrato real (OpenAPI).
Construir o backend antes dá um contrato de verdade pra desenhar contra, evitando a armadilha
aspiracional↔construível.

```
mini-mundo
  → ddd-architect:  estratégico → tático → arquitetura → código+testes (agents)  → docs/ddd/ + app + OpenAPI
  → ui-architect:   discovery → ia-flows → design-system (specs) → implementação (agents)  → web/
```

⚠️ Garanta uma **OpenAPI** ao fim do backend (ou ao menos `docs/ddd/` com tático+ADRs) — é o que
o gate da UI lê.

## 3. Prompts sugeridos (copiar e colar)

**Abertura — backend:**
> Quero construir um sistema design-first a partir de um mini-mundo, usando a skill `ddd-architect`.
> O mini-mundo está em `<caminho>`. Stack Go. Comece pela fase estratégica e **pare pra checkpoint**
> antes de avançar.

A cada fase, revise o artefato em `docs/ddd/` e responda **"seguir"** (ou aponte ajustes). Se a
OpenAPI não sair sozinha:
> Gere a OpenAPI (`docs/api/openapi.yaml`) a partir dos handlers/DTOs.

**Abertura — frontend (depois do backend):**
> Agora o frontend, design-first, com a skill `ui-architect`. O contrato está em
> `docs/api/openapi.yaml` e o domínio em `docs/ddd/`. Stack `vite-react-ts`. Rode do discovery,
> **checkpoint a cada fase**.

## 4. Todo list

```
[ ] DDD 1 — estratégico (subdomínios, linguagem ubíqua, bounded contexts)
[ ] DDD 2 — tático (agregados, VOs, invariantes)
[ ] DDD 3 — arquitetura (ADRs, decisões)
[ ] DDD 4 — código + testes (agents em paralelo)
[ ] OpenAPI gerada e commitada
[ ] UI 1 — discovery (personas, JTBD, jornadas)
[ ] UI 2 — IA & fluxos (mapa de telas, navegação)
[ ] UI 3 — design system (tokens + specs por componente)
[ ] UI 4.0 — fundação (scaffold, tokens→tailwind, types da OpenAPI)
[ ] UI 4.1..N — cortes verticais (code-gen + test-gen em paralelo)
[ ] UI 4.Z — smoke (build + suíte + verificação ao vivo)
```

## 5. Dicas (aprendidas na prática)

- **Checkpoint é sagrado:** cada fase pausa. Edite o artefato `.md` — ele é a fonte da verdade da fase seguinte.
- **Confie no gate:** se a UI parar pedindo o contrato, é de propósito.
- **Specs ricos = agents convergem.** Na Fase 3, capriche na *superfície testável* (props, estados,
  copy, `data-testid`). É o que faz code-gen e test-gen baterem sem se ver.
- **Reconciliação roda a suíte completa**, não só os arquivos do corte — divergência teste↔código é
  sinal, conserte a causa.
- As lições já estão embutidas nos agents/templates: âncora de toast no spec, formulário é
  assíncrono (await/waitFor), atualizar o mock do pai ao montar filho novo, datas de calendário em UTC.
- **Commit por fase/corte**; build + testes antes (regra global).

## 6. Depois da v1 — regime permanente (`feature-loop`)

Com a v1 de pé, **toda demanda emergente** — feature nova, mudança de comportamento, remoção de
feature, bug — vira uma **fatia vertical** conduzida pela skill `feature-loop`. Ela **não
reconstrói**: veste o chapéu de PM/PO + arquiteto e **delega** a construção de volta pra
`ddd-architect`/`ui-architect` em modo incremental, mantendo o modelo vivo (`docs/ddd`, `docs/ui`)
em sincronia.

```
demanda → feature-loop:  intake/triagem → impacto/deliberação → build (delega às builder) → aterrissagem
          (CHG-NNN)        valor+aceite     log de decisões        código+testes              aceite indep. + footprint + commit
```

- **Registro:** todas as mudanças vivem em `docs/changes/` — um `backlog.md` (inbox + priorização)
  e um `CHG-NNN-<slug>.md` por fatia (problema, critério de aceite, decisões, footprint).
- **Deliberação:** dúvidas de design/domínio são debatidas no chat, mas o *porquê* de cada escolha
  fica no **log de decisões** do CHG (mini-ADR por dúvida).
- **Aceite independente:** a aterrissagem despacha o agent `feature-loop-acceptance-verifier`
  (read-only, cego ao build) — separação de deveres "quem decide ≠ quem aprova". O orquestrador
  **adjudica** o parecer (o verificador pode errar).
- **Reversibilidade:** cada fatia entregue grava seu **footprint**, pra que *"remover a feature
  CHG-XXX"* seja uma operação barata depois (revert limpo vs. remoção cirúrgica).

**Prompt de abertura (chat novo, projeto existente):**
> Tenho uma demanda nova no `<projeto>` (já tem `docs/ddd`/`docs/ui`). Vamos conduzir pela skill
> `feature-loop`, começando pela triagem. A demanda é: `<descreva>`.

**Pra remover uma feature:**
> Vamos remover a feature `CHG-XXX` do `<projeto>`, usando a `feature-loop`. Motivo: `<por quê>`.

---

Referência viva do que isso produziu na prática: `performance-acoes` — arranque com `ddd-architect`
+ `ui-architect` (`docs/ddd/`, `docs/ui/04-build-log.md`), e o regime permanente com `feature-loop`
(`docs/changes/`, com o `CHG-001` como primeira fatia de evolução fim a fim).
