# agentic-archflow

**Arquitetura em fluxo, conduzida por agentes.** Um conjunto de skills e agents para o
[Claude Code](https://docs.claude.com/en/docs/claude-code) que leva um sistema de software de uma
**história de negócio** (mini-mundo) até uma **v1 rodando** — e daí em diante, mantém a evolução
contínua em cortes verticais, no estilo de um fluxo ágil/kanban.

O fio condutor: **decisões de arquitetura (de software e sistêmica), engenharia, design e modelagem
com excelência, sem perda entre as camadas** — o domínio, a arquitetura, a UI e a evolução ficam
amarrados por artefatos versionáveis que são a fonte da verdade de cada fase.

## O que tem aqui

Três skills, cobrindo as duas fases do ciclo de vida de um sistema:

| Skill | Fase | O que faz |
|---|---|---|
| **`ddd-architect`** | Arranque (backend) | Do mini-mundo ao código: estratégico → tático → arquitetura → implementação. Ancorado em Domain-Driven Design. Gera código + testes em paralelo via subagents. Artefatos em `docs/ddd/`. |
| **`ui-architect`** | Arranque (frontend) | Do mini-mundo (+ contrato do backend) à UI: discovery → IA & fluxos → design system → implementação em cortes verticais. Artefatos em `docs/ui/`. |
| **`feature-loop`** | Regime permanente | Depois da v1, toda demanda emergente (feature, mudança, remoção, bug) vira uma fatia vertical. Veste o chapéu de PM/PO + arquiteto e **delega** a construção de volta às skills builder. Registro em `docs/changes/`. |

E os agents que as skills despacham em paralelo (em `agents/`):

- `ddd-domain-{code,test}-generator`, `ddd-app-{code,test}-generator` — camadas de domínio e aplicação
- `ui-component-{code,test}-generator` — componentes de UI
- `feature-loop-acceptance-verifier` — aceite independente, read-only, cego ao build (separação de deveres "quem decide ≠ quem aprova")

## Princípios

- **Design-first.** Reflete sobre o domínio antes de escrever código. Cada fase produz um artefato markdown revisável.
- **Checkpoint é sagrado.** Cada fase pausa para revisão humana. O artefato editado vira a fonte da verdade da fase seguinte.
- **Artefatos são o contrato entre fases.** A fase N+1 nunca usa informação que não esteja escrita. Artefato vago? Conserta o artefato — não compensa depois.
- **Cortes verticais, não camadas horizontais.** Cada volta entrega uma mudança fim-a-fim, verde e commitada.
- **Não superdesenhe.** A complexidade é calibrada pelo mini-mundo, com tradeoffs explícitos.
- **Modelo vivo.** `docs/ddd`, `docs/ui` e `docs/changes` evoluem junto com o sistema — nunca ficam desatualizados.

## Como usar

### Instalação

As skills e agents vivem neste repo. Aponte o Claude Code para eles via symlink no seu diretório de configuração:

```bash
# Skills
ln -s "$(pwd)/skills"/* ~/.claude/skills/

# Agents (as fases de implementação os despacham em paralelo)
ln -s "$(pwd)/agents"/*.md ~/.claude/agents/
```

> Ajuste os caminhos conforme sua instalação. Os agents precisam estar em `~/.claude/agents/` — as skills param e pedem instalação se faltar algum obrigatório.

### Arrancar um projeto novo (do zero até a v1)

Veja o **[`KICKOFF.md`](./KICKOFF.md)** — o guia passo a passo, com a sequência recomendada
(backend → contrato → frontend), prompts prontos para copiar e colar, e a todo list.

Resumo do fluxo:

```
mini-mundo
  → ddd-architect:  estratégico → tático → arquitetura → código+testes  → docs/ddd/ + app + OpenAPI
  → ui-architect:   discovery → IA & fluxos → design system → implementação  → web/
```

### Evoluir um projeto existente (regime permanente)

Com a v1 de pé, abra um chat novo e conduza pela `feature-loop`:

> Tenho uma demanda nova no `<projeto>` (já tem `docs/ddd`/`docs/ui`). Vamos conduzir pela skill
> `feature-loop`, começando pela triagem. A demanda é: `<descreva>`.

```
demanda → feature-loop:  intake/triagem → impacto/deliberação → build (delega às builder) → aterrissagem
          (CHG-NNN)        valor+aceite     log de decisões        código+testes              aceite indep. + footprint + commit
```

## Estrutura do repo

```
skills/
  ddd-architect/     # backend design-first (DDD)
  ui-architect/      # frontend design-first
  feature-loop/      # evolução contínua em cortes verticais
agents/              # subagents despachados pelas fases de implementação
docs/design/         # notas de design das próprias skills
KICKOFF.md           # guia para arrancar um projeto novo
```

## Stacks

- **Backend:** default Go, com override (java-spring, node-ts, etc.) via arquivo de stack.
- **Frontend:** default Vite + React + TS + Tailwind + shadcn/ui, com override (next-app-router, remix, sveltekit, etc.).

---

Idioma dos artefatos e da condução: **português brasileiro**.
