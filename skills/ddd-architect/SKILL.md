---
name: ddd-architect
description: "Desenha, constrói E EVOLUI sistemas de software ancorados em Domain-Driven Design e arquitetura equilibrada. Greenfield: de uma história de negócio (mini-mundo) via estratégico → tático → decisões arquiteturais com tradeoffs → geração de código com testes em paralelo via subagents; artefatos versionáveis em docs/ddd/. MODO EVOLUÇÃO (brownfield): recebe um CHG-NNN da feature-loop e responde 'o que precisa mudar?' e 'quais trade-offs?' (análise de impacto com checkpoint humano duro) antes de construir cirurgicamente a fatia e evoluir os artefatos com histórico de revisões — sem commit (a aterrissagem é da feature-loop). Triggers: 'modelar X com DDD', 'desenhar a arquitetura de Y', 'construir um sistema partindo do domínio', 'design-first', 'gerar código a partir do domínio', 'trabalhar a fatia CHG-NNN em modo evolução', 'analisar o impacto de uma mudança no domínio', ou invocação explícita da skill. Stack default Go, com override (java-spring, node-ts, etc)."
---

# ddd-architect

Constrói sistemas a partir de um mini-mundo, em quatro fases checkpointadas — e **evolui** sistemas já construídos em fatias, no modo evolução. Cada fase produz um artefato em `docs/ddd/` (no projeto target, não na skill). O usuário revisa cada artefato antes da próxima fase.

## Fluxo

Dois modos:

**Greenfield** (do mini-mundo à v1):

| Fase | Método | Artefato |
|---|---|---|
| 1. Estratégico | `phases/01-strategic.md` | `docs/ddd/01-strategic.md` |
| 2. Tático | `phases/02-tactical.md` | `docs/ddd/02-tactical.md` |
| 3. Arquitetura | `phases/03-architecture.md` | `docs/ddd/03-architecture.md` |
| 4. Implementação | `phases/04-implementation.md` | código + `docs/ddd/04-build-log.md` |

A fase 4 se decompõe em sub-fases (4.0 contratos → 4.1 domínio ∥ → 4.2 aplicação ∥ → 4.3 adapters → 4.4 composition).

**Evolução** (brownfield — entrada é um `CHG-NNN` da `feature-loop`):

| Fase | Método | Saída |
|---|---|---|
| E1. Análise de impacto | `phases/evolution.md` | seções 6–7 do CHG (**checkpoint humano duro**) |
| E2. Construção cirúrgica | `phases/evolution.md` | diff da fatia, suíte inteira verde |
| E3. Artefatos + relatório | `phases/evolution.md` | artefatos evoluídos com histórico de revisões; **sem commit** |

## Protocolo de ativação

Ao ativar a skill:

1. **Detectar o modo.** Se a entrada é um `CHG-NNN` em `docs/changes/` (handoff da `feature-loop`) ou o usuário pede pra evoluir/mudar/consertar um sistema que já tem `docs/ddd/` + código construído → **modo evolução**: leia `phases/evolution.md` e siga por ele (as fases 1–4 abaixo são o repertório, não o roteiro). Caso contrário, greenfield.

2. **Detectar estado (greenfield).** Verifique se `docs/ddd/` existe no projeto. Se sim, liste os artefatos presentes e pergunte: **retomar**, **revisar fase específica**, ou **recomeçar**. Pule fases já concluídas a menos que o usuário peça revisão.

3. **Obter o mini-mundo.** Se o usuário não forneceu, peça. Sondar: atores, eventos de negócio, restrições, integrações, escala.

4. **Detectar stack.** Default Go. Se o usuário passou `stack=<nome>` ou mencionou outra stack, use-a. Leia `templates/stacks/<stack>.md`. Se o arquivo não existir, **pare** e peça ao usuário as convenções; escreva o arquivo de stack antes de prosseguir.

5. **Rodar fase a fase, com checkpoint.** Leia o arquivo de método da fase, conduza o trabalho em conversa, escreva o artefato usando o template correspondente, apresente, pause. Não avance sem ordem do usuário.

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
- **Em modo evolução: editar, não reescrever; escopo = fatia; sem commit.** A análise de impacto (o que muda? que trade-offs?) precede qualquer código e **para no checkpoint humano**. Artefatos existentes são editados com entrada no histórico de revisões (`CHG-NNN`), nunca regenerados. A suíte **inteira** verde é o detector de preservação. Lacuna de produto na história devolve o card à `feature-loop` — não se remenda aqui.

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
- `phases/evolution.md` — modo evolução (brownfield): análise de impacto com checkpoint duro → construção cirúrgica → evolução de artefatos; entrada `CHG-NNN`, entrega sem commit
- `phases/04-0-contracts.md` — geração de contratos compartilhados
- `phases/04-1-domain.md` — domínio em paralelo (testes ∥ código via subagents)
- `phases/04-2-application.md` — aplicação em paralelo (testes ∥ código via subagents)
- `phases/04-3-adapters.md` — adapters (sequencial: código → testes de integração)
- `phases/04-4-composition.md` — composition root, main, smoke test
- `templates/01-strategic.md.tmpl` — template do artefato estratégico
- `templates/02-tactical.md.tmpl` — template do artefato tático
- `templates/03-architecture-adr.md.tmpl` — template do ADR de arquitetura
- `templates/stacks/go.md` — convenções Go (default)
