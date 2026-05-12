# Fase 4.4 — Composition root + main + smoke test

**Entrada:** tudo gerado nas fases anteriores
**Saída:** entrypoint conectado, configuração, smoke test que prova que o sistema sobe

## Método

### 1. Composition root

Conectar dependências num único lugar (conforme convenção do arquivo de stack). Sem service locator, sem singletons globais. Passar dependências explicitamente nos construtores.

Estrutura típica:
- Construir conexões (DB, broker)
- Construir repositórios (com a conexão)
- Construir use cases (com os repositórios e portas)
- Construir handlers HTTP (com os use cases)
- Devolver tudo para o `main` montar o servidor

### 2. Main

O entrypoint:

- Carrega config (env vars, arquivo, conforme stack)
- Constrói o composition root
- Inicia servidor HTTP / consumers / workers conforme o ADR
- Trata graceful shutdown (sinais SIGTERM/SIGINT, drain das goroutines)

Mantenha `main` curto. A lógica de wiring vive no composition root, não no main.

### 3. Smoke test

Um único teste que:

- Sobe o sistema (com testcontainers ou fakes em memória, conforme stack)
- Faz **uma operação happy-path end-to-end por bounded context**
- Tear down

Não é E2E exaustivo — é checagem "o sistema realmente sobe e faz alguma coisa". Pegar mismatches grosseiros de wiring que escapam dos testes unitários.

### 4. Checkpoint final

Rodar a suíte inteira. Apresentar:

- Número de testes, pass/fail por camada (domínio, aplicação, adapters, smoke)
- Cobertura por pacote (se a stack definiu meta)
- Árvore de arquivos gerados
- Estado final do `docs/ddd/04-build-log.md`

Diga ao usuário que o sistema está pronto para rodar, e como rodar (comando do arquivo de stack).

## Anti-padrões

- **Wiring no main.** Main vira um arquivo de 200 linhas conectando tudo. Mova para o composition root.
- **Singleton de DB ou broker.** Passe explicitamente. Singletons impedem testabilidade.
- **Smoke test exaustivo.** Se você está testando todos os caminhos de erro no smoke, está duplicando o trabalho das outras fases. Smoke é "boots e respira".
- **Esquecer graceful shutdown.** Em sistemas com consumers/workers, shutdown sujo perde mensagens em flight.
