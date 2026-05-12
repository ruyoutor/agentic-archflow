# Fase 4.1 — Domínio (testes ∥ código em paralelo)

**Entrada:** artefato tático + contratos da 4.0 + arquivo de stack
**Saída:** implementação de domínio + testes de domínio, ambos compilando contra os mesmos contratos

## Método

Para cada bounded context (um de cada vez, para manter o contexto dos subagents pequeno):

### 1. Despachar dois subagents em paralelo

Em uma única mensagem, dispare dois subagents `general-purpose`:

#### Subagent A — Gerador de testes de domínio

**Prompt deve conter:**

- **Ler:** caminho absoluto do artefato tático (seção do contexto), caminho do arquivo de contratos, caminho do arquivo de stack
- **Escrever:** arquivos de teste de domínio (e.g., `internal/<context>/domain/*_test.go`)
- **Mandato:** cobrir cada invariante listado na seção tática deste contexto. **No mínimo um teste por invariante.** Usar **apenas** tipos e assinaturas do arquivo de contratos. Não inventar comportamento além do documentado.
- **Restrições:** não testar nada de infra (sem DB, sem HTTP, sem mock de fora dos contratos). Domínio puro.
- **Retorno esperado:** lista de arquivos escritos + lista de invariantes que o tático deixou ambíguos (não tente adivinhar — sinalize).

#### Subagent B — Gerador de código de domínio

**Prompt deve conter:**

- **Ler:** caminho absoluto do artefato tático (seção do contexto), caminho do arquivo de contratos, caminho do arquivo de stack
- **Escrever:** arquivos de implementação de domínio (e.g., `internal/<context>/domain/*.go`)
- **Mandato:** agregados, entidades, VOs, domain services. Implementar cada invariante listado. Usar **apenas** tipos e assinaturas dos contratos.
- **Restrições:** domínio não pode depender de infraestrutura. Sem `database/sql`, sem HTTP, sem broker. Sem features além da spec.
- **Retorno esperado:** lista de arquivos escritos.

Ambos subagents devem produzir **apenas o que está na spec**. Sem features bônus.

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
