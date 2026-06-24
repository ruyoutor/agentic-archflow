# Fase 1 — UX Discovery

**Entrada:** mini-mundo (descrição do negócio) OU spec/PRD estruturado + (opcional) `docs/ddd/01-strategic.md` + (opcional) `docs/ddd/02-tactical.md`
**Saída:** `docs/ui/01-discovery.md` baseado em `templates/01-discovery.md.tmpl`

## Modo de input

Antes de conduzir o método, classifique o input recebido em **três modalidades**:

- **A. Mini-mundo puro (narrativa).** Texto descritivo do negócio, sem regras formalizadas. Conduza os 7 passos do método integralmente.
- **A. Mini-mundo enriquecido.** Texto narrativo + regras de negócio formalizadas + decisões fechadas + fora-de-escopo explícito. **Caso mais comum no mundo real.** Conduza os passos cirurgicamente: identifique o que do template já está coberto pelo input, **não refaça**, pergunte apenas o que falta (frequência de uso, contexto físico, motivação dominante, estado emocional, etc.).
- **B. Spec / PRD estruturado.** Documento formal com personas, requisitos, casos de uso explícitos. Mapeie pro template; sinalize lacunas; valide item por item antes de aceitar. Não repita discovery exploratória do zero — mas pergunte "por quê" em pontos que parecem assumidos sem rastro.

Não trate input como A puro só porque tem narrativa, nem como B só porque tem regras formalizadas. A maioria dos casos reais cai em **A enriquecido**.

## Método

Conduza o raciocínio nesta ordem. Não escreva o artefato antes de passar por todos os passos em conversa com o usuário. **Pule passos cujos dados já estão no input** — não refaça trabalho que o usuário já entregou destilado.

### 1. Entender o usuário antes de modelar

Reconte o mini-mundo focando em **quem usa o sistema e em que contexto**. Devolva ao usuário. Pergunte sobre o que estiver vago:

- Quem opera o sistema no dia-a-dia? Há mais de um tipo de usuário?
- Esses usuários são internos (funcionários, ops) ou externos (clientes, parceiros)?
- Em que contexto físico/temporal eles usam (escritório, mobile em campo, sob pressão, casual)?
- Qual a expertise técnica média no domínio?
- Qual dispositivo primário (desktop, mobile, ambos)?

Se há `docs/ddd/01-strategic.md`, leia a seção **Atores** e cruze com os usuários do recorte de UX. Se há discrepância (ex.: ator do DDD que não aparece como persona de UI ou vice-versa), sinalize.

Se o usuário não souber, **registre a lacuna**. Não invente.

### 2. Definir personas

Para cada tipo de usuário identificado, declare uma persona **funcional**:

- **Nome do papel** — descritivo, não invenção criativa ("Operador de Loja", "Cliente Comprador", não "Maria, 32 anos…")
- **Contexto de uso** — onde, quando, com que frequência
- **Expertise no domínio** — novato / intermediário / avançado
- **Dispositivo primário** — desktop / mobile / ambos
- **Motivação dominante** — eficiência? exploração? aprendizado? compliance?

Heurística: se você está inventando dados demográficos (idade, cidade, hobbies), parou de fazer trabalho de design e começou creative writing. Persona aqui é papel funcional, não personagem.

Para mini-mundos pequenos, **uma persona é OK**. Não fabrique personas pra encher.

### 3. Mapear jobs-to-be-done

Para cada persona, liste os JTBD no formato: **"Quando [situação], eu quero [resultado], para que [benefício]."**

Quando há `docs/ddd/02-tactical.md`, **cada use case ou comando vira pelo menos um JTBD**. O contrário não é verdade: pode haver JTBD de UI (visualizar, comparar, filtrar, exportar) que não corresponde a use case do tático — isso é sinal de gap no tático, que você deve registrar como ambiguidade.

JTBD ≠ feature. "Usar o dashboard" não é JTBD. "Quando recebo uma reclamação, quero ver o histórico do cliente, para decidir se ofereço desconto" é JTBD.

### 4. Esboçar jornadas principais

Selecione as **2-5 jornadas mais importantes** (não todas). Para cada uma:

