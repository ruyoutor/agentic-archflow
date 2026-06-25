# Stack: Vite + React + TS + Tailwind + shadcn/ui

Convenções de **como o código frontend é construído** nesta stack. É o documento que os
agents de implementação (`ui-component-code-generator` / `ui-component-test-generator`)
consultam na fase 4 e que o orquestrador segue ao fazer scaffold. Reutilizável entre
projetos; o que é específico do projeto (tokens, telas) vive no `docs/ui/03-design-system.md`.

> **Versões:** não fixe versões aqui. No scaffold, instale as últimas estáveis e pine no
> `package.json`. Convenções de API de framework envelhecem — confirme detalhes da versão
> corrente na doc oficial (via context7) no momento do scaffold, não de memória.

## Stack e porquês

| Camada | Escolha | Porquê |
|---|---|---|
| Build/dev | **Vite** | HMR rápido, zero-config TS, SPA. Sem SSR (app de leitura autenticado/local). |
| UI | **React + TypeScript** (strict) | `tsconfig` com `strict: true`, sem `any` implícito. |
| Estilo | **Tailwind** | utilitários + tokens via CSS vars. |
| Componentes | **shadcn/ui** | primitivas copiadas pro repo (`components/ui/`), não dependência opaca. Tema por CSS vars. |
| Estado de servidor | **TanStack Query** | cache, revalidação, loading/error declarativos. É o coração de um app que é 90% leitura de API. |
| Formulários | **react-hook-form + zod** | validação tipada, schema = fonte da verdade do form. |
| Roteamento | **React Router** | rotas para drill-down (consolidado → ativo → lançamento). |
| Testes | **Vitest + React Testing Library** | mesmo motor do Vite; testa comportamento, não implementação. |

## Estrutura de pastas — cortes verticais por feature

Organize por **feature/use-case**, não por tipo de arquivo. Cada feature é um corte vertical
que pode ser construído e testado isolado (alinhado ao "cortes verticais" da fase 4).

```
web/
  src/
    app/                 # shell: providers, router, layout, tema
      App.tsx
      router.tsx
      providers.tsx      # QueryClientProvider, etc.
    features/
      portfolio/         # uma feature por área de negócio
        api.ts           # hooks de query/mutation (usePortfolioSnapshot, ...)
        components/      # componentes da feature (ReguaGanho, PositionsTable, ...)
        PortfolioPage.tsx
      trades/
      asset-detail/
    components/
      ui/                # primitivas shadcn (geradas)
      shared/            # componentes transversais (StaleBadge, MoneyText, BenchmarkToggle)
    lib/
      api/
        client.ts        # fetch wrapper + base URL via env
        types.gen.ts     # tipos GERADOS da OpenAPI — não editar à mão
        errors.ts        # mapeia ErrorResponse.error → mensagem pt-BR
      format.ts          # dinheiro, percentual, datas (pt-BR)
      tokens.css         # CSS vars dos design tokens
    test/
      setup.ts
  package.json           # próprio — build independente do backend
  vite.config.ts
  .env.example           # VITE_API_BASE_URL=http://localhost:8080/api
```

## Camada de dados — o contrato é o único acoplamento

- **Tipos gerados da OpenAPI.** Gere `lib/api/types.gen.ts` a partir do `openapi.yaml`
  (ex.: `openapi-typescript`). Nunca redigite DTO à mão — o contrato é a fonte. Script
  `npm run gen:api` regenera; rode quando a OpenAPI mudar.
- **Client HTTP fino** (`lib/api/client.ts`): wrapper sobre `fetch`, base URL de
  `import.meta.env.VITE_API_BASE_URL`, parse de `ErrorResponse` em erro tipado. Sem
  acoplamento a código do backend — só HTTP. Isso é o que mantém o `web/` extraível.
- **TanStack Query por uso:** cada use case vira um hook em `features/<x>/api.ts`
  (`usePortfolioSnapshot(benchmark)`, `useImportTrades()`, ...). Query keys incluem os
  parâmetros (`['portfolio', benchmark]`) pra cache correto na troca de benchmark.
- **Mutations invalidam o que afetam:** registrar/editar/ignorar trade → invalida
  `['portfolio']` e `['trades']` (o consolidado reflete na hora).

## Dinheiro e formatação (proveniência do contrato)

