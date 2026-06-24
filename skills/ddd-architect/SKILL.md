---
name: ddd-architect
description: "Desenha e constrói sistemas de software a partir de uma história de negócio (mini-mundo), ancorado em Domain-Driven Design e arquitetura equilibrada. Use quando o usuário quer design-first: refletir sobre domínio, subdomínios, linguagem ubíqua, bounded contexts (estratégico) → entidades, value objects, agregados, repositórios, invariantes (tático) → decisões arquiteturais com tradeoffs (sync/async, outbox, CQRS, monolito vs distribuído) → geração incremental de código com testes em paralelo via subagents. Cada fase produz um artefato markdown versionável em docs/ddd/. Triggers: 'modelar X com DDD', 'desenhar a arquitetura de Y', 'construir um sistema partindo do domínio', 'design-first', 'gerar código a partir do domínio', ou invocação explícita da skill. Stack default Go, com override (java-spring, node-ts, etc)."
---

# ddd-architect

Constrói sistemas a partir de um mini-mundo, em quatro fases checkpointadas. Cada fase produz um artefato em `docs/ddd/` (no projeto target, não na skill). O usuário revisa cada artefato antes da próxima fase.

## Fluxo

| Fase | Método | Artefato |
|---|---|---|
| 1. Estratégico | `phases/01-strategic.md` | `docs/ddd/01-strategic.md` |
| 2. Tático | `phases/02-tactical.md` | `docs/ddd/02-tactical.md` |
| 3. Arquitetura | `phases/03-architecture.md` | `docs/ddd/03-architecture.md` |
| 4. Implementação | `phases/04-implementation.md` | código + `docs/ddd/04-build-log.md` |

A fase 4 se decompõe em sub-fases (4.0 contratos → 4.1 domínio ∥ → 4.2 aplicação ∥ → 4.3 adapters → 4.4 composition).

## Protocolo de ativação

Ao ativar a skill:

1. **Detectar estado.** Verifique se `docs/ddd/` existe no projeto. Se sim, liste os artefatos presentes e pergunte: **retomar**, **revisar fase específica**, ou **recomeçar**. Pule fases já concluídas a menos que o usuário peça revisão.

2. **Obter o mini-mundo.** Se o usuário não forneceu, peça. Sondar: atores, eventos de negócio, restrições, integrações, escala.

3. **Detectar stack.** Default Go. Se o usuário passou `stack=<nome>` ou mencionou outra stack, use-a. Leia `templates/stacks/<stack>.md`. Se o arquivo não existir, **pare** e peça ao usuário as convenções; escreva o arquivo de stack antes de prosseguir.

4. **Rodar fase a fase, com checkpoint.** Leia o arquivo de método da fase, conduza o trabalho em conversa, escreva o artefato usando o template correspondente, apresente, pause. Não avance sem ordem do usuário.

## Protocolo de checkpoint

Após cada fase ou sub-fase (4.0, 4.1, 4.2, 4.3.X, 4.4) — antes de propor commit:

1. **Atualizar `docs/ddd/04-build-log.md`** com a nova seção (campos mínimos: iniciada/concluída, modo, resultados build/test, arquivos criados/modificados, decisões tomadas, ambiguidades flagadas, ajustes pós-review). **Antes** do commit, não depois.

2. **Verificar build/test verde** — `go build/vet/test ./...` (ou equivalente da stack). Não propor commit com vermelho.

3. **Invocar revisões pré-commit** quando aplicável:
   - `cardpay-skills:review-go` — projetos Go (já no CLAUDE.md global).
   - `code-review` (skill do marketplace oficial) — sempre que houver mudança significativa (≥50 LOC, ≥3 arquivos, ou camada nova). Pedir `effort=high` pra cobertura ampla.
   - Apresentar achados ao usuário; aplicar os que ele aprovar; **documentar os mantidos com justificativa no build log**.

4. **Salvar artefato no caminho documentado** + apresentar ao usuário.

5. **Pausar** — não avançar sem ordem.

Se o usuário editar o `.md` manualmente, a versão editada vira a fonte da verdade — a próxima fase lê do disco, não da memória da conversa.

## Regras transversais

- **Os artefatos são o contrato entre fases.** A fase N+1 nunca usa informação da fase N que não esteja escrita no artefato. Se o artefato está vago, conserte o artefato — não compense na próxima fase.
- **Seja explícito sobre incerteza.** Ao propor um modelo ou decisão, liste alternativas consideradas e o porquê da escolha. Tradeoffs vão no artefato, não só na conversa.
- **Não superdesenhe.** Calibre a complexidade pelo mini-mundo. Sistema pequeno não precisa de 5 bounded contexts e outbox. A fase 3 explicita esse julgamento.
- **Não invente regras de negócio.** Se o usuário não disse, não escreva como se ele tivesse dito. Quando faltar info, pergunte.

## Agents (dependências)

A fase 4 despacha agents dedicados em paralelo (não `general-purpose`). Eles devem estar instalados em `~/.claude/agents/`:

| Agent | Usado em | Status |
|---|---|---|
| `ddd-domain-test-generator` | fase 4.1 | obrigatório |
| `ddd-domain-code-generator` | fase 4.1 | obrigatório |
| `ddd-app-test-generator` | fase 4.2 | obrigatório |
| `ddd-app-code-generator` | fase 4.2 | obrigatório |

Se um agent obrigatório está faltando, a skill PARA e pede pra instalar antes de prosseguir. Os agents são versionados junto com a skill (mesmo repo).

## Arquivos

- `phases/01-strategic.md` — método estratégico (domínio, subdomínios, linguagem ubíqua, bounded contexts, mapa de contextos)
- `phases/02-tactical.md` — método tático (entidades, VOs, agregados, repositórios, invariantes, eventos)
- `phases/03-architecture.md` — árvore de decisão arquitetural (topologia, comunicação, consistência, padrões de leitura, evolução)
- `phases/04-implementation.md` — orquestrador da fase de código
- `phases/04-0-contracts.md` — geração de contratos compartilhados
- `phases/04-1-domain.md` — domínio em paralelo (testes ∥ código via subagents)
- `phases/04-2-application.md` — aplicação em paralelo (testes ∥ código via subagents)
- `phases/04-3-adapters.md` — adapters (sequencial: código → testes de integração)
- `phases/04-4-composition.md` — composition root, main, smoke test
- `templates/01-strategic.md.tmpl` — template do artefato estratégico
- `templates/02-tactical.md.tmpl` — template do artefato tático
- `templates/03-architecture-adr.md.tmpl` — template do ADR de arquitetura
- `templates/stacks/go.md` — convenções Go (default)
