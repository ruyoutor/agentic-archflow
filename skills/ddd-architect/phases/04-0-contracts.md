# Fase 4.0 — Contratos

**Entrada:** artefato tático + arquivo de stack
**Saída:** por bounded context, arquivo de contratos (interfaces, comandos, eventos, erros)

## Propósito

Gerar o **vocabulário compartilhado** que os subagents de teste e de código vão ler na 4.1 e 4.2. Isso elimina divergência em nomes, assinaturas e tipos.

Sem essa fase, os dois tracks paralelos podem inventar nomes diferentes para o mesmo conceito, e os testes deixam de compilar contra o código. Com ela, ambos partem da mesma fonte da verdade.

## Método

Para cada bounded context no artefato tático:

### 1. Interfaces de repositório

Para cada raiz de agregado, escreva a interface do repositório com base na seção de repositórios do tático. Apenas métodos, sem implementações.

### 2. Tipos de comando e evento

- **Comandos**: structs com os campos necessários para invocar um use case
- **Eventos**: structs representando eventos de domínio emitidos

Nomes seguem a linguagem ubíqua do artefato estratégico.

### 3. Tipos de erro

- **Sentinel errors** para falhas conhecidas e estáveis (`ErrPedidoJaPago`, `ErrEstoqueInsuficiente`)
- **Tipos de erro** quando precisa carregar contexto (struct com campos)

Ver `templates/stacks/<stack>.md` para o estilo idiomático.

### 4. Interfaces de serviço

Para domain services e portas de use case declarados no tático.

## Localização da saída

Stack-dependente — ver `templates/stacks/<stack>.md`. Em Go: `internal/<context>/contracts.go` (pode dividir em arquivos se ficar grande, mas mantenha **um único ponto de entrada** por contexto para os subagents lerem).

## Verificação por build

Após gerar, rode o comando de build da stack. Os contratos devem compilar sozinhos (sem implementações ainda, mas as interfaces e tipos têm que ser válidos).

## Checkpoint

Mostre os arquivos de contrato ao usuário e pause. Pergunte se algum nome, assinatura ou tipo deve mudar antes da fase paralela. **Mudanças aqui são baratas; mudanças depois da 4.1 cascateiam** (se um nome muda depois, testes e código precisam re-alinhar e o ganho do paralelismo se perde).

Pergunta padrão: "Os contratos estão em `<paths>`. Eles definem a linguagem que testes e código vão compartilhar. Algo deve mudar antes da fase paralela?"
