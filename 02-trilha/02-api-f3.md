# 03 — API Bug Bounty (F3)

[🏠 Início](../README.md) › [02 · Trilha](README.md) › API Bug Bounty (F3)

**Sumário:**
- [🛰️ F3 — API Bug Bounty](#-f3--api-bug-bounty) (fundamentos + OWASP API Top 10 2023, recon de API, labs, GraphQL)
- [🎯 Missão F3 — "Trinca de labs + teste real"](#-missão-f3--trinca-de-labs--teste-real)
- [✅ Checklist F3](#-checklist-f3)

---

> APIs são a maior superfície de ataque moderna e onde a metodologia de 2026 mais rende: **automação descobre os endpoints escondidos; o humano quebra a autorização e a lógica**. Releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase).

---

# 🛰️ F3 — API Bug Bounty

**Duração:** 2–4 meses · **Carga:** ~15h/sem · **Badge:** 🛰️ *Caçador de API*

**Foco:** dominar a caça em APIs REST e GraphQL, com forte ênfase em **autorização** (onde estão os bugs que pagam) e em **recon de API** (onde a automação e a IA acham a superfície que ninguém documentou).

> **Por que API é tão fértil:** o front-end é só uma das formas de falar com a API. Endpoints antigos (`/api/v1`), métodos não usados pela UI, parâmetros não documentados e versões legadas vivem expostos — e scanner genérico raramente entende a **autorização** por trás deles. É terreno de caçador.

<details>
<summary>📚 <strong>Estuda</strong> — fundamentos de API + a OWASP API Top 10 (2023)</summary>

**Fundamentos:**
- REST: recursos, métodos (GET/POST/PUT/PATCH/DELETE), status, versionamento (`/v1`, `/v2`), content negotiation.
- Autenticação/autorização em API: API keys, OAuth 2.0, JWT (e seus abusos), sessões.
- Documentação: OpenAPI/Swagger, Postman collections — quando expostas, são um mapa do tesouro.
- Referência: [OWASP API Security](https://owasp.org/API-Security/), [Hacking APIs (Corey Ball)](https://nostarch.com/hacking-apis), [APIsec University](https://academy.apisec.ai/api-pentesting-fundamentals).

**OWASP API Security Top 10 (2023) — saiba testar cada uma:**

| # | Risco | Essência | Como testar (manual) |
|---|---|---|---|
| **API1** | Broken Object Level Authorization (**BOLA**) | Acessar objetos de outros via ID | Trocar IDs entre duas contas; **Autorize** |
| **API2** | Broken Authentication | Auth fraca/quebrada | JWT fraco, reset de senha, brute, tokens |
| **API3** | Broken Object Property Level Authorization (**BOPLA**) | Ler/escrever propriedades além do permitido | Mass assignment + exposição excessiva de dados |
| **API4** | Unrestricted Resource Consumption | Sem rate limit / custo | Flood controlado, paginação abusiva, payloads grandes |
| **API5** | Broken Function Level Authorization (**BFLA**) | Chamar funções de outro papel | Endpoint de admin como user; trocar verbo HTTP |
| **API6** | Unrestricted Access to Sensitive Business Flows | Abuso de fluxo de negócio | Automatizar compra/voto/convite além do razoável |
| **API7** | Server Side Request Forgery (**SSRF**) | API busca URL controlada | Parâmetros de URL/webhook; Collaborator (blind) |
| **API8** | Security Misconfiguration | Config insegura | CORS, headers, métodos abertos, verbose errors |
| **API9** | Improper Inventory Management | Endpoints/versões esquecidos | `/v1` legado, hosts de staging, docs expostas |
| **API10** | Unsafe Consumption of APIs | Confiar em API de terceiro | Dados de terceiros sem validação; cadeias |

</details>

<details>
<summary>🧪 <strong>Pratica</strong> — recon de API + labs + alvo real</summary>

**Recon específico de API (automação + IA):**
- Extraia endpoints de API do **JS do front** (LinkFinder/JSluice — [motor de recon](../01-recon/recon-engine-ia-e-automacao.md)) e de **docs expostas** (Swagger/OpenAPI).
- [kiterunner](https://github.com/assetnote/kiterunner): brute force de rotas de API com wordlists especializadas (entende padrões REST melhor que ffuf).
- Descubra **parâmetros escondidos** com [arjun](https://github.com/s0md3v/Arjun) / [x8](https://github.com/Sh1Yo/x8).
- **IA aplicada à API:** cole o JS/Swagger e peça para reconstruir a superfície ("liste todos os endpoints, métodos, parâmetros e o provável modelo de autorização"); peça para gerar uma Postman collection a partir das rotas; peça hipóteses de BOLA/BFLA por endpoint.

```bash
# Brute de rotas de API com kiterunner
kr scan https://api.exemplo.com -w routes-large.kite -x 20

# Descoberta de parâmetros escondidos num endpoint REST
arjun -u https://api.exemplo.com/v2/account -m GET,POST
```

**Quebra de autorização (o coração da F3):**
- **BOLA/API1:** com duas contas (A e B), pegue um request de A, troque o ID/objeto para o de B no Repeater. **Autorize** automatiza a comparação.
- **BFLA/API5:** pegue um endpoint de admin e chame com token de user comum; tente trocar o verbo HTTP (`GET`→`PUT`/`DELETE`).
- **Mass assignment/BOPLA:** adicione campos extras no body (`"role":"admin"`, `"isVerified":true`) e veja se a API aceita.

```http
PATCH /api/v2/users/me HTTP/1.1
Host: api.exemplo.com
Authorization: Bearer <token_de_user_comum>
Content-Type: application/json

{ "name": "teste", "role": "admin", "account_balance": 999999 }
```
*Se a resposta refletir `role: admin`, você tem mass assignment (BOPLA).*

**Labs obrigatórios:**
```bash
# crAPI — API vulnerável realista da OWASP (BOLA, mass assignment, JWT)
git clone https://github.com/OWASP/crAPI && cd crAPI && docker compose up -d
# VAmPI — API REST com flag liga/desliga de vulnerabilidades
docker run -d -p 5000:5000 erev0s/vampi
# DVGA — Damn Vulnerable GraphQL App
docker run -d -p 5013:5013 dolevf/dvga
```

</details>

<details>
<summary>🧬 <strong>GraphQL — uma fronteira própria</strong></summary>

GraphQL muda as regras: um único endpoint, queries flexíveis, e vetores próprios.

- **Introspection:** se habilitada, expõe o schema inteiro (todos os tipos, queries, mutations). Primeiro passo sempre.
- **Batching & aliases:** mandar muitas operações numa request — ótimo para **brute force** e para burlar rate limit (ex.: testar N senhas via aliases).
- **Injection & autorização:** os mesmos BOLA/BFLA aparecem em resolvers; injeções fluem para o backend.
- **Ferramentas:** [InQL](https://github.com/doyensec/inql) (extensão Burp), [graphql-cop](https://github.com/dolevf/graphql-cop) (audita misconfig), [graphw00f](https://github.com/dolevf/graphw00f) (fingerprint do engine), [GraphQL Voyager](https://github.com/IvanGoncharov/graphql-voyager) (visualiza o schema).

```graphql
# Introspection mínima: o schema inteiro cabe aqui
{ __schema { types { name fields { name } } } }
```

```bash
# Fingerprint do engine GraphQL + auditoria de misconfig
python3 graphw00f.py -d -f -t https://api.exemplo.com/graphql
graphql-cop -t https://api.exemplo.com/graphql
```

> 💡 **IA + GraphQL:** cole o resultado da introspection e peça para a IA listar as mutations mais sensíveis (criação de usuário, mudança de papel, transferência) e sugerir testes de BOLA/BFLA para cada uma.

</details>

<details>
<summary>✍️ <strong>Posta</strong></summary>

- Writeup metodológico: "como eu mapeio a superfície de uma API moderna" (do JS/Swagger ao endpoint escondido).
- Quando tiver bug de API aceito e divulgável (pós-fix), escreva — bugs de BOLA/BFLA rendem ótimos writeups.

</details>

<details>
<summary>📖 <strong>Writeups da semana</strong></summary>

Procure writeups de **BOLA, mass assignment, BFLA e GraphQL**. Atenção em como o autor **descobriu o endpoint** (recon de JS? versão legada? doc exposta?) e como provou a autorização quebrada.

</details>

### 🎯 Missão F3 — "Trinca de labs + teste real"
Feche **crAPI + VAmPI + DVGA** completos **e** faça pelo menos um teste de API em um alvo autorizado, partindo de recon de API automatizado.

**Critério de aceitação:** os três labs documentados (writeup/notas) + um teste real de API registrado (com ou sem bug — o processo conta), achando a superfície via JS/Swagger/kiterunner.

### ✅ Checklist F3
- [ ] OWASP API Top 10 (2023) sabida e testável item a item
- [ ] crAPI, VAmPI e DVGA fechados e documentados
- [ ] Faz introspection e abusa de batching/aliases em GraphQL
- [ ] Detecta mass assignment (BOPLA) e BOLA/BFLA em REST
- [ ] Mapeia superfície de API via JS + Swagger + kiterunner + IA
- [ ] Pelo menos 1 teste de API em alvo real registrado

---

⬅️ [Anterior: Fundação + Web (F0·F1·F2)](01-fundacao-e-web-f0-f2.md) · [⬆️ Topo](#) · [Próximo: CTF · CVE · Carreira (F4–F7) ➡️](03-ctf-cve-carreira-f4-f7.md)
