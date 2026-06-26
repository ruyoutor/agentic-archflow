# Fase 1 — Intake & triagem (chapéu PM/PO)

Transforma uma demanda crua numa fatia bem definida e fechável. **Não** abre código nem decide solução técnica — isso é fase 2+. O produto é um `CHG-NNN` com problema, valor, critério de aceite e a fatia mínima.

## Passos

1. **Capturar a demanda crua.** Na voz do usuário, sem traduzir pra solução. Registre no inbox do `backlog.md` se ainda não estiver.

2. **Classificar o tipo** — explicitamente, confirmando com o usuário (não inferir do enunciado):
   - **feature nova** — capacidade que não existe.
   - **mudança de comportamento** — a feature existe, a regra muda.
   - **remoção** — tirar algo.
   - **bug** — comportamento errado vs. o esperado.

   Caso de fronteira comum: "quero ver X de outro jeito / com um filtro" costuma ser **feature nova** (capacidade de visão nova), não mudança — confirme.

3. **Articular o job-to-be-done.** "Como {ator}, quero {capacidade}, para {resultado}." O *para* é o que importa — é o valor. Se não está claro por que dói hoje, pergunte antes de seguir.

4. **Definir o valor / hipótese.** Que decisão fica melhor, que dor some. Para projeto pessoal, basta uma frase honesta — não force métrica.

5. **Escrever o critério de aceite testável.** Em forma "dado/quando/então", observável. **Sempre** incluir um critério de *preservação*: o que NÃO pode mudar (o comportamento existente que continua funcionando). É o que protege contra regressão.

6. **Fatiar.** A fatia é o menor vertical que entrega valor de verdade. Se não cabe numa volta, liste as fatias sequenciais no backlog e marque qual é esta. Evite fatia que só entrega "metade da camada" (ex.: só backend sem nada observável).

7. **Caçar ambiguidades de domínio/produto.** Toda fatia carrega perguntas que parecem detalhe mas mudam o cálculo/comportamento (ex.: "ao filtrar só X, o que entra no agregado Y?"). Liste-as na seção 5 do spec. **Não decida agora** — elas são resolvidas na fase 2, com o modelo vivo à mão e o usuário no loop. Listá-las é metade do trabalho da triagem.

8. **Escrever o `CHG-NNN`** a partir do template, criar/atualizar o `backlog.md`, apresentar, **pausar**.

## Saída

- `docs/changes/CHG-NNN-<slug>.md` com seções 1–5 preenchidas (6–8 ficam pras fases seguintes).
- `docs/changes/backlog.md` atualizado.

## Regras

- Não proponha solução técnica nem nomes de endpoint/campo aqui. Isso é fase 2.
- Critério de aceite sem o item de preservação é incompleto — toda evolução protege o que já existe.
- Ambiguidade não resolvida é registrada, não chutada.
- Calibre a cerimônia pelo tamanho do projeto. O spec leve basta; campos de PM/PO pleno ficam vazios.
