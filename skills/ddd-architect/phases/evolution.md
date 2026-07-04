# Modo evolução — uma fatia sobre o sistema existente (entrada: CHG-NNN)

Brownfield. A `feature-loop` (PO) escreveu uma história em `docs/changes/CHG-NNN-<slug>.md` e o
usuário roteou o card pra cá. Este modo responde, **com o código à mão**, as duas perguntas que o PO
não responde: **o que precisa mudar para que o pedido funcione?** e **quais impactos/trade-offs
considerar e decidir?** — e só então constrói, cirurgicamente, dentro da fatia.

As fases greenfield (1–4) não rodam aqui; são o repertório. Contratos do modo:
- **Editar, não reescrever.** Reescrita de arquivo existente só quando uma edição não resolve — e dito explicitamente.
- **Escopo = a fatia.** O agregado/use-case/adapter que ela toca. Refatoração fora da fatia é demanda nova no backlog, não carona.
- **Sem commit.** A entrega é o diff + artefatos evoluídos + relatório; quem aterrissa (aceite independente + commit) é a `feature-loop`.
- **Suíte inteira, sempre.** Preservação é critério de aceite; a suíte completa é o detector.

## E0 — Ler tudo antes de opinar

1. **A história:** o `CHG-NNN` completo — JTBD, contexto de produto, critérios de aceite (incluindo
   preservação), fatia, ambiguidades **5b** (as técnicas, que são suas), mockups se houver.
2. **O modelo vivo:** `docs/ddd/01..03`, OpenAPI, e o build-log (o que as fatias anteriores tocaram).
3. **O código na vizinhança da fatia:** o agregado/serviço/use-case/adapters afetados, e seus testes.
   Vá ao código, não só aos docs — doc atrás do código é um achado: registre como divergência/dívida,
   não silencie.
4. **A stack:** `templates/stacks/<stack>.md` (default `go`).

## E1 — Análise de impacto & desenho da mudança

**Ler o modelo dissolve ambiguidade.** Antes de deliberar qualquer item 5b, cheque se o código/tático
já responde — dúvida que parecia difícil no abstrato muitas vezes já está decidida no modelo, e a
"decisão" vira só expor o que existe.

1. **O que precisa mudar** — por camada, escrito na seção 6 do CHG:
   - **Domínio:** agregados/VOs/invariantes tocados. Muda invariante? Novo cálculo? Novo estado?
   - **Aplicação:** use cases/comandos novos ou alterados.
   - **Adapters:** persistência (migração de schema?), http, parsers, integrações.
   - **Arquitetura:** toca/cria/revisita ADR?
   - **Contrato:** endpoint/campo novo ou alterado — **aditivo ou breaking**.
   - **E explicitamente o que NÃO muda** — a proteção do critério de preservação.

2. **Quais trade-offs decidir** — deliberação como sparring partner: pra cada decisão genuína
   (alternativas de modelagem, versionar vs. substituir comportamento, migração), **traga opções,
   exponha trade-offs e recomende** — não devolva a pergunta crua. O resíduo vira entrada no log de
   decisões do CHG (mini-ADR: pergunta → opções pesadas → decisão → porquê). Ambiguidades 5b todas
   endereçadas; a que sobrar sem decisão **bloqueia** a construção.

3. **Plano de build (seção 7):** calibragem (ver E2), ordem, e o **footprint previsto** — o que a
   fatia vai adicionar/tocar, pra aterrissagem só confirmar.

4. **CHECKPOINT DURO — apresente e PARE.** Status do CHG → `em análise de impacto`, checkpoint
   `pendente`. O usuário pode:
   - **"segue"** → registre a aprovação no CHG e vá pra E2;
   - **ajustar** decisões da análise → incorpore e reapresente;
   - **devolver o card ao refinamento** — se a análise revelou lacuna de **produto** (critério
     ambíguo, história subespecificada), anote a lacuna no checkpoint do CHG, status →
     `em refinamento`, e pare de vez. Lacuna de produto não se remenda aqui — é da `feature-loop`.

   **Nunca construa sem o "segue" explícito.** Se a análise revelar mudança grande demais pra uma
   volta (novo agregado central, ADR pesado), recomende quebrar em fatias no backlog em vez de inchar esta.

