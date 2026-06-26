# Fase 3 — Construção (handoff pra camada de desenvolvimento)

É aqui que a `feature-loop` para de decidir e **sensibiliza as skills builder**. As fases 1–2 precedem
a construção (decisão pura); a 3 é o handoff; a 4 volta pra orquestração. Esta fase produz **só o
diff** da fatia — não comita (a fase 4 aterrissa).

## Contrato de delegação

A `feature-loop` passa pra builder, como entrada: o `CHG-NNN` (com impacto e plano), o **footprint
previsto**, e a restrição "**modo incremental, sem commit**":
- **Incremental:** toca só o agregado/use-case/componente da fatia; **edita**, não reescreve; roda os
  testes da suíte inteira na reconciliação (divergência teste↔código é sinal).
- **Sem commit:** entrega o diff; quem comita/abre PR/atualiza docs é a `feature-loop` (fase 4). Isso
  mantém o CHG como unidade de commit e evita commit duplo.

## Calibrar o peso (não use canhão pra matar mosquito)

| Peso da fatia | Caminho |
|---|---|
| Trivial (campo derivado, ajuste de cópia, flag) | a `feature-loop` edita direto + roda testes. Dispatchar agents aqui é overhead inútil. |
| Média (componente novo, use-case novo simples) | dispara os **agents** dedicados (`ddd-*`, `ui-component-*`) sobre os arquivos da fatia. |
| Substancial (agregado novo, fluxo de telas, redesenho de contrato) | delega à **builder completa** em modo incremental. |

A regra: o menor mecanismo que faz o trabalho com qualidade. Registre no build-log quando calibrar
pra baixo — é a lição que evita superengenharia na próxima fatia.

## Ordem

Mesma convergência do greenfield quando a fatia é full-stack: **domínio → contrato (OpenAPI) → front**.
O front nunca precede o contrato real (o gate da `ui-architect` continua valendo).

## Saída

- Diff da fatia (código + testes), build/test verde localmente.
- Nada commitado ainda; pronto pra fase 4 aterrissar.

## Regras

- A `feature-loop` não escreve domínio/UI quando a fatia é média/substancial — delega.
- Builder em modo evolução **não comita**. Sempre.
- Preserve o que o critério de aceite mandou preservar — rode a suíte inteira, não só a fatia.
