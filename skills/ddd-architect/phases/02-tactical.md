# Fase 2 — DDD Tático

**Entrada:** `docs/ddd/01-strategic.md` (deve existir)
**Saída:** `docs/ddd/02-tactical.md` baseado em `templates/02-tactical.md.tmpl`

## Método

Faça modelagem tática **por bounded context**. Cada contexto vira uma seção no artefato.

Para cada bounded context:

### 1. Identificar agregados

Agregado é um cluster de objetos tratado como unidade de consistência. Heurísticas:

- Encontre os **invariantes**: regras que devem valer numa única transação. Os objetos envolvidos em um invariante pertencem ao mesmo agregado.
- Encontre a **raiz**: a entidade pela qual todo acesso passa. Os outros objetos só são alcançados via raiz.
- **Mantenha agregados pequenos.** Se você está com agregado de 5 entidades, procure invariantes que não sejam realmente transacionais — geralmente indicam outro agregado se comunicando por evento.

### 2. Distinguir entidades, value objects e domain services

Para cada peça do modelo:

- **Entity** — tem identidade que persiste no tempo (Pedido, Cliente)
- **Value Object** — definido pelos atributos, imutável, substituível (Dinheiro, Endereço, ItemDoPedido)
- **Domain Service** — operação que não pertence naturalmente a uma entidade/VO (Transferência que toca duas contas)

Default para **value object**. Promova para entidade só quando identidade importar.

### 3. Repositórios

Um repositório por raiz de agregado. Repositórios são **coleções** na linguagem do domínio — não expõem CRUD por padrão. Métodos refletem operações de domínio:

- `PedidoRepository.OfID(id)` — não `GetByID`
- `PedidoRepository.Save(pedido)` — cobre insert e update; o repo decide
- `PedidoRepository.CriadosNaData(d)` — query de domínio

### 4. Invariantes e regras

Para cada agregado, liste invariantes explicitamente. Eles viram asserts de teste depois. Formato:

- "Um pedido não pode ser cancelado depois de pago"
- "A soma dos itens deve igualar o total do pedido"
- "Um cliente bloqueado não pode realizar pedidos"

Se você não consegue listar invariantes, o modelo está sub-especificado. Volte ao usuário.

### 5. Eventos de domínio

Para cada contexto, liste os eventos emitidos (PedidoCriado, PagamentoConfirmado, EstoqueReservado). Eles serão a superfície de integração na fase 3.

### 6. Comandos

Comandos representam intenções que entram no sistema (CriarPedido, CancelarPedido). Liste o comando, o agregado que recebe e os campos.

### 7. Escrever o artefato

Use `templates/02-tactical.md.tmpl`. Salve em `docs/ddd/02-tactical.md`. Uma seção por bounded context. Apresente e pause.

## Anti-padrões

- **Domínio anêmico.** Entidades só com getters/setters e nenhum comportamento — sinal de que regras vazaram para serviços.
- **Agregado gigante.** "Cliente" com 30 entidades filhas. Procure eventos no lugar.
- **Repositório como DAO.** Se o repo tem 12 métodos de query nomeados como filtros SQL, você está construindo camada de dados, não domínio.
- **Inventar invariantes.** Se o usuário não disse, não escreva.
- **Promover tudo a entidade.** Promova só quando identidade importar de fato.
