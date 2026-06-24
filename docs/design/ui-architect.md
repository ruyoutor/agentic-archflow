# Design — skill `ui-architect`

> **Status:** rascunho de design v2. Não é a skill ainda — é o documento que precede a implementação dela.

## Resumo

Skill design-first para **pensar, arquitetar e implementar a UI** de um sistema a partir do mini-mundo de negócio, espelhando o padrão da `ddd-architect`: fases checkpointadas, artefatos markdown versionáveis em `docs/ui/`, subagents dedicados para a geração de código.

Quando `docs/ddd/` existe no projeto, a skill **consome** os artefatos de DDD como input (bounded contexts → áreas do produto; use cases → telas/jornadas; linguagem ubíqua → labels). Quando não existe, a skill funciona standalone a partir do mini-mundo bruto.

## Premissas

- A skill produz **estrutura, especificação e código** — não substitui design visual humano em projetos que tenham essa exigência.
- Excalidraw é a ferramenta de prototipação default (diagramas de fluxo e wireframes low-fi), via MCP `excalidraw` disponível no ambiente.
- Stack default opinativa, com override.
- Cada fase tem checkpoint humano. A versão editada do `.md` é a fonte da verdade pra fase seguinte.
- Implementação trabalha em **cortes verticais por use case**, não por camada horizontal.
- **Input flexível na fase 1:** aceita mini-mundo narrativo (exploratório, default) ou spec/PRD estruturado (validador). Ver seção "Modos de input".

## Não-objetivos (v1)

- **Mobile** (iOS / Android / React Native / Flutter). Fora permanente da skill principal — se vier demanda, vira skill irmã `ui-architect-mobile`, não override de stack.
- Geração automática de Figma (via API/plugin).
- Design visual hi-fi automático (paleta, marca, ilustração).
- Acessibilidade automatizada além do mínimo declarado no design brief (semântica HTML correta, contraste mínimo via tokens, foco visível).

## Fluxo

| Fase | Método | Artefato |
|---|---|---|
| 1. UX Discovery | `phases/01-discovery.md` | `docs/ui/01-discovery.md` |
| 2. IA & Fluxos | `phases/02-ia-flows.md` | `docs/ui/02-ia-flows.md` + diagramas Excalidraw |
| 3. Design system | `phases/03-design-system.md` | `docs/ui/03-design-system.md` |
| 4. Implementação | `phases/04-implementation.md` | código + `docs/ui/04-build-log.md` |

A fase 4 é organizada em **cortes verticais por use case** (espelhando o trabalho por bounded context do DDD): fundação única + um corte por use case + smoke final.

## Detalhamento das fases

### Fase 1 — UX Discovery

**Entrada:** mini-mundo OU spec/PRD estruturado (ver "Modos de input" abaixo) + (opcional) `docs/ddd/01-strategic.md`
**Saída:** `docs/ui/01-discovery.md`

Método (conduzido em conversa antes de escrever o artefato):

1. Identificar **personas** — quem usa, em que contexto, qual frequência. Quando há DDD estratégico, cruzar com atores do mini-mundo.
2. Mapear **jobs-to-be-done** por persona. Quando há use cases no tático, cada use case vira pelo menos um JTBD.
3. Esboçar **jornadas** principais — sequência de passos, pontos de fricção, momentos de decisão.
4. Listar **requisitos não-funcionais de UX** — performance percebida, offline-first, multi-tenant, idioma, acessibilidade mínima.
5. Listar **restrições** — branding existente, design system corporativo, dispositivos-alvo (default: desktop + mobile responsivo; mobile nativo fora de v1).

*Checkpoint:* jornadas e personas batem com o negócio? Algum JTBD não foi coberto pelo DDD (sinal de gap no tático)?

### Fase 2 — Information Architecture & Fluxos

**Entrada:** `docs/ui/01-discovery.md` + (opcional) `docs/ddd/02-tactical.md`
**Saída:** `docs/ui/02-ia-flows.md` + diagramas em Excalidraw

Método:

1. **Mapa de telas** — lista hierárquica. Quando há tático, cada agregado/use case sugere telas; cada bounded context pode virar uma área do produto.
2. **Navegação** — como o usuário transita (top nav, side nav, tabs, modais). Heurística: se a navegação não tem nome óbvio, a IA está confusa.
3. **Fluxos por jornada** — para cada jornada da fase 1, sequência de telas + decisões. Diagramas obrigatórios via Excalidraw MCP.
4. **Wireframes por tela-chave** — low-fi em Excalidraw. Foco em hierarquia de informação, regiões de ação, estados (vazio, erro, carregando, sucesso). Sem visual hi-fi.
5. **Componentes recorrentes detectados** — lista de blocos que aparecem em múltiplas telas (tabela, card, formulário X, modal Y). Input pra fase 3.

