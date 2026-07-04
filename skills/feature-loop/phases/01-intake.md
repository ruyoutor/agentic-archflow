# Fase 1 — Refinamento de produto (chapéu PO)

Transforma uma demanda crua numa **história pronta pra desenvolvimento**: problema, valor, critérios
de aceite, fatia mínima e — quando a fatia tem superfície de UI — o mockup de UX. **Não** abre código
de produto nem decide solução técnica — isso é trabalho da builder, na análise de impacto. O produto
desta fase é um `CHG-NNN` que uma builder consegue pegar e responder sozinha "o que precisa mudar?".

## Passos

1. **Capturar a demanda crua.** Na voz do usuário, sem traduzir pra solução. Registre no inbox do
   `backlog.md` se ainda não estiver.

2. **Classificar o tipo** — explicitamente, confirmando com o usuário (não inferir do enunciado):
   - **feature nova** — capacidade que não existe.
   - **mudança de comportamento** — a feature existe, a regra muda.
   - **remoção** — tirar algo (segue `phases/removal.md`).
   - **bug** — comportamento errado vs. o esperado.

   Caso de fronteira comum: "quero ver X de outro jeito / com um filtro" costuma ser **feature nova**
   (capacidade de visão nova), não mudança — confirme.

3. **Articular o job-to-be-done.** "Como {ator}, quero {capacidade}, para {resultado}." O *para* é o
   que importa — é o valor. Se não está claro por que dói hoje, pergunte antes de seguir.

4. **Escrever o contexto de produto.** O que o usuário do produto vive hoje, o que muda pra ele com
   a mudança, e o objetivo em uma frase. É o que dá à builder o *porquê* sem ela precisar adivinhar.
   Product-only: nenhum nome de endpoint, campo, agregado ou componente aqui.

5. **Definir o valor / hipótese.** Que decisão fica melhor, que dor some. Para projeto pessoal, basta
   uma frase honesta — não force métrica.

6. **Escrever o critério de aceite testável.** Em forma "dado/quando/então", observável. **Sempre**
   incluir um critério de *preservação*: o que NÃO pode mudar (o comportamento existente que continua
   funcionando). É com esses critérios que o `feature-loop-acceptance-verifier` vai trabalhar na
   aterrissagem — critério vago hoje é aceite inverificável depois.

7. **Fatiar.** A fatia é o menor vertical que entrega valor de verdade. Se não cabe numa volta, a
   demanda é um **épico**: liste as histórias sequenciais no backlog e marque qual é esta. Evite fatia
   que só entrega "metade da camada" (ex.: só backend sem nada observável).

8. **Separar as ambiguidades em dois montes** (seção 5 do spec):
   - **De produto** ("é esse o corte certo?", "o que o usuário espera ver quando X?") — deliberadas
     **aqui**, como sparring partner: traga opções, exponha trade-offs, recomende. O resíduo vira
     entrada no log de decisões do CHG (pergunta → opções pesadas → decisão → porquê).
   - **Técnicas/de domínio** ("o que entra no cálculo quando filtro X?", "isso muda invariante?") —
     **não decida e não chute**: registre-as explicitamente **para a builder responder** na análise
     de impacto, com o código à mão. Listá-las bem é metade do trabalho da triagem.

9. **Mockup de UX (quando a fatia tem superfície de UI).** Despache o agent **`ux-mockup-designer`**
   passando: caminho do CHG, `docs/ui/` (design system, specs, telas existentes) e o contrato OpenAPI.
   Ele devolve um mockup product-side em `docs/changes/mockups/CHG-NNN-<slug>.md` — wireframe, estados,
   copy, proveniência de dados e superfície testável sugerida — **rotulado construível/aspiracional**.
   Apense a referência na seção de mockup do CHG. Revise com o usuário: o mockup é a intenção de
   interface da história; a `ui-architect` o promove a spec de componente na análise de impacto.
   Fatia sem UI pula este passo sem cerimônia.

10. **Escrever o `CHG-NNN`** a partir do template, criar/atualizar o `backlog.md` (status
    `em refinamento`), apresentar, **pausar**.

## Saída

- `docs/changes/CHG-NNN-<slug>.md` com seções 1–5 preenchidas (+ mockup, quando houver UI).
- `docs/changes/backlog.md` atualizado.

## Regras

- Não proponha solução técnica nem nomes de endpoint/campo/agregado. O *como* é da builder.
- Critério de aceite sem o item de preservação é incompleto — toda evolução protege o que já existe.
- Ambiguidade de produto se delibera aqui; ambiguidade técnica se registra pra builder. Nenhuma se chuta.
- O mockup é intenção de produto, não spec de implementação — quem o promove a spec é a `ui-architect`.
- Calibre a cerimônia pelo tamanho do projeto. O spec leve basta; campos de PM/PO pleno ficam vazios.
