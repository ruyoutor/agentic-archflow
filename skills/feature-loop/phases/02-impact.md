# Fase 2 — Análise de impacto (chapéu arquiteto) + deliberação

Mapeia a fatia contra o **modelo vivo** (código + `docs/ddd` + `docs/ui`) e resolve, com o usuário,
as ambiguidades levantadas na fase 1. Produz as seções 6 (impacto) e 7 (plano de build) do CHG, e o
log de decisões.

## Princípio que aparece quase sempre

**Ler o modelo dissolve ambiguidade.** Dúvidas que pareciam difíceis no abstrato muitas vezes já
têm resposta no código — porque o domínio foi bem modelado. Antes de deliberar uma ambiguidade,
**leia o código relevante**: a resposta pode já estar lá, e a "decisão" vira só expor o que existe.
Não delibere no vácuo o que o modelo já decide.

## Passos

1. **Ler o modelo vivo na vizinhança da fatia.** O agregado/serviço/componente que a fatia toca, e
   o contrato (OpenAPI). Vá ao código, não só aos docs — os docs podem estar atrás do código (e isso
   mesmo já é um achado: registre como bug/divergência).

2. **Resolver as ambiguidades (seção 5) — deliberação como sparring partner.** Para cada dúvida:
   - Primeiro cheque se o modelo já responde (princípio acima).
   - Se é decisão genuína, **traga opções, exponha tradeoffs e recomende** — não devolva a pergunta
     crua pro usuário. Você é o par de PM/PO dele, não um estenógrafo.
   - O debate é chat livre. O resíduo durável vira uma entrada no **log de decisões** do CHG
     (pergunta → opções pesadas → decisão → porquê). Um mini-ADR por dúvida.

3. **Mapear o impacto (seção 6).** Por camada: domínio (agregados/VOs/invariantes — muda invariante?),
   arquitetura (toca/cria ADR?), contrato (aditivo ou breaking?), UI (telas/design system). Marque se
   resolve ou cria um DEM. Diga explicitamente o que **não** muda (proteção do critério de preservação).

4. **Definir o plano de build (seção 7).** A quais skills builder delega (em modo incremental) e em que
   ordem (em geral backend → contrato → frontend). Antecipe o **footprint** — o que a fatia vai
   adicionar/tocar — pra a fase 4 só confirmar.

5. **Apresentar, pausar.** O usuário confirma as decisões antes da construção.

## Saída

- `CHG-NNN` com seções 6 e 7 preenchidas e o log de decisões.
- Ambiguidades da seção 5 todas endereçadas (resolvidas pelo modelo ou decididas).

## Regras

- Não delibere o que o modelo já decide — leia primeiro.
- Recomende sempre; o usuário decide. Registre o porquê, não só o quê.
- Ambiguidade que sobrar sem decisão **bloqueia** a fase 3 — não construa sobre dúvida.
- Se a fatia revelar que precisa de mudança grande no modelo (novo agregado, ADR pesado), isso pode
  exceder uma volta — quebre em fatias no backlog em vez de inchar esta.