*Checkpoint:* fluxos cobrem todos os JTBD? Wireframes deixam clara a hierarquia? Componentes recorrentes catalogados?

### Fase 3 — Design system

**Entrada:** `docs/ui/02-ia-flows.md` + (opcional) `docs/ddd/01-strategic.md`
**Saída:** `docs/ui/03-design-system.md`

Método:

1. **Design tokens** — cores semânticas (primary, danger, surface…), tipografia, espaçamentos, raios, sombras, breakpoints. Sem branding existente, partir dos defaults do shadcn/ui e ajustar.
2. **Catálogo de componentes** — para cada componente recorrente: nome, props essenciais, variantes, estados, comportamento. Mapear para primitivas do shadcn/ui quando possível.
3. **Spec por tela** — árvore de componentes em markdown estruturado, conteúdo (labels reais usando linguagem ubíqua), bindings de dados (ex.: `pedido.total`), estados.
4. **(Opcional) Refino visual humano** — se há designer no time, o spec + os wireframes em Excalidraw servem como starting point pro refino visual em Figma (ou outra ferramenta). A skill não orquestra esse loop; o artefato deixa explícito que esse caminho existe e descreve como reentrar (tokens atualizados → fase 4.0).

*Checkpoint:* tokens e componentes cobrem todas as telas? Labels usam linguagem ubíqua? Pronto para implementação ou vai passar por refino visual antes?

### Fase 4 — Implementação (cortes verticais)

**Entrada:** os três artefatos em `docs/ui/` + arquivo de stack
**Saída:** código rodável + `docs/ui/04-build-log.md`

Estrutura:

| Sub-fase | Conteúdo | Modo | Frequência |
|---|---|---|---|
| 4.0 Fundação | Tipos compartilhados, tokens em código, app shell, routing skeleton, componentes base catalogados | sequencial | uma vez |
| 4.1..N Corte vertical | Por use case: tela(s) + componentes específicos + chamada de backend + testes | paralelo (testes ∥ código) via subagents | um por use case |
| 4.Z Smoke | Smoke test end-to-end percorrendo os cortes implementados | sequencial | uma vez no final |

Em cada corte vertical (4.x), o orquestrador despacha `ui-component-test-generator` ∥ `ui-component-code-generator` em paralelo (mesma mensagem), igual o DDD faz por bounded context. Eles não se veem e convergem nos contratos de props/tipos gerados em 4.0 + adições do corte em andamento.

**Checkpoints totais** = 1 (fundação) + N (cortes) + 1 (smoke). App com 3 use cases = 5 checkpoints. App com 1 use case (corte mínimo viável) = 3 checkpoints.

Protocolo de divergência teste ∥ código: idêntico ao da fase 4.1 do DDD — listar mismatches, identificar lado que bate com a spec (fase 3 como árbitro), pedir aprovação humana antes de reconciliar.

## Stack default

**Decidido:** **Vite + React + TypeScript + Tailwind CSS + shadcn/ui**, testes com **Vitest + React Testing Library**, e-2-e opcional com **Playwright**.

Por que esse default:
- **Vite** é simples de operar (`vite dev`, `vite build`); sem magia de framework full-stack.
- **React + TS** é o ecossistema com mais material e melhor representação em modelos generativos — agentes acertam mais.
- **Tailwind** elimina CSS manual e mantém tokens consistentes via config.
- **shadcn/ui** é copy-paste em vez de dependência runtime — você possui os componentes, fáceis de adaptar. Componentes acessíveis (sobre Radix) por padrão.
- **Vitest + RTL** é o caminho ergonômico moderno para teste de componente.

**Override por stack** (mesma mecânica do DDD): `stack=next-app-router` para SSR/SEO, `stack=remix`, `stack=sveltekit`, etc. Cada stack em `templates/stacks/<stack>.md` define layout, padrão de componente, convenção de teste e comandos build/lint. Mobile **não** entra como stack — vira skill irmã se vier demanda.

## Integração com `docs/ddd/`

Quando `docs/ddd/` existe no projeto target, a skill lê:

- `01-strategic.md` → personas (atores), linguagem ubíqua (labels), bounded contexts (áreas do produto)
- `02-tactical.md` → use cases (telas/jornadas), comandos (formulários/ações), eventos (notificações de UI, otimismo)
- `03-architecture.md` → estilo de comunicação (REST/gRPC/eventos → data fetching strategy), CQRS (telas de leitura otimizadas)

