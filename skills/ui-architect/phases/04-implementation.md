# Fase 4 — Implementação

Orquestra a construção do frontend em **cortes verticais por use case**, despachando
`ui-component-code-generator` e `ui-component-test-generator` **em paralelo** por corte. Lê:
specs da fase 3, contrato de backend (OpenAPI), arquivo de stack. Produz código +
`docs/ui/04-build-log.md`.

**Gate:** os dois agents devem estar instalados em `~/.claude/agents/`. Se faltar, PARE e peça
pra instalar (são versionados no mesmo repo da skill, em `agents/`).

## 4.0 — Fundação (uma vez)
- Scaffold da stack (`templates/stacks/<stack>.md`): Vite+React+TS, Tailwind, shadcn/ui, libs
  (TanStack Query, React Router, react-hook-form, zod).
- Gerar tipos da OpenAPI (`src/lib/api/types.gen.ts`) + script `gen:api`.
- Tokens da fase 3 → `tailwind.config`/CSS vars + tema shadcn (ponte token→produção).
- App shell: providers (QueryClient), router, base URL por env (+ proxy de dev pro backend).
- Camada base: `lib/api/client.ts` (verbos + `ApiError`), `lib/format.ts`, `lib/api/errors.ts`.
- Setup de testes: Vitest + RTL + jsdom (`src/test/setup.ts`).
- Verificar: `npm run build` verde.

## 4.1..N — Cortes verticais (um por use case/tela)
Por corte, o orquestrador prepara o **contrato** e despacha os dois agents:

1. **Contrato de dados:** escreva os hooks em `features/<x>/api.ts` (query/mutation) a partir
   dos tipos da OpenAPI. Signatures fixas — é o que os dois agents consomem.
2. **Stubs de componente:** crie os `.tsx` com a **props interface** + um esqueleto que já
   posiciona os `data-testid` declarados no spec (lógica como TODO). Isso ancora a superfície
   testável: o code-gen preenche sem mover os testids; o test-gen asserta os mesmos.
3. **Despache EM PARALELO** (uma única mensagem, dois agents — senão não há ganho de tempo):
   - `ui-component-code-generator`: specs + stack + `types.gen.ts` + `api.ts` + stubs a preencher.
   - `ui-component-test-generator`: specs + stack + `types.gen.ts` + `api.ts` + caminhos dos `*.test.tsx`.
   Eles não se veem; convergem pelo spec (superfície testável) + signatures dos hooks.
4. **Reconciliação:** rode `npm run build` e `vitest run`. Teste vermelho por selector/copy que
   não bate é **sinal**, não ruído: ou o code-gen não honrou o spec, ou o spec estava ambíguo.
   Conserte a **causa** (código ou spec), nunca afrouxe o teste pra passar. Esse sinal é o valor
   do approach (a) — verificação real, não tautologia.
5. **Checkpoint por corte:** build verde + testes + screenshot ao vivo (quando o backend estiver
   no ar). Registre no `04-build-log.md` com proveniência (cada elemento → campo do contrato).

## 4.Z — Smoke
Suíte completa (`vitest run`) + build + verificação visual ao vivo da app inteira contra o backend.

## Regras
- Cada elemento de tela rastreia a um campo do contrato (proveniência no build-log).
- Mockup aspiracional que excede o backend → demanda registrada (`docs/ddd/` backlog), não gambiarra.
- Submissão/mutação real contra dados do usuário só com ordem explícita.
- Não silencie divergência teste×código fechando o teste — investigue a causa.
