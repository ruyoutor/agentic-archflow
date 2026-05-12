# Fase 4.1 — Domínio (testes ∥ código em paralelo)

**Entrada:** artefato tático + contratos da 4.0 + arquivo de stack
**Saída:** implementação de domínio + testes de domínio, ambos compilando contra os mesmos contratos

## Método

Para cada bounded context (um de cada vez, para manter o contexto dos subagents pequeno):

### 1. Despachar dois subagents em paralelo

Em uma única mensagem, dispare dois agents via `Agent` tool com `subagent_type` específico (NÃO usar `general-purpose` — os agents dedicados têm tool restrictions e role definitions já carregadas):

- **Agent A — `ddd-domain-test-generator`** (definição em `~/.claude/agents/ddd-domain-test-generator.md`)
- **Agent B — `ddd-domain-code-generator`** (definição em `~/.claude/agents/ddd-domain-code-generator.md`)

O role / mandato / convenções / restrições / formato de retorno vivem **nos arquivos dos agents**. No `prompt` da invocação, o orquestrador passa **apenas** o que muda a cada execução:

1. **Caminhos absolutos dos artefatos a ler** (tático, stack, contratos, stubs)
2. **Caminhos absolutos dos arquivos a escrever** (testes para o A, stubs a sobrescrever para o B)
3. **Nome do bounded context** sendo processado
4. **Ambiguidades já resolvidas** pelas fases anteriores (e.g., "para `Multa.Quitar` com data anterior à geração, use `ErrDataQuitacaoInvalida`")
5. **Dados de convergência** quando o tático menciona um algoritmo conhecido mas não detalha os exemplos (e.g., "CPFs válidos para teste: 52998224725, 39053344705, 11144477735"). **Verifique esses exemplos antes de passar** — eles são o ponto de convergência entre os dois agents e um erro aqui produz divergência inevitável (lição registrada na seção 3).
6. **Algoritmos específicos** quando aplicável (e.g., "algoritmo CPF: mod-11 brasileiro com rejeição de dígitos repetidos")

Despache os dois agents na **mesma mensagem** (Agent calls em paralelo). Eles não se veem; convergem pelos contratos compartilhados.

### 2. Build e test

Após os dois subagents retornarem, rode os comandos de build e test da stack.

### 3. Resolver divergências

Se os testes não compilarem ou falharem ao rodar:

1. Liste cada divergência: arquivo, linha, o que o teste esperava, o que o código provê.
2. **Identifique qual lado bate com a spec do tático.** Ordem de arbitragem:
   - **Spec explícita no tático** (e.g., "CPF válido pelo algoritmo mod-11", "transição one-way") → use o tático como árbitro. Ajuste o lado que diverge da spec.
   - **Tático silencioso ou ambíguo** → recomende ajustar o código para casar com o teste (heurística: teste é mais expressivo da intenção). Marque a ambiguidade pra fortalecer o tático.
   - **Ambos divergem da spec** → ambos precisam mudar.
3. Mostre o diff e a recomendação justificada ao usuário.
4. **Aguarde aprovação.**
5. Aplique a correção. Re-rode build/test.

> **Atenção — o default "código segue teste" não é universal.** Ele pressupõe que o teste é a melhor representação da spec. Esse pressuposto **quebra** quando:
>
> - O orquestrador da skill injetou dados de convergência errados no prompt do test agent (exemplos numéricos errados, listas factuais incorretas, etc.). Sintoma: o test agent não tinha como saber que estava errado.
> - Os dois subagents interpretaram uma ambiguidade do tático de formas diferentes (test escolheu A, code escolheu B). Nesse caso, NENHUM dos dois é necessariamente "a spec" — o tático precisa ser refinado primeiro.
>
> Sempre retorne ao tático como árbitro definitivo antes de aplicar o default.

Repita até verde.

### 4. Atualizar build log

Adicione ao `docs/ddd/04-build-log.md`: resultados dos subagents, divergências encontradas, resoluções.

### 5. Checkpoint

Apresente a saída verde dos testes e a lista de arquivos. Pause antes da fase 4.2.

## Anti-padrões

- **Não deixe os subagents se verem.** O ponto é interpretações independentes encontrando-se no contrato.
- **Não re-rode subagents após divergência.** Conserte cirurgicamente. Re-rodar queima tokens e pode introduzir divergências diferentes.
- **Não gere testes de integração aqui.** Isso é domínio puro. Nada de DB, nada de HTTP.
- **Não invente invariantes "óbvios" não escritos no tático.** Se faltar, sinalize como ambiguidade — não preencha.
