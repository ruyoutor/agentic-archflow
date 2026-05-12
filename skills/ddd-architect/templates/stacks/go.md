# Stack: Go

> Convenções idiomáticas Go para a fase de implementação.

## Layout do projeto

```
<project>/
├── cmd/
│   └── <app>/
│       └── main.go               # entrypoint mínimo
├── internal/
│   ├── <context-a>/
│   │   ├── domain/               # agregados, VOs, invariantes — sem deps externas
│   │   ├── app/                  # use cases / application services
│   │   ├── adapters/
│   │   │   ├── postgres/         # repositórios e migrations
│   │   │   ├── http/             # handlers
│   │   │   ├── messaging/        # publishers / consumers
│   │   │   └── outbox/           # se ADR escolheu outbox
│   │   └── contracts.go          # interfaces, comandos, eventos, erros (gerado em 4.0)
│   ├── <context-b>/
│   │   └── ...
│   └── platform/                 # composition root, config, logger, observability
├── pkg/                          # APENAS se algo é genuinamente compartilhado entre contextos
├── go.mod
└── go.sum
```

## Convenções

### Pacotes
- Nomes curtos, em letra minúscula, sem underscores.
- Um pacote = um conceito. `domain`, não `domainmodels`.
- `internal/` para tudo que não é API pública. Praticamente tudo.

### Interfaces
- **Pequenas** (1–3 métodos). Compor é melhor que herdar.
- Declaradas no consumidor, não no produtor (a app declara `OrderRepository`, o adapter postgres satisfaz).
- Em `contracts.go`, agrupar interfaces por contexto, não por camada.

### Erros
- **Sentinel errors** para falhas conhecidas e estáveis: `var ErrPedidoJaPago = errors.New("pedido: já pago")`
- **Tipos de erro** quando precisa carregar contexto: `type ErrEstoqueInsuficiente struct { Item string; Pedidos, Disponivel int }`
- `errors.Is` / `errors.As` para checagem. Não comparar strings.
- Wrap com `fmt.Errorf("...: %w", err)` ao propagar entre camadas.

### Construtores
- `func NewX(...) (*X, error)` quando há validação. `func NewX(...) *X` quando não.
- Retornar `*Type` quando o tipo tem mutabilidade ou identidade.

### Domínio
- Sem dependências em `database/sql`, HTTP, ou qualquer infra.
- Agregados expõem operações com nomes do domínio: `pedido.Cancelar()`, não `pedido.SetStatusCancelado()`.
- Invariantes verificados na construção e em cada operação que possa violá-los.

### Aplicação
- Use cases recebem dependências via construtor (repositórios, portas).
- Cada use case = um struct com método `Execute` (ou nome de domínio: `PlaceOrder`).
- Sem regra de negócio. Apenas orquestração.

## Testes

- **Framework:** stdlib `testing` + `testify/assert` para asserções.
- **Tabela de casos:** padrão preferido.
  ```go
  tt := []struct {
      name string
      // ...
  }{...}
  for _, tc := range tt {
      t.Run(tc.name, func(t *testing.T) { ... })
  }
  ```
- **Naming:** `TestPedido_Cancelar_QuandoJaPago_RetornaErro`. Verboso é OK.
- **Domínio:** sem mocks. Testa contra os tipos reais.
- **Aplicação:** fakes hand-rolled dos repositórios (preferir fakes a mocks gerados — mais ágeis pra evoluir).
- **Integração:** tag `//go:build integration`, roda com `go test -tags=integration`. Use `testcontainers-go` para DB e brokers.

## Build / lint commands

- `go build ./...` — build tudo
- `go vet ./...` — análise estática
- `go test ./...` — testes unitários
- `go test -tags=integration ./...` — inclui integração
- `golangci-lint run` — se o projeto tiver config

A skill deve rodar `go build ./...` e `go vet ./...` após cada sub-fase de geração. `go test ./...` após 4.1, 4.2 e 4.4.

## Outbox em Go

Se a ADR escolheu outbox:

- Tabela `outbox_events`:
  ```sql
  id UUID PRIMARY KEY,
  aggregate_id TEXT NOT NULL,
  event_type TEXT NOT NULL,
  payload JSONB NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  sent_at TIMESTAMPTZ
  ```
- Writer: chamado dentro da mesma `*sql.Tx` que persiste o agregado. Falha aqui aborta a transação.
- Relayer: goroutine periódica (`time.Ticker`) que consulta `WHERE sent_at IS NULL ORDER BY created_at LIMIT N`, publica no broker, marca como enviado. Idempotência no consumer (use `id` do evento como dedup key).

## CQRS em Go (se aplicável)

- Comandos passam por use cases que escrevem no modelo de domínio.
- Queries vão para projeções otimizadas (views, tabelas desnormalizadas), **bypassando os agregados**.
- Não force CQRS em todo contexto — só onde a ADR justificou.

## Observabilidade (mínimo viável)

- Logger estruturado (slog da stdlib é suficiente para v1).
- Trace ID no contexto, propagado entre camadas via `context.Context`.
- Métricas: contadores em pontos chave (use case executado, evento publicado, erro de domínio). `expvar` ou Prometheus client conforme o ambiente.
