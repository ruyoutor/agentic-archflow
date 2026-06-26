# Protocolo de remoção (tipo: remoção)

Remove uma feature já entregue do produto, de forma rastreável e reversível-quando-possível.
Disparado por: *"Vamos remover a feature CHG-XXX"* (ou triagem que classifica uma demanda como remoção).

É uma variante do ciclo — não um caminho à parte. Tem intake (por que remover), análise de
impacto **reversa**, execução e aterrissagem. A diferença é que a análise olha pra trás (o que
isto sustenta?) em vez de pra frente.

## Princípio

**A reversibilidade é conquistada na construção, não na remoção.** Se o CHG-XXX foi entregue com
um footprint preciso (fase 4, seção 9), remover é ler esse footprint e agir. Se não foi, a remoção
começa por reconstruir o footprint via git (commits do ID) — mais cara, mas possível.

**Remoção é uma mudança pra frente, não reescrita de história.** `git revert` é um rascunho útil
no cenário limpo; nunca reescreva a history da main. O resultado é sempre um commit novo.

## Passos

1. **Confirmar o alvo e capturar o motivo.** Qual CHG. **O "por que remover" é o novo job-to-be-done**
   e entra no registro (seção 10) — é o aprendizado de produto mais valioso do ciclo. Não pule.

2. **Análise de impacto reversa.** Leia o footprint (seção 9) do CHG-XXX. Para cada item introduzido
   (símbolo, campo de contrato, componente), **varra dependentes**: algum CHG/código posterior passou
   a usar isto? Classifique:
   - **Limpo** — nada depende do que o CHG-XXX criou. Revert é viável.
   - **Entrelaçado** — há dependentes. Revert quebraria; a remoção é cirúrgica e desenhada.

3. **Ser honesto sobre o cenário com o usuário.** "Tudo volta a como era antes" só vale no limpo.
   No entrelaçado, o "antes" pode nem existir mais (o produto evoluiu) — apresente o que será preciso
   desentrelaçar e confirme se ele ainda quer seguir.

4. **Branch.** Toda remoção nasce em branch fora da main (regra git global).

5. **Executar:**
   - **Limpo:** `git revert <commits do CHG-XXX>` como ponto de partida; revise o resultado.
   - **Entrelaçado:** remova cirurgicamente — código + testes da feature, desentrelaçando os
     dependentes conforme a decisão do passo 3.
   - **Contrato:** se a feature adicionou campo/endpoint, removê-lo é **breaking** para quem consome.
     Cheque consumidores; se houver, deprecate antes de remover, ou registre a quebra.
   - Remova telas/controles de UI e o que mais o footprint listar.

6. **Aterrissagem da remoção:**
   - Build/test verde + app de pé (verificação ao vivo de que nada essencial quebrou).
   - **Sincronizar o modelo vivo ao contrário:** reverta no tático/ADR/OpenAPI/design system o que a
     feature havia adicionado. O modelo volta a descrever o produto real.
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
- Sem footprint registrado, reconstrua-o pelos commits do ID antes de agir.