## E2 — Construção cirúrgica (após o "segue"; status → `em construção`)

**Calibre o mecanismo pelo peso da fatia** — o menor que faz o trabalho com qualidade. A calibragem
mora aqui (dentro de quem constrói), não no PO:

| Peso | Caminho |
|---|---|
| Trivial (campo derivado, ajuste de regra pontual) | edite direto + testes. Dispatchar agents aqui é overhead. |
| Média (use-case novo, operação nova em agregado existente) | despache os agents (`ddd-domain-*`, `ddd-app-*`) **escopados aos arquivos da fatia**. |
| Substancial (agregado novo, mudança de modelagem, migração) | rode as sub-fases 4.x aplicáveis em modo evolução — só o contexto/camadas afetados. |

Registre a calibragem escolhida no build-log — é a lição da próxima fatia.

- **Agents em modo edição:** passe caminhos dos arquivos **existentes** a modificar (não stubs a
  sobrescrever), a spec revisada (tático já atualizado ou a seção 6 do CHG) e a instrução explícita
  de preservar o que está fora da fatia. Testes ∥ código continuam cegos entre si, convergindo pelos
  contratos.
- **Ordem:** domínio → aplicação → adapters → contrato (OpenAPI). Fatia full-stack: o front (via
  `ui-architect`) nunca precede o contrato real.
- **Protocolo de divergência** (fase 4) continua valendo: mismatch teste↔código se apresenta, não se
  reconcilia em silêncio. Divergência com **código preexistente** idem: se o teste antigo contradiz o
  novo comportamento aprovado, a mudança do teste é decisão explícita, registrada.
- **Verde total:** `go build/vet/test ./...` (ou equivalente) na suíte **inteira**. Teste antigo
  vermelho é sinal de preservação violada — conserte a causa ou volte ao checkpoint.
- **Revisões de checkpoint** (protocolo da skill): `review-go` e `code-review` quando aplicável.

## E3 — Evoluir os artefatos + relatório de entrega

O modelo vivo é atualizado **por quem construiu**, agora — não "depois", não pelo PO:

1. **Editar** (não regenerar) os artefatos tocados:
   - `docs/ddd/02-tactical.md` — agregados/VOs/invariantes/comandos como ficaram.
   - `docs/ddd/03-architecture.md` — ADR novo ou revisado.
   - `docs/api/openapi.yaml` — o contrato como está servido.
2. **Histórico de revisões:** em cada artefato tocado, uma entrada `{{data}} · CHG-NNN · o que mudou
   · por quê`. Se o artefato ainda não tem a seção, crie-a no rodapé.
3. **Build-log:** seção da fatia (modo evolução), com calibragem, resultados de build/test, arquivos
   tocados, decisões e ajustes pós-review.
4. **Relatório de entrega** (no chat + apontado no CHG): diff resumido, footprint **real** vs.
   previsto, resultados da suíte, revisões rodadas e achados. Status do CHG → `construída`.
5. **PARE. Sem commit.** Aterrissagem (aceite independente, auditoria, commit) é da `feature-loop`,
   com o usuário como autoridade de merge.

## Regras

- As duas perguntas antes de qualquer código: *o que muda?* e *que trade-offs?* — escritas no CHG,
  aprovadas no checkpoint. Análise não escrita = análise não feita.
- Escopo é a fatia. A vontade de refatorar o vizinho vira demanda registrada, não carona.
- Artefato dessincronizado no fim da E3 é entrega incompleta — a aterrissagem devolve.
- Tipo **remoção**: a E1 vira análise reversa (footprint → dependentes → limpo/entrelaçado) — método
  em `feature-loop/phases/removal.md`; mesmos checkpoints, mesma entrega sem commit.
