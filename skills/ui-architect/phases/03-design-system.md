# Fase 3 — Design system

Produz `docs/ui/03-design-system.md`: **tokens**, **catálogo de componentes** e **um spec por
componente/tela não-trivial** — sendo os specs o contrato que viabiliza os agents de código e
teste em paralelo na fase 4. Lê: a fase 2 (IA & fluxos), o **contrato de backend** (gate da
ativação) e, se houver, `docs/ddd/`.

## Método

### 1. Tokens (processo artifact-design)
Carregue a skill `artifact-design` e siga o processo: brainstorm de um sistema de tokens
(cor / tipo / layout) → autocrítica contra os defaults de IA → fecha a paleta. Ancore no
**subject** (o mini-mundo), não em template. Registre os tokens como **CSS vars** — eles viram
`tailwind.config` + tema shadcn na fase 4 (a ponte token→produção; o mesmo token do protótipo
vira o tema real).

Precedência: se já existe design system no projeto (CLAUDE.md, arquivo de tokens/tema), ele
manda — o processo só preenche lacunas.

### 2. Catálogo de componentes
Liste os átomos (primitivas shadcn usadas) + os compostos próprios + as telas do mapa (fase 2).
Calibre pela complexidade do mini-mundo — não superdesenhe.

### 3. Spec por componente (o contrato de convergência)
Para cada componente/tela **não-trivial**, escreva um spec com `templates/component-spec.md.tmpl`.
É a peça que torna o approach (a) possível — os dois agents da fase 4 convergem por ele sem se
ver. Cada spec pina:
- **props** (interface TS; dados referenciam `types.gen.ts`);
- **dados & proveniência** (hook + endpoint + campo→elemento);
- **estados** (loading/vazio/erro/carregado, quando se aplicam);
- **superfície testável** — `data-testid`, copy-chave, roles/nomes acessíveis.

Regra de ouro: **se o teste vai assertar, o spec declara.** Copy e selectors frouxos = agents
divergem na fase 4. Átomo shadcn usado direto não precisa de spec próprio.

## Saída
`docs/ui/03-design-system.md` com tokens (CSS vars) + catálogo + specs (ou specs em
`docs/ui/specs/<componente>.md` se forem muitos). Os specs são **input read-only da fase 4**.

## Checkpoint
Salve, apresente, pause. O usuário revisa tokens e specs — em especial a **superfície testável**,
que amarra código e teste. Ajuste aqui é barato; divergência descoberta na fase 4 é cara.