A skill **não** modifica artefatos de DDD. Se identifica gap (ex.: jornada da UI precisa de use case ausente no tático), sinaliza ao usuário em vez de inventar.

## Modos de input

A fase 1 aceita dois formatos de input, com comportamentos diferentes da skill. O modo é detectado pelo formato do que o usuário fornece (texto narrativo vs. documento estruturado com seções).

### Modo A — Mini-mundo (narrativa, exploratório)

Default quando o domínio é novo, está sendo modelado pela primeira vez, ou ainda há tensões/ambiguidades a destilar.

- **Entrada:** texto narrativo do negócio (atores, eventos, regras, restrições, integrações), na linguagem do negócio.
- **Comportamento da skill:** conduz discovery exploratória — restate, perguntas pontuais, identifica gaps, destila em personas/JTBD/jornadas. Conflitos e ambiguidades aparecem durante a destilação.
- **Saída:** `docs/ui/01-discovery.md` preenchido após conversa.

### Modo B — Spec / PRD (estruturado, validador)

Default quando o time já fez o trabalho de destilação fora da skill (PRD existente, design doc, RFC, spec à la Kiro/Spec Kit). O input já chega formalizado.

- **Entrada:** documento estruturado com requisitos, casos de uso, personas, critérios de aceitação (qualquer subset do template da fase 1).
- **Comportamento da skill:** **não** repete a discovery exploratória do zero. Lê o spec, mapeia pro template da fase 1, **sinaliza lacunas** vs. o que o template espera, e pede confirmação ao usuário item por item antes de aceitar como `01-discovery.md`.
- **Risco a mitigar:** specs chegam com decisões implícitas que ninguém debateu. A skill deve perguntar "por que" em pontos que parecem assumidos sem rastro — não só transcrever. O valor da discovery é fazer a fricção aparecer; se o spec já a removeu, a skill precisa reabri-la cirurgicamente.
- **Saída:** `docs/ui/01-discovery.md` preenchido a partir do spec + perguntas de validação respondidas.

### Os dois modos não são exclusivos

Um time pode trazer mini-mundo para discovery (fase 1) e ter spec formal mais tarde, ou começar com spec e voltar a explorar quando descobrir gap. A skill detecta o modo pelo input recebido na ativação.

### Propagar para `ddd-architect`

A mesma melhoria cabe na `ddd-architect` (fase 1 estratégica aceita mini-mundo OU spec/PRD). Registrar como item futuro para manter as duas skills coerentes. **[?]** Quando essa skill for implementada, abrir issue/tarefa pra retroaplicar na `ddd-architect`.

## Subagents previstos

| Agent | Usado em | Status |
|---|---|---|
| `ui-component-test-generator` | 4.1..N | a criar |
| `ui-component-code-generator` | 4.1..N | a criar |

Mesmo padrão dos agents de DDD: dois agents paralelos em mesma mensagem, não se veem, convergem via contratos compartilhados (props/tipos da 4.0 + adições do corte vertical em andamento).

## Tratamento de Excalidraw

O MCP `excalidraw` está disponível no ambiente e é **obrigatório** na fase 2 para:
- Diagramas de fluxo por jornada
- Wireframes low-fi por tela-chave

Excalidraw cobre diagramas + prototipação suficiente para o escopo dessa skill — sem necessidade de Figma no caminho default. Refino visual em Figma é caminho opcional pós-fase 3, fora da orquestração da skill.

## Riscos e questões abertas

1. **Estética do default.** Tailwind + shadcn implica visual utilitário/neutro. Times com identidade visual forte vão sobrescrever tokens — o doc da fase 3 deixa isso explícito.
2. **Design system existente.** Se o time já tem tokens/componentes corporativos, a fase 3 precisa modo "consumir" em vez de "gerar". **[?]** Adicionar fase opcional `0 — auditar design system existente`? Decisão adiada pra v2.
3. **Tamanho do corte vertical.** Use case pequeno pode não justificar checkpoint próprio. Mitigação: o orquestrador detecta cortes pequenos relacionados e oferece agrupar.
4. **Loop com designer humano.** Quando o time opta pelo refino visual após fase 3, o retorno (tokens, componentes ajustados) precisa de protocolo claro pra entrar na 4.0 sem reabrir tudo. Esboçar esse protocolo no `phases/04-implementation.md` antes de escrever a skill.

## Aprendizados da validação inicial

Primeira execução da fase 1 com mini-mundo real (sistema pessoal de acompanhamento de performance de ações vs. índices, 2026-05-27, sem `docs/ddd/` no target). Três observações pra iterar antes de implementar a fase 2.

