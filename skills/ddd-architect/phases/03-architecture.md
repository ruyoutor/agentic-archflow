# Fase 3 — Arquitetura

**Entrada:** `docs/ddd/01-strategic.md`, `docs/ddd/02-tactical.md`
**Saída:** `docs/ddd/03-architecture.md` (formato ADR) baseado em `templates/03-architecture-adr.md.tmpl`

## Método

Decisões arquiteturais decorrem dos artefatos estratégico e tático. **Não proponha padrões que o mini-mundo não justifica.**

### 1. Avaliar complexidade

Pontue o sistema nestes eixos (low / medium / high):

- **Nº de bounded contexts** (1 = monolito, 5+ = distribuição séria)
- **Consistência inter-contextos** (eventual ok? forte necessária?)
- **Throughput / escala**
- **Tamanho e estrutura do time** (Conway's law)
- **Maturidade operacional** (o time roda sistemas async hoje?)
- **Pressão temporal** (entrega em 2 semanas ou 6 meses?)

Os scores vão para o ADR. Eles justificam as decisões.

### 2. Decidir topologia de deployment

Árvore de decisão:

- **Um único bounded context, baixa complexidade** → monolito modular, um deployable
- **Múltiplos contextos, baixa maturidade ops** → monolito modular com fronteiras de pacote internas (prepara extração futura, não extrai agora)
- **Múltiplos contextos, alta maturidade ops, alto throughput** → serviços por contexto

**Default monolito primeiro.** Distribuição custa. Pague quando justificado.

### 3. Decidir comunicação inter-contextos

Para cada edge do mapa de contextos:

- **API síncrona (REST/gRPC)** — quando o downstream precisa de resposta síncrona (e.g., checkout precisa verificar estoque agora)
- **Eventos assíncronos** — quando o downstream reage a mudanças mas não bloqueia (e.g., notificação ao criar pedido)
- **Ambos** — comum: chamada síncrona para necessidade imediata + eventos para os outros consumidores

Documente a escolha por edge.

### 4. Decidir estratégias de consistência

Para cada operação cross-aggregate ou cross-context:

- **Transação local** — se fica num único agregado, default
- **Outbox pattern** — quando emite eventos que precisam refletir uma mudança de estado atomicamente (quase sempre, em async)
- **Saga / process manager** — para workflows multi-passo de longa duração com compensação
- **2PC / transações distribuídas** — quase nunca. Justifique muito bem.

### 5. Decidir padrões de leitura

- **CRUD direto a partir do agregado** — default
- **CQRS com projeções** — quando o shape de leitura diverge muito do modelo de escrita, ou escala de leitura > escala de escrita
- **Event sourcing** — só quando auditoria / queries temporais / replay são requisitos de primeira classe. Pesa operacionalmente.

### 6. Planejar evolução

Escreva:

- Qual a versão **mais simples** que entrega valor (v1)?
- Quais sinais justificariam o **próximo passo** (extrair contexto, adicionar CQRS, adotar event sourcing)?
- O que está deliberadamente **fora de escopo** para v1?

Essa seção mantém o ADR honesto sobre o agora vs o depois.

### 7. Escrever o ADR

Use `templates/03-architecture-adr.md.tmpl`. Salve em `docs/ddd/03-architecture.md`. Cada decisão tem: contexto, opções consideradas, decisão, consequências. Apresente e pause.

## Anti-padrões

- **Microservices num mini-mundo pequeno.** Custo > benefício.
- **Outbox sem async.** Se você não emite eventos, não precisa de outbox.
- **CQRS por default.** Adiciona complexidade. Justifique com divergência concreta de read/write ou argumento de escala.
- **Event sourcing porque é cool.** Se auditoria não é requisito declarado, pule.
- **Decisão sem alternativa registrada.** "Decidimos REST" sem listar gRPC/eventos como considerados é uma decisão sem rastro. ADR registra alternativas.
