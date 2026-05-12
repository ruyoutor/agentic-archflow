# Fase 4.2 — Aplicação (testes ∥ código em paralelo)

**Entrada:** artefato tático + contratos (atualizados, se a 4.1 mudou algum) + camada de domínio da 4.1 + arquivo de stack
**Saída:** application services / use cases + seus testes

## Método

Mesmo padrão paralelo da 4.1, mas na camada de aplicação.

Para cada bounded context:

### 1. Despachar dois subagents em paralelo

#### Subagent A — Gerador de testes de aplicação

**Prompt deve conter:**

- **Ler:** artefato tático, contratos, arquivo de stack
- **Escrever:** testes de use case (`internal/<context>/app/*_test.go`)
- **Mandato:** cada use case tem um teste de happy-path mais testes para os modos de falha documentados (erros e invariantes que o use case pode disparar). Mockar repositórios na fronteira do contrato. **Não testar a camada de domínio aqui** (isso é 4.1).
- **Retorno esperado:** lista de arquivos + ambiguidades para sinalizar.

#### Subagent B — Gerador de código de aplicação

**Prompt deve conter:**

- **Ler:** artefato tático, contratos, arquivo de stack, tipos do domínio da 4.1
- **Escrever:** use cases / application services (`internal/<context>/app/*.go`)
- **Mandato:** orquestrar domínio + repositórios. **Sem regra de negócio aqui** (regras vivem no domínio). Traduzir comandos para operações de domínio. Tratar fronteiras de transação conforme o ADR de arquitetura (e.g., chamar outbox se o ADR escolheu outbox).
- **Retorno esperado:** lista de arquivos escritos.

### 2. Build, test, resolver divergências

Mesmo protocolo da 4.1.

### 3. Checkpoint

Pause antes da fase 4.3.

## Nota sobre outbox / async

Se o ADR escolheu outbox, o application service deve chamar uma **porta de outbox** (interface dos contratos). A implementação real do outbox é um adapter na 4.3. Código de aplicação só depende da interface.

Mesma regra para qualquer integração externa: aplicação fala com **portas**, não com adapters concretos.

## Anti-padrões

- **Regra de negócio na aplicação.** Se você está validando "pedido pode ser cancelado" no use case, mova para o agregado.
- **Use case fazendo I/O direto.** Use case orquestra; portas fazem I/O. Se vê `db.Query` num use case, errou.
- **Mock do domínio em testes de aplicação.** O domínio é puro e barato — use os tipos reais. Mock só os repositórios e portas externas.
