---
name: ui-architect
description: "Desenha e constrói a UI de um sistema a partir de uma história de negócio (mini-mundo), opcionalmente consumindo artefatos de DDD. Use quando o usuário quer design-first para UI: refletir sobre personas, jornadas, jobs-to-be-done (discovery) → mapa de telas, navegação, wireframes em Excalidraw (IA) → design tokens, catálogo de componentes, spec por tela (design system) → implementação em cortes verticais por use case com testes em paralelo via subagents. Cada fase produz um artefato markdown versionável em docs/ui/. Triggers: 'desenhar a UI de X', 'arquitetar a interface de Y', 'construir frontend a partir do domínio', 'design-first UI', 'gerar UI a partir do DDD', ou invocação explícita da skill. Stack default Vite+React+TS+Tailwind+shadcn/ui, com override (next-app-router, remix, sveltekit, etc). Mobile nativo fora de escopo (vira skill irmã se vier demanda)."
---

# ui-architect

Constrói a UI de um sistema a partir de um mini-mundo, em quatro fases checkpointadas. Cada fase produz um artefato em `docs/ui/` (no projeto target, não na skill). Quando o projeto tem `docs/ddd/`, a skill consome esses artefatos como input. A implementação trabalha em **cortes verticais por use case**, não por camada horizontal.

## Fluxo

| Fase | Método | Artefato |
|---|---|---|
| 1. UX Discovery | `phases/01-discovery.md` | `docs/ui/01-discovery.md` |
| 2. IA & Fluxos | `phases/02-ia-flows.md` | `docs/ui/02-ia-flows.md` + diagramas Excalidraw |
| 3. Design system | `phases/03-design-system.md` | `docs/ui/03-design-system.md` |
| 4. Implementação | `phases/04-implementation.md` | código + `docs/ui/04-build-log.md` |

A fase 4 se organiza em sub-fases (4.0 fundação → 4.1..N cortes verticais por use case → 4.Z smoke).

## Protocolo de ativação

Ao ativar a skill:

1. **Detectar estado.** Verifique se `docs/ui/` existe no projeto target. Se sim, liste os artefatos presentes e pergunte: **retomar**, **revisar fase específica**, ou **recomeçar**. Pule fases já concluídas a menos que o usuário peça revisão.

2. **Detectar artefatos de DDD.** Verifique se `docs/ddd/` existe. Se sim, informe ao usuário quais artefatos foram encontrados e que serão usados como input nas fases correspondentes. Se não existe, a skill funciona standalone a partir do mini-mundo bruto — peça o mini-mundo se não foi fornecido.

3. **Obter o mini-mundo.** Se o usuário não forneceu e não há `docs/ddd/01-strategic.md`, peça. Sondar: atores, contextos de uso, dispositivos-alvo, restrições visuais.

4. **Detectar stack.** Default `vite-react-ts`. Se o usuário passou `stack=<nome>` ou mencionou outra stack, use-a. Leia `templates/stacks/<stack>.md`. Se o arquivo não existir, **pare** e peça ao usuário as convenções; escreva o arquivo de stack antes de prosseguir.

5. **Rodar fase a fase, com checkpoint.** Leia o arquivo de método da fase, conduza o trabalho em conversa, escreva o artefato usando o template correspondente, apresente, pause. Não avance sem ordem do usuário.

## Protocolo de checkpoint

Após cada fase:
- Salve o artefato no caminho documentado.
- Diga: "Salvei em `<path>`. Revise/edite o arquivo. Quando quiser seguir pra fase N+1, me diga **seguir**, ou aponte ajustes."
- Se o usuário editar o `.md` manualmente, a versão editada vira a fonte da verdade — a próxima fase lê do disco, não da memória da conversa.

## Regras transversais

- **Os artefatos são o contrato entre fases.** A fase N+1 nunca usa informação da fase N que não esteja escrita no artefato. Se o artefato está vago, conserte o artefato — não compense na próxima fase.
- **Não modifique artefatos de DDD.** Se identificar gap (ex.: JTBD sem use case correspondente, tela que precisa de comando ausente no tático), sinalize ao usuário em vez de inventar. Gaps de DDD são consertados via `ddd-architect`, não via `ui-architect`.
- **Seja explícito sobre incerteza.** Ao propor um modelo, decisão de IA ou componente, liste alternativas consideradas e o porquê da escolha. Tradeoffs vão no artefato, não só na conversa.
- **Não superdesenhe.** Calibre a complexidade pelo mini-mundo. Sistema pequeno não precisa de 5 personas e 10 jornadas. Cada fase explicita esse julgamento.
- **Não invente requisitos.** Se o usuário não disse, não escreva como se ele tivesse dito. Quando faltar info, pergunte.
- **Não desenhe telas antes da fase 2.** Discovery é sobre quem/quê/por quê. Telas vêm depois.

## Agents (dependências)

A fase 4 despacha agents dedicados em paralelo (não `general-purpose`). Eles devem estar instalados em `~/.claude/agents/`:

| Agent | Usado em | Status |
|---|---|---|
| `ui-component-test-generator` | 4.1..N | a criar |
| `ui-component-code-generator` | 4.1..N | a criar |

Se um agent obrigatório está faltando, a skill PARA e pede pra instalar antes de prosseguir. Os agents são versionados junto com a skill (mesmo repo).

## Arquivos

- `phases/01-discovery.md` — método de UX discovery (personas, JTBD, jornadas, requisitos NF, restrições)
- `phases/02-ia-flows.md` — método de IA e fluxos *(a criar)*
- `phases/03-design-system.md` — método de design system (tokens, catálogo, spec por tela) *(a criar)*
- `phases/04-implementation.md` — orquestrador da fase de código por cortes verticais *(a criar)*
- `templates/01-discovery.md.tmpl` — template do artefato de discovery
- `templates/stacks/vite-react-ts.md` — convenções da stack default *(a criar)*
