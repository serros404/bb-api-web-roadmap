# 🎓 Trilha PortSwigger — semana a semana (manual-first)

[🏠 Início](README.md) · espinha de estudo do [ROADMAP MINIMALISTA](ROADMAP-MINIMALISTA.md)

> Os **272 labs** da [Web Security Academy](https://portswigger.net/web-security/all-labs) fechados na **ordem que te deixa caçando o dinheiro (authz/lógica/DOM XSS) já no mês 1** — não na ordem da página. Contagem de labs conforme o site (2026); a fonte da verdade é sempre a Academy. Meta: **[BSCP](https://portswigger.net/web-security/certification)** por volta da semana ~18–20.

## Como rodar (3 regras)

1. **O peso manda, não o relógio.** Alvo: **~10–14 labs/semana**. Classe pesada (13+ labs — XSS `30`, smuggling `22`, SQLi `18`, cache poisoning `13`, Access control `13`) = **2 semanas ou mais**; leves (`3–6`) = **junta 2–3** numa semana. Semana com 15+ labs pode virar 1,5–2 semanas — tudo bem.
2. **Faça por _Topic_, não só por _Path_.** Suas classes-rainhas — **Access control, Business logic, DOM-based** — são *topics* (aba "[All topics](https://portswigger.net/web-security/all-topics)"), **não** "Learning Paths"; se você seguir só a aba Paths, pula exatamente o que paga. Priorize **Apprentice + Practitioner**; os labs **_Expert_ são bônus — a BSCP não exige.**
3. **Semana ativa (senão é só colecionar lab):** cada semana = labs fechados **+** 1–2 writeups da classe **+** procurar essa classe num **alvo real** autorizado. Lab → writeup → caça.

---

## 🅰️ Bloco A — Autorização, lógica & auth (a sua praia) · ~sem 1–6 · 💰

Fecha todo o seu ganha-pão **primeiro**. No fim deste bloco você já caça o que **paga** em alvo real — não precisa terminar o resto pra começar.

- [ ] **Sem 1** — [Essential skills](https://portswigger.net/web-security/essential-skills) `2` + [**Access control**](https://portswigger.net/web-security/access-control) `13` — 💰 IDOR/BOLA/BFLA, o nº 1. Autorize + 2 contas.
- [ ] **Sem 2** — [**Business logic**](https://portswigger.net/web-security/logic-flaws) `11` + [Information disclosure](https://portswigger.net/web-security/information-disclosure) `5` — 💰 quebrar as regras do fluxo + achar leaks.
- [ ] **Sem 3** — [**Authentication**](https://portswigger.net/web-security/authentication) `14` — 💰 os caminhos de ATO.
- [ ] **Sem 4** — [OAuth](https://portswigger.net/web-security/oauth) `6` + [JWT](https://portswigger.net/web-security/jwt) `8` — 💰 SSO/token → ATO.
- [ ] **Sem 5** — [**DOM-based**](https://portswigger.net/web-security/dom-based) `7` + [Race conditions](https://portswigger.net/web-security/race-conditions) `6` — 💰 DOM XSS (ler JS, source→sink) + TOCTOU.
- [ ] **Sem 6** — [Prototype pollution](https://portswigger.net/web-security/prototype-pollution) `10` — a classe-assinatura do JS (client + server).

## 🅱️ Bloco B — API & injeções de alto valor · ~sem 7–12

- [ ] **Sem 7** — [SSRF](https://portswigger.net/web-security/ssrf) `7` + [API testing](https://portswigger.net/web-security/api-testing) `5` — SSRF→cloud, recon de API + server-side param pollution.
- [ ] **Sem 8** — [GraphQL](https://portswigger.net/web-security/graphql) `5` + [NoSQL injection](https://portswigger.net/web-security/nosql-injection) `4` — API/GraphQL + NoSQL.
- [ ] **Sem 9–10** — [SQL injection](https://portswigger.net/web-security/sql-injection) `18` — base clássica (2 semanas).
- [ ] **Sem 11** — [Path traversal](https://portswigger.net/web-security/file-path-traversal) `6` + [File upload](https://portswigger.net/web-security/file-upload) `7` — arquivo → web shell/RCE.
- [ ] **Sem 12** — [OS command injection](https://portswigger.net/web-security/os-command-injection) `5` + [XXE](https://portswigger.net/web-security/xxe) `9` — injeção server-side.

## 🅲 Bloco C — XSS a fundo + client-side · ~sem 13–15

- [ ] **Sem 13–14** — [Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting) `30` — o resto do XSS além do DOM (reflected/stored, CSP bypass, polyglots). Grande — 2 (ou 3) semanas.
- [ ] **Sem 15** — [CSRF](https://portswigger.net/web-security/csrf) `12` + [CORS](https://portswigger.net/web-security/cors) `3` — client-side de estado/origem.

## 🅳 Bloco D — Advanced: parser, cache & cadeias (estilo Orange Tsai) · ~sem 16–19

- [ ] **Sem 16** — [SSTI](https://portswigger.net/web-security/server-side-template-injection) `7` + [Insecure deserialization](https://portswigger.net/web-security/deserialization) `10` — RCE server-side.
- [ ] **Sem 17** — [HTTP Host header](https://portswigger.net/web-security/host-header) `7` + [Web cache deception](https://portswigger.net/web-security/web-cache-deception) `5` — confusão de parser/roteamento ([lente Orange](03-playbooks/orange-tsai-decodificado-js.md)).
- [ ] **Sem 18–19** — [HTTP request smuggling](https://portswigger.net/web-security/request-smuggling) `22` + [Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) `13` — desync + cache (pesado, 2+ semanas).
- [ ] **Encaixe/leves** — [Clickjacking](https://portswigger.net/web-security/clickjacking) `5` + [WebSockets](https://portswigger.net/web-security/websockets) `3` · *(opcional)* [Web LLM attacks](https://portswigger.net/web-security/llm-attacks) `7`.

> Os **31 topics** da Academy estão todos aqui (server-side, client-side e advanced) — nenhum ficou de fora.

## 🏁 BSCP — ~semana 18–20

- [ ] Todos os labs **Apprentice + Practitioner** fechados → treina com o **[Mystery Lab Challenge](https://portswigger.net/web-security/certification)** (lab aleatório, sem contexto — igual à prova).
- [ ] [Practice exam](https://portswigger.net/web-security/certification) → **[Burp Suite Certified Practitioner (BSCP)](https://portswigger.net/web-security/certification)** (4h, 100% prático).

---

## Régua

- [ ] **Bloco A fechado** (Access Control + Business Logic + Auth + DOM XSS + PP à mão)
- [ ] Acha **IDOR/BOLA/logic num alvo real** (não só no lab)
- [ ] ≥1 **writeup por classe** do Bloco A no `ctf-writeups`/blog
- [ ] Resolve um **Mystery Lab** sem dica
- [ ] **BSCP** no bolso

> **Realismo:** são ~272 labs, então a trilha inteira é ~4–6 meses no seu ritmo. Mas o ponto é: **o dinheiro (Bloco A) você domina em ~6 semanas** e já leva pra caça. Ninguém fecha 100% antes de caçar — feche o Bloco A e **vá pro alvo real em paralelo** (é o [loop do minimalista](ROADMAP-MINIMALISTA.md)).

---

## 📊 Tracker — data / feito

> **Progresso:** `0 / 272` labs · `0 / 31` topics · BSCP: 🔲
> Atualize a linha ao mexer em cada topic. **Status:** 🔲 a fazer · 🔄 fazendo · ✅ feito · **Labs:** feitos/total · **Data:** quando fechou · **WU:** link do writeup.

### 🅰️ Bloco A — sua praia 💰
| Topic | Labs | Status | Data fim | WU | Notas |
|---|---|---|---|---|---|
| [Essential skills](https://portswigger.net/web-security/essential-skills) | 0/2 | 🔲 | | | |
| [Access control](https://portswigger.net/web-security/access-control) | 0/13 | 🔲 | | | |
| [Business logic](https://portswigger.net/web-security/logic-flaws) | 0/11 | 🔲 | | | |
| [Information disclosure](https://portswigger.net/web-security/information-disclosure) | 0/5 | 🔲 | | | |
| [Authentication](https://portswigger.net/web-security/authentication) | 0/14 | 🔲 | | | |
| [OAuth](https://portswigger.net/web-security/oauth) | 0/6 | 🔲 | | | |
| [JWT](https://portswigger.net/web-security/jwt) | 0/8 | 🔲 | | | |
| [DOM-based](https://portswigger.net/web-security/dom-based) | 0/7 | 🔲 | | | |
| [Race conditions](https://portswigger.net/web-security/race-conditions) | 0/6 | 🔲 | | | |
| [Prototype pollution](https://portswigger.net/web-security/prototype-pollution) | 0/10 | 🔲 | | | |

### 🅱️ Bloco B — API & injeções
| Topic | Labs | Status | Data fim | WU | Notas |
|---|---|---|---|---|---|
| [SSRF](https://portswigger.net/web-security/ssrf) | 0/7 | 🔲 | | | |
| [API testing](https://portswigger.net/web-security/api-testing) | 0/5 | 🔲 | | | |
| [GraphQL](https://portswigger.net/web-security/graphql) | 0/5 | 🔲 | | | |
| [NoSQL injection](https://portswigger.net/web-security/nosql-injection) | 0/4 | 🔲 | | | |
| [SQL injection](https://portswigger.net/web-security/sql-injection) | 0/18 | 🔲 | | | |
| [Path traversal](https://portswigger.net/web-security/file-path-traversal) | 0/6 | 🔲 | | | |
| [File upload](https://portswigger.net/web-security/file-upload) | 0/7 | 🔲 | | | |
| [OS command injection](https://portswigger.net/web-security/os-command-injection) | 0/5 | 🔲 | | | |
| [XXE injection](https://portswigger.net/web-security/xxe) | 0/9 | 🔲 | | | |

### 🅲 Bloco C — XSS & client-side
| Topic | Labs | Status | Data fim | WU | Notas |
|---|---|---|---|---|---|
| [Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting) | 0/30 | 🔲 | | | |
| [CSRF](https://portswigger.net/web-security/csrf) | 0/12 | 🔲 | | | |
| [CORS](https://portswigger.net/web-security/cors) | 0/3 | 🔲 | | | |

### 🅳 Bloco D — advanced / parser & cache
| Topic | Labs | Status | Data fim | WU | Notas |
|---|---|---|---|---|---|
| [SSTI](https://portswigger.net/web-security/server-side-template-injection) | 0/7 | 🔲 | | | |
| [Insecure deserialization](https://portswigger.net/web-security/deserialization) | 0/10 | 🔲 | | | |
| [HTTP Host header](https://portswigger.net/web-security/host-header) | 0/7 | 🔲 | | | |
| [Web cache deception](https://portswigger.net/web-security/web-cache-deception) | 0/5 | 🔲 | | | |
| [HTTP request smuggling](https://portswigger.net/web-security/request-smuggling) | 0/22 | 🔲 | | | |
| [Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) | 0/13 | 🔲 | | | |
| [Clickjacking](https://portswigger.net/web-security/clickjacking) | 0/5 | 🔲 | | | |
| [WebSockets](https://portswigger.net/web-security/websockets) | 0/3 | 🔲 | | | |
| [Web LLM attacks](https://portswigger.net/web-security/llm-attacks) | 0/7 | 🔲 | | | |

> Fechou tudo Apprentice+Practitioner? Marca o BSCP no cabeçalho e vai pro [Mystery Lab](https://portswigger.net/web-security/certification). 272 labs somam os 31 topics acima (2+13+11+5+14+6+8+7+6+10 · 7+5+5+4+18+6+7+5+9 · 30+12+3 · 7+10+7+5+22+13+5+3+7).

---

⬅️ [ROADMAP MINIMALISTA](ROADMAP-MINIMALISTA.md) · [🏠 Início](README.md)
