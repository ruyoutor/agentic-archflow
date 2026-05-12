# Fase 4 — Implementação

**Entrada:** os três artefatos em `docs/ddd/` + arquivo de stack escolhido
**Saída:** código rodável + `docs/ddd/04-build-log.md` (decisões e divergências)

## Sub-fases

A implementação roda em 5 sub-fases, cada uma com seu checkpoint.

| Sub-fase | Método | Modo |
|---|---|---|
| 4.0 Contratos | `phases/04-0-contracts.md` | sequencial |
| 4.1 Domínio (testes ∥ código) | `phases/04-1-domain.md` | paralelo via subagents |
| 4.2 Aplicação (testes ∥ código) | `phases/04-2-application.md` | paralelo via subagents |
| 4.3 Adapters | `phases/04-3-adapters.md` | sequencial, código → testes |
| 4.4 Composition | `phases/04-4-composition.md` | sequencial |

Após cada sub-fase: build, vet, test. Mostre o diff (ou os arquivos novos) ao usuário e pause.

## Tratamento de stack

Leia `templates/stacks/<stack>.md` uma vez no início da fase 4. Esse arquivo define:

- Convenções de layout
- Padrões idiomáticos (tamanho de interface, estrutura de pacote, error handling)
- Framework e convenções de teste
- Comandos de build/lint

`<stack>` default é `go`. Se o usuário especificou override, use-o. Se `templates/stacks/<stack>.md` não existir, **pare** e peça ao usuário as convenções; escreva o arquivo de stack antes de continuar.

## Protocolo de divergência (4.1, 4.2)

Quando os tracks paralelos de teste ∥ código produzem mismatches (teste referencia método/tipo que o código não gerou):

1. **Mostre o diff** ao usuário — liste cada mismatch (teste espera `X.Y(...)`, código provê `X.Z(...)`).
2. **Recomende** ajustar o código para casar com o teste (teste está mais próximo da spec).
3. **Aguarde a decisão do usuário** antes de aplicar qualquer mudança.

Nunca reconcilie em silêncio.

## Build log

Mantenha `docs/ddd/04-build-log.md` à medida que avança:

- Sub-fase iniciada / concluída
- Arquivos criados
- Divergências encontradas e resoluções
- Perguntas em aberto

Esse é o rastro de auditoria. Não pule.

## Regras gerais para subagents

Em 4.1 e 4.2, ao despachar subagents (use `Agent` com `subagent_type=general-purpose`), o prompt deve conter:

1. **O que ler** (caminhos absolutos dos artefatos e contratos)
2. **O que escrever** (caminhos absolutos dos arquivos de saída)
3. **Mandato explícito**: gerar **apenas** o que está na spec. Nada de features bônus.
4. **Restrições da camada**: domínio não pode importar infra; aplicação não pode ter regra de negócio; etc.
5. **Formato do retorno**: lista de arquivos escritos + lista de ambiguidades encontradas (não inventar; sinalizar).

Despache os dois subagents (testes e código) **na mesma mensagem**, com chamadas `Agent` paralelas. Eles não devem se ver. O ponto do paralelismo é exatamente forçar duas interpretações independentes a encontrar-se nos contratos compartilhados.