### 1. "Modo A enriquecido" funciona, mas não está documentado

O mini-mundo trazido pelo usuário já tinha regras de negócio formalizadas (RN-01..RN-07), decisões fechadas e fora-de-escopo explícito — meio-termo entre Modo A puro (narrativa) e Modo B (spec). A skill aproveitou o que já estava destilado e fez discovery cirúrgica só dos pontos faltantes (frequência, contexto físico, motivação dominante, estado emocional, tema).

**Ação:** documentar essa heurística em `phases/01-discovery.md` — quando o input está em meio-termo, o agente identifica o que falta vs o template e pergunta **só isso**, em vez de refazer trabalho já destilado. Isso é uma terceira modalidade implícita ("Modo A enriquecido") que vale registrar formalmente, possivelmente como degrau intermediário no protocolo de detecção de modo.

### 2. Seção "Cruzamento com DDD" fica órfã quando não há DDD

Sem `docs/ddd/` no target, a seção foi preenchida com nota de "não aplicável". Funciona, mas polui o artefato sem agregar.

**Ação:** o método deve **omitir** a seção quando `docs/ddd/` não existe, não preencher com nota de ausência. Ajustar `phases/01-discovery.md` (instrução condicional) + template (marcar a seção como opcional).

### 3. Separar "ambiguidades de domínio" de "decisões diferidas por fase"

A seção "Ambiguidades / decisões pendentes" no template atual misturou três coisas distintas no caso real:

- **Restrições que avisam fases seguintes** (ex.: "não inventem seletor de período — janela é sempre data do lançamento → hoje").
- **Decisões diferidas pra fase 2** (ex.: drill-down via rota vs expansão in-place).
- **Decisões diferidas pra fase 3** (ex.: formato do indicador de dado desatualizado, persistência da troca de índice).

Sem separação, a fase 2 não consegue filtrar rapidamente quais itens são responsabilidade dela.

**Ação:** quebrar a seção do template em duas:
1. **Restrições e notas pra fases seguintes** — avisos que não são decisão, são "não invente isso".
2. **Decisões diferidas**, com sub-listas por fase alvo (fase 2, fase 3, fase 4).

### 4. Protocolo de "revisão por enriquecimento" não está formalizado

Cenário identificado durante o planejamento da continuação da validação: a fase 1 foi executada **sem** `docs/ddd/`, e depois o usuário pretende rodar `ddd-architect` no mesmo projeto e voltar pra `ui-architect`. Quando voltar, há `docs/ui/01-discovery.md` existente + `docs/ddd/` novo. O comportamento desejado **não** é "recomeçar fase 1" (joga fora trabalho) nem "pular fase 1 e ir pra fase 2" (perde o cruzamento que ficou disponível), mas **revisar cirurgicamente** o artefato existente: ler o `01-discovery.md`, ler artefatos DDD, gerar a seção "Cruzamento com DDD" + identificar gaps (JTBD sem use case, ator sem persona), apresentar diff ao usuário, e inserir no artefato existente preservando o resto.

**Ação:** formalizar no SKILL.md um sub-modo "revisar pra incorporar DDD". Quando o protocolo de ativação detectar `docs/ui/<artefato>.md` existente **+** `docs/ddd/` novo desde a última execução, oferecer essa opção explicitamente, além de "retomar", "revisar fase específica" e "recomeçar". A detecção de "DDD novo" pode ser por mtime do `01-discovery.md` vs mtime dos artefatos de DDD; se houver dúvida, perguntar.

## Próximos passos

1. ~~Validar este design.~~ **concluído**
2. ~~Implementar `SKILL.md` + `phases/01-discovery.md` + template do artefato 1.~~ **concluído**
3. ~~Validar a fase 1 com um mini-mundo real.~~ **concluído** (2026-05-27, `~/Documents/Pessoal/workspace/performance-acoes/`)
4. ~~Aplicar aprendizados 1, 2 e 3 na fase 1.~~ **concluído** (2026-05-27)
5. **Formalizar "revisão por enriquecimento"** (aprendizado 4) no protocolo de ativação do `SKILL.md`, antes ou durante a próxima sessão de validação.
6. Implementar `phases/02-ia-flows.md` + template do artefato 2.
7. Continuar a validação no projeto real `~/Documents/Pessoal/workspace/performance-acoes/`: rodar `ddd-architect` primeiro pra gerar `docs/ddd/`, depois retomar `ui-architect` pra exercitar o cenário de "revisão por enriquecimento" (aprendizado 4) e seguir pra fase 2.
