# Fase 4 — Aterrissagem (chapéu PM/PO + eng)

Fecha a fatia: o código construído vira mudança entregue, com o modelo vivo sincronizado, o rastro completo e — crucial pra evolução saudável — o **footprint** que torna a remoção futura barata.

## Passos

1. **Build/test verde.** Rode a suíte **completa** do projeto (não só os arquivos da fatia) — divergência teste↔código é sinal, conserte a causa. Não prossiga no vermelho (regra global).

2. **Sincronizar o modelo vivo.** A fatia tocou o sistema; os artefatos têm que refletir isso:
   - `docs/ddd/02-tactical.md` — se mudou agregado/VO/invariante.
   - `docs/ddd/03-architecture.md` — se criou/revisou ADR.
   - `docs/api/openapi.yaml` — se mudou contrato.
   - `docs/ui/03-design-system.md` + build-log — se mudou componente/tela.
   - O build-log da skill builder correspondente.
   Modelo que diverge do código é dívida — atualize agora, não "depois".

3. **Registrar o footprint (seção 9 do CHG).** Liste com precisão o que a fatia introduziu/tocou: código adicionado/alterado, campos/endpoints de contrato (aditivo ou breaking), dados/migração, dependências externas, e o **risco de entrelaçamento** (o quanto outras partes tendem a depender disto). Isto não é burocracia: é o que o protocolo de remoção lê depois pra decidir entre revert limpo e remoção cirúrgica. Capriche — o "eu do futuro" agradece.

4. **Revisão de defeito de código (independente).** `review-go` (Go) e/ou `code-review` (mudança significativa). Apresente achados; aplique os aprovados; documente os mantidos com justificativa.

5. **Aceite independente (separação de deveres).** Despache o agent **`feature-loop-acceptance-verifier`** — contexto fresco, cego ao raciocínio do build, **read-only** (não pode consertar nada). Passe o caminho do `CHG-NNN` e os arquivos tocados; ele valida cada critério de aceite (seção 3), incluindo **preservação**, com evidência `file:line`, e devolve PASS/FAIL/CONCERN + lacunas. É o análogo do aceite do PO: quem decidiu/construiu não é quem aprova. Distinto do passo 4 (defeito de código ≠ satisfação do prometido). **O orquestrador adjudica** o parecer (o verificador pode errar — cheque contra o código antes de aceitar um FAIL ou descartar um PASS). Lacuna real encontrada volta pra fase 3.

6. **Commit referenciando o ID.** `<tipo>: <descrição> (CHG-NNN)`. Atômico. **O humano é a autoridade de merge** — confirme antes de commit/push/PR (regra global). O agente nunca se auto-autoriza ação externa.

7. **Verificação ao vivo.** Suba o app e observe o critério de aceite acontecendo de verdade (não só teste verde). Registre o que viu na seção 8.

8. **Atualizar o registro.** Mova o CHG pra "Entregues" no `backlog.md`; status → `entregue`; preencha a seção 8 (rastros: artefatos, commit, verificação).

## Saída

- Mudança entregue, build/test verde, app verificado.
- `CHG-NNN` com seções 8 (rastros) e 9 (footprint) preenchidas; status `entregue`.
- Modelo vivo e `backlog.md` sincronizados.

## Regras

- Sem footprint, não há entrega. É o que diferencia uma feature removível de uma irreversível.
- Verificação ao vivo é obrigatória — teste verde não é a mesma coisa que feature funcionando.
- Não deixe o modelo vivo desatualizado "pra depois". A próxima fatia lê dele.
