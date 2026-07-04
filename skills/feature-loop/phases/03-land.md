# Fase 3 — Aterrissagem (chapéu PO — verifica e comita, não constrói)

Fecha a fatia: o que a builder entregou (`construída`, sem commit) vira mudança entregue, com o
modelo vivo auditado, o rastro completo e o **footprint** que torna a remoção futura barata. O PO
verifica e comita — **não conserta código**: lacuna encontrada aqui volta pra builder.

## Entrada

- `CHG-NNN` com seções 6–7 preenchidas pela builder (análise aprovada no checkpoint) e o relatório
  de entrega dela (diff, footprint real, resultados de teste, revisões rodadas).

## Passos

1. **Build/test verde.** Rode a suíte **completa** do projeto você mesmo (não confie só no relatório).
   Vermelho → volta pra builder com o sinal; não aterrisse no vermelho.

2. **Auditar a sincronização do modelo vivo.** A builder evoluiu os próprios artefatos (é a fase E3
   dela); aqui se **confere**, não se escreve:
   - `docs/ddd/02-tactical.md` — reflete agregado/VO/invariante mudado? Entrada no histórico de revisões com o CHG?
   - `docs/ddd/03-architecture.md` — ADR criado/revisado registrado?
   - `docs/api/openapi.yaml` — contrato bate com o código?
   - `docs/ui/03-design-system.md` + specs — componente/tela tocado refletido?
   - Build-log da builder — seção da fatia presente?
   Artefato dessincronizado é entrega incompleta da builder — devolva pra ela fechar. A feature-loop
   só edita `docs/changes/`.

3. **Registrar o footprint (seção 9 do CHG).** Consolide, do relatório da builder + `git diff`:
   código adicionado/alterado, campos/endpoints de contrato (aditivo ou breaking), dados/migração,
   dependências externas, e o **risco de entrelaçamento**. É o que o protocolo de remoção lê depois
   pra decidir entre revert limpo e remoção cirúrgica. Capriche — o "eu do futuro" agradece.

4. **Conferir a revisão de defeito de código.** O relatório da builder deve mostrar `review-go` /
   `code-review` rodados nos checkpoints dela. Se a mudança é significativa e não foi coberta, rode
   `code-review` agora. Achados que exigem mudança de código **voltam pra builder** (retrabalho na
   fase E2 dela); achados aceitos-como-estão ficam documentados com justificativa no CHG.

5. **Aceite independente (separação de deveres).** Despache o agent **`feature-loop-acceptance-verifier`**
   — contexto fresco, cego ao raciocínio do build, **read-only**. Passe o caminho do `CHG-NNN` e os
   arquivos tocados; ele valida cada critério de aceite (seção 3), incluindo **preservação**, com
   evidência `file:line`, e devolve PASS/FAIL/CONCERN + lacunas. **O orquestrador adjudica** o parecer
   (o verificador pode errar — cheque contra o código antes de aceitar um FAIL ou descartar um PASS).
   Lacuna real confirmada volta pra builder.

6. **Commit referenciando o ID.** `<tipo>: <descrição> (CHG-NNN)`. Atômico. **O humano é a autoridade
   de merge** — confirme antes de commit/push/PR (regra global). O agente nunca se auto-autoriza ação
   externa.

7. **Verificação ao vivo.** Suba o app e observe o critério de aceite acontecendo de verdade (não só
   teste verde). Registre o que viu na seção 8.

8. **Atualizar o registro.** Mova o CHG pra "Entregues" no `backlog.md`; status → `entregue`; preencha
   a seção 8 (rastros: artefatos, commit, verificação).

## Saída

- Mudança entregue, build/test verde, app verificado.
- `CHG-NNN` com seções 8 (rastros) e 9 (footprint) preenchidas; status `entregue`.
- Modelo vivo auditado e `backlog.md` sincronizado.

## Regras

- **Aterrissagem não conserta.** Qualquer lacuna (teste vermelho, artefato dessincronizado, achado de
  review, FAIL de aceite) volta pra builder — o PO não abre código nem "dá um jeitinho".
- Sem footprint, não há entrega. É o que diferencia uma feature removível de uma irreversível.
- Verificação ao vivo é obrigatória — teste verde não é a mesma coisa que feature funcionando.
- Artefato vivo dessincronizado não passa — a próxima fatia lê dele.
