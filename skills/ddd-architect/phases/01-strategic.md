# Fase 1 — DDD Estratégico

**Entrada:** mini-mundo (descrição de negócio do usuário)
**Saída:** `docs/ddd/01-strategic.md` baseado em `templates/01-strategic.md.tmpl`

## Método

Conduza o raciocínio estratégico nesta ordem. Não escreva o artefato antes de passar por todos os passos em conversa com o usuário.

### 1. Entender o negócio antes de modelar

Antes de nomear domínios, reconte o mini-mundo com suas próprias palavras e devolva ao usuário. Pergunte sobre o que estiver vago:

- Quem são os atores / personas?
- Quais eventos de negócio importam (pedido criado, pagamento capturado, estoque ajustado…)?
- Quais invariantes o negócio assume (cliente não pode pedir sem crédito, pagamento não pode ser estornado duas vezes…)?
- Quais integrações / sistemas externos?
- Escala e pressão temporal (1 usuário, 1 tenant, multi-tenant SaaS, B2C com milhões…)?

Se o usuário não souber, **registre a lacuna**. Não invente regras de negócio.

### 2. Identificar o domínio e subdomínios

Declare o **domínio core** em uma frase: a parte onde o negócio ganha dinheiro ou se diferencia. Depois classifique o resto:

- **Core** — diferencial competitivo, merece o máximo esforço de modelagem
- **Supporting** — necessário mas não diferencia (pondere build vs buy)
- **Generic** — problema resolvido (auth, pagamento, notificação) — prefira SaaS / biblioteca

Quando estiver em dúvida sobre uma classificação, pergunte ao usuário.

### 3. Estabelecer a linguagem ubíqua

Para cada subdomínio, liste os termos que o negócio usa, com o significado **como o negócio usa**, não como você adivinha pelo nome. Surface conflitos: "Pedido" significa o mesmo em Vendas e em Faturamento? Esses conflitos são pistas de fronteiras de contexto.

### 4. Identificar bounded contexts

Um bounded context é uma fronteira dentro da qual um modelo é consistente. Heurísticas:

- Termo que significa coisas diferentes em partes diferentes do negócio → provável fronteira
- Fronteira de time / departamento muitas vezes (mas nem sempre) corresponde a um contexto
- Ciclos de vida diferentes ("Produto" no Catálogo vs "Produto" no Estoque)

Para mini-mundos pequenos, **um contexto único é OK**. Não fabrique fronteiras.

### 5. Desenhar o mapa de contextos

Para cada par de contextos que interage, declare a relação:

- **Partnership** — times coordenam mudanças
- **Customer/Supplier** — downstream depende do roadmap do upstream
- **Conformist** — downstream aceita o modelo do upstream como está
- **Anticorruption Layer (ACL)** — downstream traduz para proteger seu modelo
- **Open Host Service / Published Language** — upstream oferece contrato estável
- **Shared Kernel** — pequeno modelo compartilhado (use com parcimônia)
- **Separate Ways** — sem integração

### 6. Escrever o artefato

Use `templates/01-strategic.md.tmpl`. Salve em `docs/ddd/01-strategic.md`. Apresente ao usuário e pause.

## Anti-padrões

- **Subdomínio ≠ bounded context.** Subdomínio é espaço de problema; bounded context é fronteira de modelo. Frequentemente alinham, mas não sempre.
- **Mega-contexto único.** Se você empurrou tudo para um contexto, está pulando o trabalho. Procure conflitos de termos com mais cuidado.
- **Fragmentação prematura.** Sem evidência de modelos distintos, não divida. Custo de integração é real.