- **Persona** envolvida
- **Trigger** — o que faz a pessoa começar a jornada (evento externo, decisão interna, agendamento)
- **Estado inicial** — emocional/cognitivo no momento do trigger (urgência? curiosidade? frustração?)
- **Sequência de passos** — alto nível, **não telas ainda**. ("Verifica saldo" não "abre a tela X e clica em Y".)
- **Momentos de decisão** — onde a pessoa precisa escolher um caminho
- **Momento "yay"** — sucesso percebido (não necessariamente sucesso técnico)
- **Pontos de fricção conhecidos** — o que hoje é difícil, lento, confuso

Se você está descrevendo telas e cliques, parou de fazer jornada e começou fluxo de UI — isso é fase 2.

### 5. Levantar requisitos não-funcionais de UX

- **Performance percebida** — tela carrega em <Xs, ação responde em <Xs
- **Idioma e i18n** — quais idiomas, default
- **Acessibilidade alvo** — WCAG nível (AA é base razoável; AAA quando há obrigação legal)
- **Offline / conectividade intermitente** — precisa funcionar sem rede? sincroniza depois?
- **Multi-tenant** — visual diferenciado por cliente?
- **Tema** — light/dark/ambos
- **Densidade de informação** — dashboard denso (operacional) vs experiência leve (consumer)?

Se o usuário não declarou nada explícito, registre os defaults razoáveis (português-BR, WCAG AA, online-first, single-tenant visual, light theme) **como suposições** no artefato — não como decisão silenciosa.

### 6. Levantar restrições

- **Branding/identidade visual existente** — paleta, logo, tipografia obrigatória?
- **Design system corporativo** a consumir
- **Dispositivo-alvo** — default web responsivo (desktop + mobile). Mobile nativo está fora desta skill.
- **Browsers suportados** — default: dois últimos majors de Chrome/Edge/Firefox/Safari

### 7. Escrever o artefato

Use `templates/01-discovery.md.tmpl`. Salve em `docs/ui/01-discovery.md`. Apresente ao usuário e pause.

**Seções condicionais do template:**
- **Cruzamento com DDD**: incluir somente se `docs/ddd/01-strategic.md` ou `docs/ddd/02-tactical.md` existir no target. Quando não existir, **omita a seção inteira** — não preencha com nota de "não aplicável".

**Classificação de itens em aberto:** o template tem duas seções no fim, com propósitos distintos. Não jogue tudo numa lista única.

- **Restrições e notas pra fases seguintes** — avisos que **não são decisão**, são "não invente isso" (ex.: "comparação entre duas datas arbitrárias está fora de escopo — fase 2 não deve inventar seletor de período"). Cada item identifica a fase alvo.
- **Decisões diferidas** — pontos de design que ficaram em aberto na fase 1 e devem ser fechados em uma fase específica posterior. Agrupe por fase alvo (fase 2, fase 3, fase 4).

Se você não consegue classificar um item (restrição vs decisão, ou pra qual fase ele pertence), **pergunte ao usuário** antes de despejar tudo na mesma seção.

## Anti-padrões

- **Persona de novela.** Maria, 32 anos, mãe de dois, gosta de café. Nada disso ajuda a desenhar UI. Persona = papel funcional + contexto de uso.
- **JTBD = feature.** "Usar a tela de pedidos" não é JTBD; é descrição de UI. JTBD descreve a intenção e o motivo.
- **Pular cruzamento com DDD quando ele existe.** Atores e use cases do DDD são input direto da fase 1. Ignorar é descartar trabalho já feito e arriscar incoerência.
- **Inventar requisitos não declarados.** Se o usuário não disse que precisa de offline, não escreva "requisito: offline-first". Marque como decisão pendente.
- **Cobrir todas as jornadas.** 10 jornadas iguais escondem as 2 críticas. Foque nas decisivas.
- **Desenhar telas.** Telas são fase 2. Se você está dizendo "abre o modal", parou de fazer discovery.
- **Inflar personas.** Sistema pequeno com um único tipo de usuário não precisa de 5 personas. Uma é OK.
- **Misturar restrição com decisão diferida.** Restrição avisa a fase seguinte "não invente isso"; decisão diferida aponta trabalho pra fazer depois. Misturadas viram lista única confusa.
- **Refazer o que o input já entregou destilado.** Em Modo A enriquecido ou Modo B, identifique o que falta e pergunte só isso — em vez de re-conduzir os 7 passos do zero.
