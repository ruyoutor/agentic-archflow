# Fase 2 — Arquitetura de Informação & Fluxos

Traduz o discovery (fase 1) num **mapa de telas**, um **modelo de navegação** e os **controles
transversais** — a estrutura que a fase 3 vai detalhar e a fase 4 implementar. Lê: `01-discovery.md`
e o **contrato de backend** (gate da ativação). Produz `docs/ui/02-ia-flows.md` (+ wireframes
Excalidraw quando ajudarem).

> Gate herdado: cada tela/elemento aqui rastreia a uma capacidade do contrato. Não desenhe o que
> o backend não entrega; capacidade ausente vira demanda (`docs/ddd/` backlog), não tela fantasma.

## Método

### 1. Mapa de telas
Uma tela por **job-to-be-done primário** / use case — calibre pelo mini-mundo (não infle).
Para cada tela registre: o JTBD que atende, o(s) endpoint(s) que consome, e os **estados** que
precisa tratar (loading / vazio / erro / carregado). Marque a tela primária (onde o usuário cai).

### 2. Modelo de navegação
- Rotas e hierarquia (ex.: consolidado `/` → detalhe `/ativo/:id` → ...).
- **Drill-down**: decida por tela — navegação (rota) vs. expansão in-place. Registre o porquê.
- Estado de entrada de cada rota (ex.: detalhe precisa de id + parâmetro transversal na URL).

### 3. Controles transversais (cross-cutting)
Interações que valem em várias telas **não são telas** — vivem no app shell. Para cada um decida:
onde mora, e **persistência** (memória de sessão? query param linkável? volta ao default?).
Ex.: seletor de benchmark como `?b=` na URL (persiste e é compartilhável); botão "atualizar agora".

### 4. Fluxos principais
Para cada jornada da fase 1, descreva a sequência de telas/ações e os **momentos de decisão**.
Aponte onde uma mutação acontece e o que o usuário vê depois (feedback, invalidação).

### 5. Wireframes (opcional, low-fi)
Quando a navegação for não-óbvia, esboce em Excalidraw. Low-fi — sem cor/tipo (isso é fase 3).
Calibre: tela simples não precisa de wireframe.

## Não-objetivos desta fase
- Não escolher cores/tipografia/componentes (fase 3).
- Não inventar telas para capacidades inexistentes no backend — sinalize o gap.
- Não criar seletores/filtros que o backend não suporta (ex.: período arbitrário se a API é
  snapshot-only). Respeite as não-capacidades do contrato.

## Saída
`docs/ui/02-ia-flows.md`: mapa de telas (com endpoints + estados), modelo de navegação,
controles transversais (com decisão de persistência), fluxos. Input read-only da fase 3.

## Checkpoint
Salve, apresente, pause. O usuário valida o mapa e a navegação antes de entrar em tokens/design.
