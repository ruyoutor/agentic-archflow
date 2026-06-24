# Fase 4.3 — Adapters

**Entrada:** todos os artefatos anteriores + camadas de domínio e aplicação completas
**Saída:** adapters de infraestrutura (DB, HTTP, mensageria, outbox se aplicável) + testes de integração

## Por que sequencial e não paralelo

Adapters dependem de infra real (schema Postgres, especificidades do framework HTTP, cliente do broker). Gerar testes em paralelo a partir da spec forçaria o agente de teste a inventar detalhes de infra. Melhor: código primeiro, depois testes de integração focados contra a infra escolhida.

## Protocolo de decisão colaborativa (OBRIGATÓRIO)

Adapters são heterogêneos (DB, HTTP server, HTML parser, cliente externo, queue, etc.) — não há padrão único. **Decisões técnicas são tomadas por adapter, em conversa com o usuário, ANTES de implementar.** Não inventar defaults silenciosos.

Antes de implementar cada adapter, alinhe explicitamente os 4 pontos abaixo:

### 1. Porta no `contracts.go`

Assinatura da interface (métodos, parâmetros, retornos, sentinel errors).

- Se a porta ainda não existe ou precisa ser ajustada, **edite `contracts.go` ANTES** de implementar o adapter.
- Mostre o diff da edição ao usuário antes de salvar.
- Adapter sempre **implementa** uma porta do domínio — nunca o contrário (adapter ditando contrato).

### 2. Biblioteca / cliente externo

Qual dependência externa será usada. Liste alternativas com tradeoffs explícitos (e.g., `mattn/go-sqlite3` cgo vs `modernc.org/sqlite` puro Go; `chi` vs `gorilla/mux` vs `net/http` stdlib).

Espere decisão do usuário. Em caso de empate técnico, recomende uma com justificativa, mas não force.

### 3. Estratégia de teste

Pra cada adapter, declare ANTES de codar:

- **Integration test** (real I/O, tag `//go:build integration`) — pra adapters de DB, fila, etc.
- **Unit com `httptest.Server` ou fakes locais** — pra clientes HTTP, parsers.
- **Mista** — base unit + smoke integration.

A escolha vira tag de build apropriada e estrutura de fixtures.

### 4. Composição (DI) no composition root

Confirme com o usuário **onde** e **como** o adapter será injetado no composition root (fase 4.4). Isso evita adapter que depende de algo que o composition root não tem como fornecer.

Decisão típica: adapter recebe `*sql.DB` / `*http.Client` / config struct via construtor; composition root provê.

---

Nunca avance pra implementação sem esses 4 pontos alinhados. **Em chat novo, isso significa: PERGUNTE; não invente padrão.**

## Método

Para cada tipo de adapter (conforme o ADR de arquitetura):

### 1. Implementações de repositório

Implementar as interfaces de repositório dos contratos usando a persistência escolhida (ver ADR). Seguir convenções do arquivo de stack para migrations / schema.

### 2. Adapters HTTP / API

Se o ADR especificou API síncrona: handlers que traduzem requisições HTTP em comandos e chamam use cases. **Validação vive aqui**, não no domínio (validação de formato, parsing, autenticação). Validação de regra de negócio fica no domínio.

### 3. Adapters de mensageria

Se eventos assíncronos: publishers e consumers conforme o broker escolhido no ADR.

### 4. Outbox

Se o ADR escolheu outbox:

- Tabela `outbox_events` (id, aggregate_id, event_type, payload, created_at, sent_at nullable)
- Writer chamado dentro da mesma transação que persiste o agregado
- Relayer: processo periódico que pega `sent_at IS NULL`, publica no broker, marca como enviado
- Consumer deve ser idempotente (use o id do evento como dedup key)

### 5. Testes de integração

Após adapters compilarem, gerar testes de integração contra infra real (ou via testcontainers). Uma passada por tipo de adapter. São mais lentos; isolar por convenção da stack (`go test -tags=integration` ou similar).

Foque em:
- Round-trip do agregado (save + load mantém invariantes)
- Queries de domínio retornam o esperado
- Handler HTTP traduz request corretamente e propaga erros de domínio
- Outbox: evento gravado na mesma tx, relayer publica, consumer não duplica

### 6. Checkpoint

Rodar a suíte completa (unit + integration). Apresentar resultados. Pausar.

## Anti-padrões

- **Regra de negócio em adapter.** Adapter traduz; não decide. Se vê um `if` checando regra num handler HTTP, mova para o domínio.
- **Um adapter fazendo dois trabalhos.** Repositório que também publica eventos é errado. Publicação acontece na app layer ou via outbox.
- **Testes de integração testando regra.** Se o teste de integração quer cobrir um invariante, esse invariante já deveria estar coberto na 4.1. Integração testa o pipe, não a regra.
- **Acoplar adapter ao domínio.** O domínio define a porta; o adapter implementa. Nunca o contrário.
