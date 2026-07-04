# Protocolo de remoção (tipo: remoção)

Remove uma feature já entregue do produto, de forma rastreável e reversível-quando-possível.
Disparado por: *"Vamos remover a feature CHG-XXX"* (ou triagem que classifica uma demanda como remoção).

É uma variante do ciclo, nas mesmas raias — não um caminho à parte. O PO captura o **porquê** e marca
pronta; a **builder** (modo evolução) faz a análise de impacto **reversa** e executa; o PO aterrissa
e registra. A diferença é que a análise olha pra trás (o que isto sustenta?) em vez de pra frente.

## Princípios

**A reversibilidade é conquistada na construção, não na remoção.** Se o CHG-XXX foi entregue com
um footprint preciso (seção 9), remover é ler esse footprint e agir. Se não foi, a remoção começa
por reconstruir o footprint via git (commits do ID) — mais cara, mas possível.

**Remoção é uma mudança pra frente, não reescrita de história.** `git revert` é um rascunho útil
no cenário limpo; nunca reescreva a history da main. O resultado é sempre um commit novo.

## Raia 1 — Refinamento (feature-loop, PO)

1. **Confirmar o alvo e capturar o motivo.** Qual CHG. **O "por que remover" é o novo job-to-be-done**
   e entra no registro (seção 10) — é o aprendizado de produto mais valioso do ciclo. Não pule.
2. **Critério de aceite da remoção:** o que deve continuar funcionando depois que a feature sair
   (preservação), e o que caracteriza "removida de verdade" (nenhum resíduo observável).
3. Status → `pronta para desenvolvimento`; handoff manual como em `phases/02-ready.md`, apontando a
   builder dona do footprint (domínio → `ddd-architect`; UI → `ui-architect`; full-stack → ambas).

## Raia 3 — Análise reversa + execução (builder, modo evolução)

A builder, na fase E1 dela, faz a análise **reversa** em vez da normal:

1. **Ler o footprint (seção 9) do CHG-XXX.** Sem footprint, reconstruí-lo pelos commits do ID antes
   de agir.
2. **Varrer dependentes.** Para cada item introduzido (símbolo, campo de contrato, componente):
   algum CHG/código posterior passou a usar isto? Classifique:
   - **Limpo** — nada depende do que o CHG-XXX criou. Revert é viável.
   - **Entrelaçado** — há dependentes. Revert quebraria; a remoção é cirúrgica e desenhada.
3. **Ser honesto sobre o cenário no checkpoint.** "Tudo volta a como era antes" só vale no limpo.
   No entrelaçado, o "antes" pode nem existir mais — apresente o que será preciso desentrelaçar.
   **Checkpoint duro:** o usuário confirma se ainda quer seguir.
4. **Executar (após o "segue"), em branch fora da main** (regra git global; criar branch não é
   commit — o commit continua sendo da aterrissagem):
   - **Limpo:** `git revert --no-commit <commits do CHG-XXX>` como rascunho na working tree;
     revise o resultado. **Nunca `git revert` sem `--no-commit`** — quem comita é a feature-loop.
   - **Entrelaçado:** remoção cirúrgica — código + testes da feature, desentrelaçando os dependentes
     conforme a decisão do checkpoint.
   - **Contrato:** remover campo/endpoint é **breaking** pra quem consome. Cheque consumidores; se
     houver, deprecate antes de remover, ou registre a quebra.
   - **Sincronizar o modelo vivo ao contrário** (fase E3): reverta no tático/ADR/OpenAPI/design
     system o que a feature havia adicionado, com entrada no histórico de revisões. O modelo volta
     a descrever o produto real.
   - Suíte inteira verde. **Sem commit** — entrega `construída`.

## Raia 4 — Aterrissagem da remoção (feature-loop, PO)

- Build/test verde + app de pé (verificação ao vivo de que nada essencial quebrou).
- Auditar a sincronização reversa dos artefatos (como em `phases/03-land.md`).
- **Registrar no próprio CHG-XXX** (não criar CHG novo): status → `removida`; preencher a seção 10
  (data, motivo, cenário, dependentes, como foi feito, commit, verificação). A vida inteira da
  feature fica num arquivo: nasceu → entregue → removida, com o porquê de cada passo.
- No `backlog.md`: mover o CHG-XXX de "Entregues" para "Removidas".
- Commit: `remove: <descrição> (CHG-XXX)` ou `revert: …`. Confirme antes (regra global).

## Saída

- Feature fora da main, build/test verde, app verificado.
- `CHG-XXX` com seção 10 preenchida e status `removida`; modelo vivo e backlog sincronizados.
- Um commit (ou PR) que entrega a remoção.

## Regras

- O motivo da remoção é obrigatório — é o aprendizado de produto que justifica o ciclo.
- Nunca prometa "volta ao estado anterior" sem antes classificar limpo vs. entrelaçado.
- Remoção também sincroniza o modelo vivo — desfazer no código sem desfazer nos docs é dívida nova.
- A feature-loop não executa a remoção — nem revert "que é só um comando". Análise e execução são da builder.