- **Dinheiro vem em centavos** (`MoneyDTO.amount: integer`). Nunca faça aritmética em float
  de reais; opere em centavos e formate só na borda. `R$ 37,85 = 3785`.
- `lib/format.ts`: `formatBRL(cents)` via `Intl.NumberFormat('pt-BR', {style:'currency'})`;
  `formatPct(ratio)`, `formatSignedBRL(cents)`.
- **`percentageDifference` é razão** (ex.: `0.0473`), não percentual pronto — **multiplique
  por 100 na exibição**. (O exemplo `4.73` na OpenAPI está enganoso; o `dto.go` serializa a
  razão crua. Sinalizado pra correção da OpenAPI.)
- **`absoluteDifference` inclui realizado** (`currentValue + realizados − equivalente`),
  conforme o código Go (ADR-011), apesar da descrição abreviada na OpenAPI.
- **Datas de calendário em UTC.** Campos de data (ex.: `tradedAt`) trafegam como `...T00:00:00Z`.
  Formate com `timeZone: 'UTC'` e fatie `slice(0,10)` para inputs `type="date"` — formatar no
  fuso local desloca o dia em fusos atrás de Greenwich (UTC-3 → dia anterior; bug UI-002).

## Tokens → Tailwind / shadcn

- Os design tokens (cores, raio, escala) saem da fase 3 e viram **CSS vars** em
  `lib/tokens.css` no `:root`.
- `tailwind.config` referencia as vars (`colors: { accent: 'var(--accent)' }`), e o tema do
  shadcn é fiado nas **mesmas** vars. Token é o contrato protótipo→produção (mesma régua do
  mockup vira o tema real).
- Light theme por default (sem dark a menos que o discovery peça).

## Componentes

- Componha a partir das primitivas shadcn (`components/ui/`). Não reescreva botão/input do zero.
- Um componente por arquivo, co-localizado na feature. Nome reflete o que o usuário vê
  (`StaleBadge`, `BenchmarkToggle`), não o mecanismo.
- Componentes de dado recebem dados já tipados (do hook), não fazem fetch dentro — separe
  "container" (usa o hook) de apresentação quando ajudar o teste.

## Estados: loading, vazio, erro — obrigatórios por tela

Toda tela que lê API trata os três, sem exceção (a UI nunca renderiza fingindo dado):

- **Loading:** skeleton, não spinner solto, em tabelas/cards densos.
- **Vazio:** convite à ação ("Nenhuma trade ainda — importe do Investidor10"), não beco sem saída.
- **Erro `503` (`quote_unavailable` / `factor_series_unavailable`):** estado específico que
  chama a ação de refresh ("Cotações desatualizadas — atualize pra ver o consolidado").
- **Dado desatualizado:** `benchmarkAsOf` vira indicador visível de defasagem (badge/carimbo),
  honesto sobre o atraso (IPCA pode estar semanas atrás). Nunca esconda a defasagem.

## Acessibilidade e motion

- WCAG AA: foco visível, navegação por teclado, labels em controles, contraste.
- Respeite `prefers-reduced-motion` — animações (count-up, desenho de barra) só com motion ligado.
- Tabela densa no desktop; no mobile, layout enxuto (consolidado primeiro, drill-down sob demanda).

## Erros da API → mensagem

`lib/api/errors.ts` mapeia cada `ErrorResponse.error` (snake_case) pra mensagem pt-BR na voz
da interface. Erro explica o que houve e como resolver; não pede desculpa, não fica vago.
Ex.: `quote_unavailable` → "Cotação indisponível. Atualize as cotações e tente de novo."

## Testes (guia para o test-generator)

- **Vitest + RTL.** Testa comportamento observável: o que o usuário vê e faz, não estado interno.
- Por feature: render com dados fake do hook (mock do client, não do `fetch` global quando der),
  asserta os três estados (loading/vazio/erro) + o caminho feliz.
- Formatação (centavos→R$, razão→%) tem teste unitário em `lib/format.ts` — é onde bug de
  dinheiro se esconde.
- Não testar shadcn/ui (é vendado/terceiro). Testar a composição e a lógica da feature.

## Separabilidade (monorepo → repo próprio)

O `web/` é autossuficiente: build/deps próprios, conversa com o backend só por HTTP via a
OpenAPI, base URL por env. Extrair pra repo separado depois = mover a pasta (preservando
histórico com `git filter-repo`/subtree) + CI próprio. Nenhum acoplamento de build-time.
