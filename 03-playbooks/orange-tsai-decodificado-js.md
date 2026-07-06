# 12 — Orange Tsai Decodificado: o Cânone Traduzido para o Ecossistema JavaScript

[🏠 Início](../README.md) › [03 · Playbooks](README.md) › Orange Tsai Decodificado (foco JS)

**Sumário:**
- [Por que Orange Tsai é a bússola deste roadmap](#por-que-orange-tsai-é-a-bússola-deste-roadmap)
- [1 — O DNA da pesquisa: mindset e filosofia](#1--o-dna-da-pesquisa-mindset-e-filosofia)
- [2 — O cânone traduzido para JavaScript](#2--o-cânone-traduzido-para-javascript)
  - [2.1 SSRF avançado → Node](#21-ssrf-avançado--node-undici-fetch-axios-e-a-inconsistência-parserrequester)
  - [2.2 Parser differentials → JS](#22-parser-differentials-breaking-parser-logic--js)
  - [2.3 HTTP Request Smuggling / desync → llhttp](#23-http-request-smuggling--desync--llhttp-e-o-node-por-trás-do-proxy)
  - [2.4 Path confusion / roteamento → Express/Fastify/Next](#24-path-confusion--normalização--roteamento--expressfastifynext)
  - [2.5 Ataques a reverse proxies e gateways → mundo Node/edge](#25-ataques-a-reverse-proxies-e-gateways--o-mundo-nodeedge)
  - [2.6 Desserialização e execução → node-serialize, vm2, prototype pollution](#26-desserialização-e-execução--node-serialize-vm2-prototype-pollution)
  - [2.7 Cadeias de nível arquitetural → stacks JS full-stack](#27-cadeias-de-nível-arquitetural--stacks-js-full-stack)
- [3 — Superfícies do ecossistema JS a dominar (por ROI)](#3--superfícies-do-ecossistema-js-a-dominar-por-roi)
- [4 — Panteão de pesquisadores de Web Security](#4--panteão-de-pesquisadores-de-web-security-seguir-ler-imitar)
- [5 — Fontes primárias](#5--fontes-primárias)
- [🧠 Síntese: a bússola em uma frase](#-síntese-a-bússola-em-uma-frase)

---

> **O que este documento é.** A metodologia do Orange Tsai (DEVCORE) destilada e **pivotada do mundo PHP/infra para o ecossistema JavaScript** (Node.js, runtimes JS, front-end moderno, npm). Não é um resumo de "ele achou o ProxyLogon" — é o **modo de pensar** dele extraído e reaplicado ao JS, com tradução técnica de cada peça do cânone. É a lente que reorganiza o resto do roadmap: [classes de bug](classes-de-bug.md), [técnicas avançadas e cadeias](tecnicas-avancadas-e-cadeias.md), [API/F3](../02-trilha/02-api-f3.md) e a caça real.
>
> Tudo para uso **autorizado** e com **PoC de impacto mínimo** — releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase). Os exemplos são para lab e validação em escopo.

> **Honestidade epistêmica.** As atribuições (talks, anos, CVEs, prêmios) foram conferidas em fontes primárias, listadas na [seção 5](#5--fontes-primárias). Onde a tradução para JS é **extrapolação minha** — e não algo que o próprio Orange publicou — o texto diz isso explicitamente ("a leitura Orange-Tsai disto seria…"). Não confunda o cânone verificado com a analogia.

---

## Por que Orange Tsai é a bússola deste roadmap

Orange Tsai (Cheng-Da Tsai, DEVCORE / time HITCON) é o arquétipo do **hunter + researcher**: não caça payload, caça **arquitetura**. O corpo de trabalho dele é uma coleção de aulas sobre o mesmo tema — **o que acontece quando dois componentes discordam sobre o significado da mesma entrada** — aplicado a alvos de altíssimo impacto (Exchange, Apache httpd, IIS, SSL VPNs, GitHub Enterprise). Ele ganhou o **Pwnie de Best Server-Side Bug (2021)** com o ProxyLogon e demonstrou o ProxyShell no **Pwn2Own 2021** (US$ 200k). O detalhe que interessa para você: **quase todo o cânone dele é sobre parsers, proxies e limites de sistema — e o ecossistema JavaScript de 2026 é um oceano de parsers, proxies e limites de sistema.**

A tese deste documento: **traduzir o mindset dele para o stack JS te coloca à frente de 90% dos caçadores**, que ainda pensam em bug isolado e payload de lista. Onde a maioria vê "o Express serve um arquivo", o pesquisador estilo Orange vê "o roteador do Express, o `path.normalize` do Node, o proxy nginx na frente e o CDN na frente do nginx — quatro normalizadores de path que **não concordam**, e a discordância é o bug".

---

## 1 — O DNA da pesquisa: mindset e filosofia

Sete traços, cada um com a tradução para como você deve operar no JS.

### 1.1 Ataca o middleware, não o app

Orange raramente ataca a lógica de negócio de um app. Ele ataca a **camada que fica embaixo de milhares de apps**: o servidor HTTP (Apache httpd — *Confusion Attacks*, 2024), o mail server (Exchange — Proxy\*), o gateway (SSL VPNs — 2019), a linguagem/runtime (URL parsers — 2017). Um bug ali não vale um alvo; vale **toda a frota** que roda aquela camada.

> **Tradução JS.** Suas "camadas embaixo de tudo" são: o **framework** (Express, Fastify, Koa, Next.js, NestJS), o **runtime** (Node/`llhttp`/`undici`, Bun, Deno, workerd/Cloudflare), o **proxy JS** (`http-proxy`, `node-http-proxy`, o middleware do Next.js, BFFs, edge functions) e as **libs onipresentes** (`qs`, `body-parser`, `lodash`, `axios`, `jsonwebtoken`, `dompurify`). Um bug no **roteamento do Next.js** ([CVE-2025-29927](#5--fontes-primárias)) não é um bug de um site — é um bug de **todo site self-hosted em Next.js vulnerável**. É por isso que um `npm audit` que revela um gadget de prototype pollution numa lib popular vale mais que dez IDORs.

### 1.2 Procura onde dois sistemas discordam da mesma entrada

Este é **o** padrão. Todo o cânone é variação disto:

| Talk / pesquisa | Os dois lados que discordam |
|---|---|
| A New Era of SSRF (2017) | o **parser** de URL vs o **requester** de URL leem `http://a@b/` diferente |
| Breaking Parser Logic (2018) | o **proxy** (nginx) normaliza o path de um jeito; o **backend** (framework), de outro |
| ProxyShell (2021) | o **front-end do Exchange** faz path confusion e roteia para o **back-end** errado |
| Confusion Attacks / Apache (2024) | **módulo A** e **módulo B** do httpd interpretam o mesmo campo (`r->filename`) diferente |
| WorstFit (2024/25) | a camada **Unicode** e a camada **ANSI "Best-Fit"** transformam o mesmo byte diferente |

> **Tradução JS.** Onde há dois parsers, há um diferencial. Mapa de discordâncias no stack JS (aprofundado na [seção 2.2](#22-parser-differentials-breaking-parser-logic--js)):
> - `JSON.parse` (estrito) vs `JSON5`/`body-parser` (tolerante) vs `qs` (arrays/objetos aninhados) — **chaves duplicadas** e **coerção de tipo** divergem.
> - `new URL()` (WHATWG) vs `url.parse()` (legacy) vs a regex de allowlist que "valida" o host — [SSRF](#21-ssrf-avançado--node-undici-fetch-axios-e-a-inconsistência-parserrequester).
> - `path.normalize`/`path.resolve` (POSIX vs Win32) vs o proxy na frente — [path confusion](#24-path-confusion--normalização--roteamento--expressfastifynext).
> - `llhttp` (Node) vs o parser do CDN/nginx — [request smuggling](#23-http-request-smuggling--desync--llhttp-e-o-node-por-trás-do-proxy).
>
> Pergunta-guia em todo alvo JS: **"quem mais lê esta string antes de mim, e nós concordamos?"**

### 1.3 Transforma bug "pequeno" em cadeia devastadora

O GitHub Enterprise dele (2017) foi **quatro** vulnerabilidades encadeadas de SSRF a RCE. ProxyLogon é SSRF (CVE-2021-26855) **+** escrita de arquivo (CVE-2021-27065). ProxyShell são **três** CVEs em fila. Um bug isolado é "informational"; a cadeia é "critical". O raciocínio: para cada achado, *"o que isto destrava se eu combinar com o próximo elo?"*.

> **Tradução JS.** As cadeias de maior valor em JS costumam ser: **prototype pollution** (fraca sozinha) → **gadget** num template/parser/`child_process` → **RCE/XSS/bypass**; ou **SSRF em Node** → **metadata de cloud** → **credencial** → pivot; ou **CSPT** → **CSRF** → ação privilegiada. Veja a [seção 2.7](#27-cadeias-de-nível-arquitetural--stacks-js-full-stack) e o [playbook de cadeias](tecnicas-avancadas-e-cadeias.md#a--cadeias-de-exploração-chaining-pensar-em-sistemas-não-em-bugs).

### 1.4 Lê o código-fonte e modela o comportamento real, não o documentado

Orange lê o source do httpd/Exchange e descobre o que o código **faz de fato** — não o que a doc promete. O bug mora exatamente na distância entre os dois.

> **Tradução JS — e aqui o JS te dá uma vantagem histórica:** o código do alvo **está na sua máquina**. `node_modules/` é o source do framework, das libs, do parser. Você pode ler `express/lib/router/`, `qs/lib/parse.js`, `undici/lib/`, o `llhttp` gerado. Aprenda a `grep` sinks em dependência (`child_process`, `vm`, `eval`, `Function(`, `[key]` em merges recursivos), a ler o `package-lock.json` para saber a versão **exata** que roda, e a diffar versões (o "patch gap": a correção saiu, o alvo não atualizou). Isto é a ponte direta para a frente de [CVE hunting (F5)](../02-trilha/03-ctf-cve-carreira-f4-f7.md#-f5--cve-hunting-lateral-code-review-oss--ia).

### 1.5 Pensa em superfície nova, não em vuln nova

"A New Era of SSRF" não é um SSRF — é uma **nova superfície de ataque** (o diferencial parser/requester) que gera SSRFs em série, em Python, PHP, Ruby, Java **e JavaScript**. Ele inventa a categoria; os outros passam anos catando instâncias dela.

> **Tradução JS.** Ambicione a categoria. "Prototype pollution" foi uma superfície nova (Arteau, 2018) que rendeu **centenas** de CVEs por anos. "Server-Side Prototype Pollution → RCE" (Silent Spring / Heyes, 2023) reabriu a superfície com gadgets universais. Quando você achar um padrão que se repete em N libs, você achou uma superfície — não um bug.

### 1.6 A estética do writeup "How I…"

Os títulos dele são narrativa: *"How I Chained 4 Vulnerabilities on GitHub Enterprise, From SSRF Execution Chain to RCE!"*. O writeup conta **hipótese → validação → escalonamento**, não um dump de payload. Isso é o que constrói reputação e vira referência.

> **Tradução para você.** Todo achado seu vira um "How I…": a pergunta que você fez, por que o sistema **deveria** ter resistido, onde ele discordou de si mesmo, como escalou. É o que separa o repo `ctf-writeups` de portfólio de um Notion morto. (Ver [F4/F5/F6](../02-trilha/03-ctf-cve-carreira-f4-f7.md).)

### 1.7 Paciência arquitetural + rigor de exploração

Orange passa **meses** num alvo, mapeando a arquitetura antes do primeiro payload sério. Mas quando dispara, a PoC é cirúrgica e reproduzível. Amplitude na compreensão, profundidade na prova — exatamente a tese [amplitude por máquina → profundidade por humano](../README.md#-a-metodologia-2026-amplitude-por-máquina-profundidade-por-humano) deste roadmap, levada ao nível de research.

---

## 2 — O cânone traduzido para JavaScript

Para cada tema central do Orange: **a técnica original (verificada)** e a **tradução para o stack JS**. Cada bloco fecha com onde treinar (CTF) e onde caçar (bounty) — o loop [CTF ↔ bug bounty](../02-trilha/03-ctf-cve-carreira-f4-f7.md#-f4--ctf--labs-a-serviço-do-bug-bounty-velocidade-cadeias-e-escalonamento) em miniatura.

---

### 2.1 SSRF avançado → Node (`undici`, `fetch`, `axios`) e a inconsistência parser/requester

**Original.** *A New Era of SSRF – Exploiting URL Parser in Trending Programming Languages* (Black Hat USA 2017). Tese: o componente que **valida** a URL e o componente que **busca** a URL usam parsers diferentes. Um lê `http://127.0.0.1\tfoo` / `http://a@127.0.0.1@b/` / URLs com fragmentos, `\`, unicode, e chega a um host; o outro chega a **outro** host. Resultado: allowlists baseadas em string caem. Ele fuzzou os parsers built-in de várias linguagens (**JavaScript incluído**) e explorou WordPress, vBulletin, MyBB e GitHub Enterprise.

**Tradução JS.** O diferencial parser/requester é **nativo** do Node moderno, porque coexistem parsers de URL diferentes e libs de fetch diferentes:

- **Dois parsers de URL no mesmo runtime.** `new URL()` (WHATWG, o padrão hoje) e o legado `url.parse()` divergem em casos de borda (backslashes, `@` múltiplos, espaços, caracteres de controle). Uma allowlist escrita com um e um `fetch` que usa o outro = SSRF.
- **`fetch`/`undici`, `node-fetch`, `axios`, `request`** têm comportamentos distintos de **redirect** e de resolução. Se você valida o host **antes** e a lib segue um **302** para `169.254.169.254` **depois**, a validação foi inútil (TOCTOU de rede). O ecossistema `follow-redirects` já teve várias falhas dessa família (vazamento de header sensível/`Authorization` em redirect cross-host, parsing impróprio de URL).
- **Allowlist por string** (`url.startsWith("https://api.")`, `host.endsWith(".alvo.com")`, `host.includes("alvo.com")`) é o pão-com-manteiga do SSRF em JS. Bypasses: `https://api.alvo.com.evil.com`, `https://evil.com/api.alvo.com`, `https://alvo.com@evil.com`, IP decimal/hex/IPv6, `[::]`, `0.0.0.0`.
- **DNS rebinding**: quando o código resolve o host **duas vezes** (uma pra validar, uma pra buscar), um domínio seu responde IP público na 1ª e interno na 2ª. Node com cache de DNS curto/inexistente é alvo clássico.

```js
// Padrão vulnerável realista (webhook / "importar de URL")
const u = new URL(req.body.url);                 // parser A (WHATWG)
if (!u.hostname.endsWith('.alvo.com')) throw 403; // allowlist por string
const r = await fetch(req.body.url, { redirect: 'follow' }); // requester + redirect
// Bypass 1: https://x.alvo.com.evil.com  (endsWith falha? não — .alvo.com.evil.com NÃO termina em .alvo.com)
// Bypass 2 (o bom): host .alvo.com que você controla e aponta p/ 169.254.169.254 (rebinding / registro DNS)
// Bypass 3: valida req.body.url mas passa a MESMA string pro fetch, que segue 302 -> interno
```

- **Escalada** idêntica ao [playbook SSRF](classes-de-bug.md#a2-server-side-request-forgery-ssrf): metadata AWS/GCP/Azure (IMDSv1), `gopher://`→Redis interno (em Node, muitas vezes o BullMQ/cache), serviços internos (`http://localhost:9200` Elastic, `:5984` Couch). Em serverless/edge, o IMDS some, mas surgem **metadata endpoints do provider** e **variáveis de ambiente com segredos** — SSRF vira leak de env.

**CTF:** categorias *web* com "fetch this URL", desafios de SSRF do [PortSwigger Academy](https://portswigger.net/web-security/ssrf), rounds de SSRF/URL-bypass em NahamCon/Intigriti. **Bounty:** qualquer feature "importar de URL", geração de PDF/thumbnail, webhooks, integrações, previews de link, avatar-by-URL — abundantes em SaaS Node.

---

### 2.2 Parser differentials (*Breaking Parser Logic*) → JS

**Original.** *Breaking Parser Logic! Take Your Path Normalization Off and Pop 0days Out* (Black Hat USA 2018 / DEF CON 26). Ele atacou **normalização de path** entre proxy e app e achou 0days em Java Spring, Ruby on Rails, Python aiohttp — e **Next.js**. E, mais amplamente, *Confusion Attacks* (Apache httpd, 2024): três tipos de confusão (**filename**, **documentRoot**, **handler**) porque módulos diferentes do httpd leem o mesmo campo de formas diferentes; corrigido no 2.4.60. *WorstFit* (2024/25, com splitline) é a mesma ideia em outra camada: a conversão **Best-Fit** de charset do Windows transforma `＜`(U+FF1C) em `<`, `∕` em `/`, etc., **depois** da validação → path traversal, argument injection, RCE.

**Tradução JS.** O stack JS é um empilhamento de parsers que quase nunca concordam. Diferenciais que você deve saber gerar e testar:

- **Query string: `querystring` vs `qs`.** O Express usa `qs` por padrão, que interpreta `a[b]=c` como **objeto aninhado** e `a=1&a=2` como **array**. O `querystring` nativo, não. Um filtro/WAF que lê a query com um e o app que lê com o outro divergem. É também a porta de entrada para **prototype pollution via query** (`?__proto__[x]=y`, `?constructor[prototype][x]=y`).
- **Chave duplicada / HTTP Parameter Pollution.** `?role=user&role=admin` — qual vence? Depende do parser. Em Node, `qs` faz array; um código que faz `params.role === 'user'` vs outro que pega `params.role[1]` divergem. Combine com um proxy que reescreve params.
- **JSON: `JSON.parse` vs `body-parser` vs `JSON5`/`json-bigint`.** Chaves duplicadas em JSON (`{"a":1,"a":2}`) — `JSON.parse` fica com a última; alguns parsers de streaming ou libs divergem. Números grandes, `__proto__` como chave, coerção — cada lib decide diferente.
- **Coerção de tipo (o "type juggling" do JS).** `req.query.x` pode ser string **ou array** conforme o input. `if (token != expected)` com `token` sendo array/objeto/`undefined` produz comparações inesperadas. `express`/`mongo` + objeto no lugar de string = **NoSQL injection** (`{"$gt":""}`) e bypass de auth.
- **Path/normalização (o *Breaking Parser Logic* direto):** `%2e%2e%2f`, `..%2f`, `....//`, `%252e`, encoded slash, unicode — o proxy (nginx/CDN) normaliza uma vez, o Node (`path.normalize`, `express.static`, `send`) normaliza de outro jeito, e a rota efetiva muda. Ver [2.4](#24-path-confusion--normalização--roteamento--expressfastifynext).
- **A leitura Orange-Tsai disto:** *Confusion Attacks* em JS seria procurar **um campo lido por dois pontos do código** — ex.: `req.path` usado no middleware de auth **e** re-derivado no handler; `Host`/`X-Forwarded-Host` usado no cache **e** na geração de link; `Content-Type` usado no parser **e** no dispatcher. Onde o mesmo campo é interpretado duas vezes, force a discordância.

```js
// Diferencial de tipo -> bypass de checagem (padrão comum em Express)
// rota: GET /reset?token=abc
if (req.query.token !== savedToken) return res.sendStatus(403);
// ataque: /reset?token[]=abc  ->  req.query.token === ['abc'] (array) !== 'abc'(string)?  sim, difere...
// ...mas em outros pontos o código faz String(req.query.token) === savedToken -> 'abc' === 'abc' -> passa
// A discordância entre os dois usos do MESMO parâmetro é o bug.
```

**CTF:** desafios de HPP, "type juggling" em JS, prototype-pollution-via-query, e normalização de path aparecem muito em CTFs web modernos (Intigriti, HTB, corfmy). **Bounty:** WAF/allowlist na frente de app Node; qualquer lugar onde há **proxy + Node** (quase todo SaaS); features que reconstroem paths/hosts.

---

### 2.3 HTTP Request Smuggling / desync → `llhttp` e o Node por trás do proxy

**Original.** Aqui o cânone-referência é do **James Kettle** (PortSwigger), não do Orange — mas é a **mesma família** ("dois sistemas discordam de onde termina a request"): *HTTP Desync Attacks* e *HTTP/2 smuggling*. O Orange contribui com o ângulo arquitetural (front-end vs back-end do Exchange). Junte os dois: front (CDN/nginx/HAProxy) e back (Node) discordam do fim da mensagem.

**Tradução JS.** O parser HTTP do Node é o **`llhttp`**. Pontos de atrito conhecidos que geram desync quando há um proxy tolerante na frente:

- **CL.TE / TE.CL / TE.TE**: o front respeita `Content-Length`, o Node respeita `Transfer-Encoding: chunked` (ou vice-versa). Historicamente o Node/`llhttp` teve tolerâncias (aceitar espaços/tabs antes de `:`, variações de `chunked`, `\n` sem `\r`) que, combinadas com um front rígido/tolerante diferente, produzem smuggling.
- **HTTP/2 → HTTP/1 downgrade**: se o CDN fala HTTP/2 com o cliente e HTTP/1 com o Node, campos que o HTTP/2 permite mas o HTTP/1 não (ex.: `\r\n` embutido em header) viram injeção de request na tradução. H2.CL / H2.TE.
- **Frameworks e streaming**: Express/Koa/Fastify entregam o corpo via streams; comportamentos de keep-alive, pipelining e request reuse por conexão podem amplificar o efeito (uma request "contrabandeada" prefixa a da próxima vítima na mesma conexão de back-end).

> **Postura.** Smuggling é a técnica **mais delicada** para escopo — a PoC pode afetar outros usuários. Confirme com **timing** e a extensão **HTTP Request Smuggler** (Kettle) no [Burp](classes-de-bug.md#a7-http-request-smuggling); **nunca** rode payload de smuggling "de verdade" contra produção sem entender exatamente o blast radius. Domine em lab primeiro (Academy).

**CTF:** os labs de *request smuggling* do PortSwigger; desafios de desync em CTFs top-tier (raros, alto valor). **Bounty:** qualquer arquitetura **CDN/reverse-proxy + origem Node** — ou seja, a maioria. Sinais: respostas trocadas entre usuários, timing anômalo.

---

### 2.4 Path confusion / normalização / roteamento → Express/Fastify/Next

**Original.** *Breaking Parser Logic* (2018) e a **path confusion do ProxyShell** (CVE-2021-34473): o front-end do Exchange interpreta um path com um segmento "mágico" (`/autodiscover/autodiscover.json?@evil.com/...`) e roteia para um back-end que **acha que a autenticação já foi feita**. Access-control bypass por discordância de path.

**Tradução JS.** O roteamento é onde o JS mais sangra hoje:

- **`express.static` / `send` / `serve-static`**: path traversal quando o app concatena input no path de arquivo sem normalizar, ou quando o encoded-slash passa pelo proxy mas é decodificado pelo Node. `GET /static/..%2f..%2f.env`. Ler o source de `send` para saber exatamente como ele resolve.
- **Confusão de rota em Express**: ordem de middleware, `app.use('/admin', auth)` vs rotas montadas depois, regex de rota gulosa, `:param` que captura `/` com encoding. A auth cobre `/admin` mas não `/admin/` ou `/Admin` ou `/admin%2f`?
- **Next.js — o alvo JS mais "Orange-Tsai-shaped" de 2025.** [CVE-2025-29927](#5--fontes-primárias) (CVSS 9.1): o header **interno** `x-middleware-subrequest` era confiado pelo runtime para marcar "esta é uma subrequest interna, pule o middleware". Bastava **enviar esse header** para **pular todo o middleware** — incluindo auth/authz. É o padrão puro do Orange: **um campo interno de confiança que o mundo externo consegue forjar**, porque middleware e handler discordam sobre "quem já autenticou". Some a isso path confusion entre o roteador do App Router/Pages Router e o proxy.
- **Fastify/NestJS/Koa**: cada um com sua normalização e sua ordem de hooks/guards. Guards que rodam por decorator vs rotas registradas dinamicamente = janelas.

```http
GET /admin/dashboard HTTP/1.1
Host: alvo
x-middleware-subrequest: middleware        # CVE-2025-29927 (Next.js vulnerável): pula o middleware de auth
```

> **A pergunta Orange-Tsai aqui:** *"o componente que decide o acesso (proxy/middleware) e o componente que serve o recurso (handler/static) concordam sobre qual é o path/rota efetivo?"* Sempre que a resposta for "não sei", há bug.

**CTF:** path traversal, "auth bypass by path", e roteamento aparecem em quase todo CTF web. **Bounty:** SPAs Next.js self-hosted, BFFs, qualquer `express.static` mal-configurado, admin atrás de middleware.

---

### 2.5 Ataques a reverse proxies e gateways → o mundo Node/edge

**Original.** SSL VPNs (2019), Exchange front/back (2021), httpd (2024) — Orange ama a **caixa no meio**: o gateway que fala com o mundo e delega para trás. É onde confiança é assumida ("se chegou aqui, já passou pela borda").

**Tradução JS.** O "meio" agora é frequentemente **JavaScript**:

- **`http-proxy` / `node-http-proxy` / `express-http-proxy`**: reescrita de path/host, headers injetáveis, `X-Forwarded-*` confiados cegamente, SSRF via proxy mal-configurado.
- **Next.js middleware, Nuxt, SvelteKit hooks, Remix**: a borda que roda **antes** do handler — e que, como o CVE-2025-29927 mostrou, é uma superfície de bypass de primeira grandeza.
- **Edge/serverless (Cloudflare Workers, Vercel Edge, Lambda@Edge, Deno Deploy)**: runtimes JS com um subconjunto de APIs, comportamentos próprios de header/cache, e o clássico "a função de borda confia num header que o cliente controla".
- **BFF (Backend-for-Frontend)**: o padrão onde um Node fica entre o browser e as APIs reais, guardando tokens em cookie de sessão. Confusão de rota/host no BFF = acesso às APIs de trás.
- **Header trust**: `X-Forwarded-Host` / `X-Forwarded-For` / `X-Real-IP` confiados para gerar links (reset de senha), decidir cache-key, ou rate-limit. É [host header/2ª ordem](tecnicas-avancadas-e-cadeias.md#g--bugs-de-segunda-ordem-e-contexto-secundário) e web cache poisoning.

**CTF:** desafios com "há um proxy na frente", SSRF-via-proxy, header smuggling. **Bounty:** arquiteturas de microsserviços com gateway Node, edge functions, qualquer `X-Forwarded-*` refletido.

---

### 2.6 Desserialização e execução → node-serialize, vm2, prototype pollution

**Original.** Exchange e outros caem por **desserialização insegura** e primitivas de execução. A ideia: dado controlado pelo atacante vira **objeto/código** do lado do servidor.

**Tradução JS.** O JS tem suas próprias primitivas de "dado vira código", e a **central é prototype pollution** — a superfície mais rentável do ecossistema:

- **`node-serialize` (CVE-2017-5941)**: o `unserialize()` reconstrói funções marcadas com `_$$ND_FUNC$$_`; um **IIFE** embutido executa na desserialização → RCE. `funcster` tem risco análogo. Se você vê blob serializado que vira objeto com métodos, investigue.
- **`vm`/`vm2` sandbox escape**: `vm` do Node **não é** sandbox de segurança (a doc avisa). O `vm2`, que se propunha a ser, foi **descontinuado em julho/2023** após escapes irrecuperáveis (**CVE-2023-37466** e **CVE-2023-37903**, CVSS 9.8, escape → RCE). Qualquer app que roda "código do usuário" em `vm2` é RCE esperando. Migração recomendada: `isolated-vm`. Procure `vm2` em `package-lock.json` de alvos que executam expressões/fórmulas/plugins.
- **Prototype pollution — o coração do ecossistema.** Poluir `Object.prototype` via `__proto__`/`constructor.prototype` em merges recursivos, parsing de query/JSON, `_.merge`/`_.set` antigos, `Object.assign` profundo. Sozinha é DoS ou "esquisitice". **Com gadget**, vira:
  - **Client-side** → **DOM XSS** (poluir uma prop que um sink lê: `sanitizer` config, `srcdoc`, template).
  - **Server-side (SSPP)** → **RCE**. A pesquisa *Silent Spring* (USENIX Security 2023 / DEF CON 31, KTH) achou **11 gadgets universais** em APIs core do Node (ex.: poluir opções que acabam em `child_process.spawn`/`require`), com RCE em NPM CLI, Parse Server, Rocket.Chat. Gareth Heyes (PortSwigger) mostrou **detecção black-box sem DoS**.
  - **Bypass/EoP**: poluir `isAdmin`, `role`, flags de config que outro código lê como default.

```js
// Prototype pollution: da poluição ao gadget
// 1) Poluição (via merge recursivo vulnerável, query, ou JSON body):
//    {"__proto__":{"polluted":"x"}}   ou   ?__proto__[polluted]=x
// 2) Confirmação:
({}).polluted            // "x"  => Object.prototype foi poluído
// 3) Gadget server-side (conceito Silent Spring): poluir uma opção default que
//    chega a um sink (spawn/require/template) e vira execução. SEMPRE em lab.
```

> **Detecção.** Client-side: **DOM Invader** (Burp) detecta PP automaticamente. Server-side: técnica black-box do Heyes (poluir uma prop que muda o comportamento observável da resposta — ex.: status, JSON, header — sem derrubar o serviço). Ver [playbook B2](classes-de-bug.md#b2-prototype-pollution).

**CTF:** prototype pollution é **onipresente** em CTF web JS (é quase um gênero próprio); DVGA e labs de PP; desafios de `vm2` escape aparecem em CTFs mais duros. **Bounty:** merges de config/JSON, libs desatualizadas (lodash < 4.17.12, etc.), qualquer endpoint que faz deep-merge do body; `vm2` em features de "fórmula/expressão/sandbox".

---

### 2.7 Cadeias de nível arquitetural → stacks JS full-stack

**Original.** ProxyLogon (SSRF → file write → RCE) e ProxyShell (path confusion → EoP → file write → RCE) são o gold standard de **cadeia arquitetural**: cada elo é um bug de componente diferente, e a soma é pre-auth RCE na frota inteira.

**Tradução JS — modelos de cadeia para montar em stack MERN/Next/Nest:**

| Cadeia | Elos (cada um num componente diferente) | Resultado |
|---|---|---|
| **SSRF → IMDS → IAM** | SSRF em `fetch`/webhook → `169.254.169.254` → credencial de role → API de cloud | comprometimento de conta cloud |
| **PP → SSTI/gadget → RCE** | prototype pollution no body → gadget num template (Pug/EJS) ou em `child_process` opts → RCE | RCE server-side |
| **CSPT → CSRF → ação privilegiada** | client-side path traversal no fetch do front → endpoint sem proteção CSRF → mudança de estado | ATO/ação como vítima |
| **Middleware bypass → BOLA** | CVE-2025-29927 (ou rota mal-coberta) → endpoint de admin/API sem authz redundante | acesso privilegiado |
| **DOM clobbering → sanitizer bypass → XSS** | HTML "limpo" clobbera var global → enfraquece config do DOMPurify → mXSS | XSS em app "protegido" |
| **JWT alg confusion → forja de role** | RS256→HS256 com a chave pública como segredo → `role:admin` | privilege escalation |

O método é sempre o do [playbook de cadeias](tecnicas-avancadas-e-cadeias.md#a--cadeias-de-exploração-chaining-pensar-em-sistemas-não-em-bugs): inventarie **todo** achado fraco, anote *o que dá / o que falta*, procure o elo que conecta dois deles. A diferença do pensador arquitetural é olhar os elos **através das fronteiras de componente** (browser ↔ BFF ↔ API ↔ worker ↔ cloud), porque é lá que ninguém validou nada.

---

## 3 — Superfícies do ecossistema JS a dominar (por ROI)

Ordenado do **maior retorno agora** ao **mais nicho/fronteira**. "Domine de cima para baixo."

| # | Superfície | Por que o ROI é alto | Onde aprofundar |
|---|---|---|---|
| 1 | **Prototype pollution** (client + **SSPP**) + gadgets | Onipresente, subexplorada em server-side, escala a RCE/XSS; é *a* assinatura JS | [B2](classes-de-bug.md#b2-prototype-pollution) · Silent Spring · Heyes |
| 2 | **Broken authorization / lógica** (BOLA/BFLA, mass assignment) | "Onde está o dinheiro"; APIs JS expõem toneladas; scanner não pega | [Parte C](classes-de-bug.md#parte-c--autorização-autenticação-e-lógica-onde-está-o-dinheiro) · [F3](../02-trilha/02-api-f3.md) |
| 3 | **SSRF em Node** (parser/requester, rebinding, redirect) | Feature-rica em SaaS Node; escala a cloud | [2.1](#21-ssrf-avançado--node-undici-fetch-axios-e-a-inconsistência-parserrequester) · [A2](classes-de-bug.md#a2-server-side-request-forgery-ssrf) |
| 4 | **XSS moderno**: DOM clobbering, mXSS, `postMessage`, CSP bypass, sanitizer bypass (**DOMPurify**), gadgets de framework | Front-end JS é enorme; `dangerouslySetInnerHTML`/`v-html`/`bypassSecurityTrust*` reabrem a porta | [B1](classes-de-bug.md#b1-cross-site-scripting-xss--reflected-stored-dom)/[B3](classes-de-bug.md#b3-postmessage-vulnerabilities)/[B5](classes-de-bug.md#b5-dom-clobbering) · Heyes/Bentkowski/Mizu |
| 5 | **API security**: GraphQL (introspection, batching/aliasing, injection, CSRF), JWT em stacks JS, IDOR, mass assignment | Maior superfície moderna; ver [F3](../02-trilha/02-api-f3.md) | [F3](../02-trilha/02-api-f3.md) · [C6](classes-de-bug.md#c6-jwt-attacks) |
| 6 | **SSTI / RCE em template engines JS**: Pug, EJS, Handlebars, Nunjucks | Menos comum que em Python, mas RCE direto quando existe; ótimo gadget de PP | [A5](classes-de-bug.md#a5-server-side-template-injection-ssti) |
| 7 | **Parser differentials / path & route confusion** (Express/Next/Fastify) | A veia Orange-Tsai pura; auth bypass e cache | [2.2](#22-parser-differentials-breaking-parser-logic--js)/[2.4](#24-path-confusion--normalização--roteamento--expressfastifynext) |
| 8 | **Desserialização & sandbox**: `node-serialize`, `funcster`, **`vm2`** escape | RCE direto quando presente; procure em lockfiles | [2.6](#26-desserialização-e-execução--node-serialize-vm2-prototype-pollution) · [A6](classes-de-bug.md#a6-insecure-deserialization) |
| 9 | **Supply chain / npm**: dependency confusion, typosquatting, install scripts, análise de `package.json`/lockfile, postinstall | Impacto sistêmico; cruza com [F5/CVE](../02-trilha/03-ctf-cve-carreira-f4-f7.md#-f5--cve-hunting-lateral-code-review-oss--ia) | Snyk/Socket · seção 4 |
| 10 | **HTTP request smuggling** (`llhttp` + proxy) | Alto impacto, delicado p/ escopo; domine em lab | [2.3](#23-http-request-smuggling--desync--llhttp-e-o-node-por-trás-do-proxy) · [A7](classes-de-bug.md#a7-http-request-smuggling) |
| 11 | **Node internals**: event loop, streams, `Buffer`, `child_process` | Base para entender sinks e DoS; não é caça direta, é fundação | [F1](../02-trilha/01-fundacao-e-web-f0-f2.md#-f1--fundação-técnica) |
| 12 | **V8 / runtimes** (Bun, Deno, workerd), memory bugs | Fronteira de research; **baixo ROI para bounty**, alto para fama/CVE de plataforma | fronteira — só depois de dominar 1–10 |

> **Regra de priorização honesta.** Para **bounty** hoje: 1–5 pagam as contas. 6–10 são o diferencial de quem fecha crítico. 11–12 são **research/fama**, não renda de curto prazo — vá quando quiser virar "o pesquisador que…", não antes. É o mesmo espírito do Orange: ele foi para o V8/runtime-level **depois** de anos dominando a camada web.

---

## 4 — Panteão de pesquisadores de Web Security (seguir, ler, imitar)

Pesquisadores com **pesquisa pública** relevante para o stack web/JS. Estude o *método*, não só o resultado — leia os writeups perguntando "que pergunta ele fez que eu não faria?". Handles mudam; na dúvida, procure pelo nome + a organização. (Lista curada e verificável; não é ranking.)

| Nome | Pesquisas / foco | Onde ler / seguir |
|---|---|---|
| **Orange Tsai** (Cheng-Da Tsai) | SSRF/URL parser, path & parser confusion, proxies, RCE chains (Proxy\*, Apache, WorstFit) | [blog.orange.tw](https://blog.orange.tw) · [DEVCORE](https://devco.re/blog/) · X: `@orange_8361` |
| **James Kettle** (albinowax) | HTTP desync/request smuggling, web cache poisoning, HTTP/2, "state machine" | [PortSwigger Research](https://portswigger.net/research) · X: `@albinowax` |
| **Gareth Heyes** | XSS, **prototype pollution (client + server)**, mXSS, CSP, DOM | [PortSwigger Research](https://portswigger.net/research) · X: `@garethheyes` |
| **Michał Bentkowski** | **mXSS**, sanitizer/DOMPurify bypass, prototype pollution, parser quirks | [Securitum research](https://research.securitum.com) · X: `@SecurityMB` |
| **Masato Kinugawa** | XSS/mXSS, sanitizer bypass, Electron, encoding | X: `@kinugawamasato` · writeups próprios |
| **Kévin Mizu** | client-side, **DOMPurify/mXSS**, sanitizers, framework gadgets | X: `@kevin_mizu` · blog próprio |
| **Sam Curry** | API/auth/logic em alvos gigantes (automotivo, web3), mass-scale | [samcurry.net](https://samcurry.net) · X: `@samwcyo` |
| **Frans Rosén** | `postMessage`, OAuth, cloud, SSRF, DOM | [Detectify Labs](https://labs.detectify.com) · X: `@fransrosen` |
| **Shubham Shah / Adam Kues (Assetnote)** | appliance/edge 0day, Node/GraphQL, mass exploitation, patch-diff | [Assetnote / Searchlight blog](https://slcyber.io/assetnote-security-research-center/) · X: `@infosec_au` |
| **Doyensec** (Luca Carettoni & time) | Electron, **CSPT**, GraphQL, appsec profunda | [blog.doyensec.com](https://blog.doyensec.com) |
| **Liran Tal / Snyk Security Labs** | Node/npm **supply chain**, prototype pollution, segurança de dependências | [snyk.io/blog](https://snyk.io/blog) · X: `@liran_tal` |
| **Mikhail Shcherbakov** | **Server-Side Prototype Pollution → RCE** (Silent Spring), análise estática | papers USENIX · [KTH-LangSec](https://github.com/KTH-LangSec) · X: `@yu5k3` |
| **Olivier Arteau** | **origem** da prototype pollution em Node (NorthSec 2018) | [prototype-pollution-nsec18](https://github.com/HoLyVieR/prototype-pollution-nsec18) |
| **Johan Carlsson** (joaxcar) | CSPT, client-side, cache, GitLab | X: `@joaxcar` · blog próprio |
| **William Bowling** (vakzz) | file-read → RCE chains, GitLab, cadeias criativas | X: `@wcbowling` · [blog](https://devcraft.io) |
| **Justin Gardner** (Rhynorater) | GraphQL, CSPT, metodologia; **podcast** de referência | *Critical Thinking – Bug Bounty Podcast* · X: `@rhynorater` |
| **Inti De Ceukelaire** | lógica de negócio, OAuth, criatividade | X: `@intidc` · [intigriti blog](https://blog.intigriti.com) |
| **PortSwigger Research (coletivo)** | meta-fonte: **"Top 10 Web Hacking Techniques"** (anual) — o índice do estado da arte | [portswigger.net/research](https://portswigger.net/research) |

> **Como usar esta tabela.** Escolha **2–3 nomes por trimestre**, leia o **corpo de trabalho** deles (não um post), e reproduza **um** achado em lab. É o [Pilar 1 — leitura ativa de writeups](../README.md#pilar-1--leitura-ativa-de-writeups-35-por-semana-inegociável) com curadoria de elite. A meta-fonte "Top 10 Web Hacking Techniques" da PortSwigger é o melhor único ponto de partida para descobrir o que importou em cada ano.

---

## 5 — Fontes primárias

Cânone do Orange Tsai (conferido). Prefira slides + vídeo + o writeup no blog dele.

- **A New Era of SSRF — Exploiting URL Parser in Trending Programming Languages** — Black Hat USA 2017 (e HITB GSEC 2017). [Slides (PDF, blackhat.com)](https://blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf) · writeup relacionado: [How I Chained 4 Vulnerabilities on GitHub Enterprise (SSRF→RCE)](https://blog.orange.tw/posts/2017-07-how-i-chained-4-vulnerabilities-on/).
- **Breaking Parser Logic! Take Your Path Normalization Off and Pop 0days Out** — Black Hat USA 2018 / DEF CON 26. [Slides (PDF)](https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf).
- **Infiltrating Corporate Intranet Like NSA — Pre-auth RCE on Leading SSL VPNs** (com Meh Chang) — Black Hat USA 2019 / DEF CON 27. [Slides (PDF)](https://i.blackhat.com/USA-19/Wednesday/us-19-Tsai-Infiltrating-Corporate-Intranet-Like-NSA.pdf).
- **ProxyLogon** (CVE-2021-26855 SSRF + CVE-2021-27065 file write → RCE; também -26857/-26858). [Blog: A New Attack Surface on MS Exchange — Part 1: ProxyLogon](https://blog.orange.tw/2021/08/proxylogon-a-new-attack-surface-on-ms-exchange-part-1.html).
- **ProxyShell** (CVE-2021-34473 path confusion → ACL bypass + CVE-2021-34523 EoP + CVE-2021-31207 file write → RCE). Pwn2Own 2021; Pwnie **Best Server-Side Bug 2021**. [Blog Part 3: ProxyShell](https://blog.orange.tw/posts/2021-08-proxyshell-a-new-attack-surface-on-ms-exchange-part-3/) · [Slides "ProxyLogon is Just the Tip of the Iceberg" (BH USA 2021)](https://i.blackhat.com/USA21/Wednesday-Handouts/us-21-ProxyLogon-Is-Just-The-Tip-Of-The-Iceberg-A-New-Attack-Surface-On-Microsoft-Exchange-Server.pdf).
- **Confusion Attacks — Exploiting Hidden Semantic Ambiguity in Apache HTTP Server** — Black Hat USA 2024 / DEF CON 32; corrigido no httpd 2.4.60. [Blog](https://blog.orange.tw/posts/2024-08-confusion-attacks-en/) · [Slides (PDF)](https://i.blackhat.com/BH-US-24/Presentations/US24-Orange-Confusion-Attacks-Exploiting-Hidden-Semantic-Thursday.pdf).
- **WorstFit — Unveiling Hidden Transformers in Windows ANSI** (com splitline) — Black Hat Europe 2024 (publicado jan/2025). [Blog](https://blog.orange.tw/posts/2025-01-worstfit-unveiling-hidden-transformers-in-windows-ansi/) · [DEVCORE](https://devco.re/blog/2025/01/09/worstfit-unveiling-hidden-transformers-in-windows-ansi/) · [worst.fit](https://worst.fit).
- **Índice de talks do próprio Orange:** [blog.orange.tw/talks](https://blog.orange.tw/talks/).

Fontes do ecossistema JS citadas:

- **Prototype pollution (origem):** Olivier Arteau, *Prototype pollution attacks in NodeJS applications*, NorthSec 2018 — [repo/paper](https://github.com/HoLyVieR/prototype-pollution-nsec18).
- **SSPP → RCE:** *Silent Spring: Prototype Pollution Leads to RCE in Node.js*, USENIX Security 2023 / DEF CON 31 — [paper](https://arxiv.org/abs/2207.11171) · [gadgets](https://github.com/KTH-LangSec/server-side-prototype-pollution). Detecção black-box: [Gareth Heyes / PortSwigger](https://portswigger.net/web-security/prototype-pollution/server-side).
- **vm2** descontinuado (CVE-2023-37466, CVE-2023-37903) — [advisory](https://github.com/advisories/GHSA-cchq-frgv-rjh5). **node-serialize** RCE: CVE-2017-5941.
- **Next.js middleware bypass** CVE-2025-29927 (`x-middleware-subrequest`) — [análise (Datadog)](https://securitylabs.datadoghq.com/articles/nextjs-middleware-auth-bypass/) · [postmortem (Vercel)](https://vercel.com/blog/postmortem-on-next-js-middleware-bypass).
- **DOMPurify mXSS:** [Bypassing DOMPurify again with mutation XSS (PortSwigger)](https://portswigger.net/research/bypassing-dompurify-again-with-mutation-xss); CVE-2024-45801, CVE-2025-26791.

---

## 🧠 Síntese: a bússola em uma frase

> **Orange Tsai não acha bugs — ele acha lugares onde dois sistemas discordam sobre a mesma entrada, e transforma a discordância em cadeia.** No ecossistema JavaScript, esses lugares são abundantes e mal-vigiados: `new URL()` vs `fetch`, `qs` vs `querystring`, proxy vs `express.static`, middleware vs handler (`x-middleware-subrequest`), `Object.prototype` vs todo o resto do código. Sua missão é parar de perguntar *"que payload funciona aqui?"* e começar a perguntar *"quem mais lê esta entrada antes de mim, e nós concordamos?"*. É a mesma tese do roadmap — **máquina dá amplitude, humano dá profundidade** — elevada ao nível de research: a máquina te mostra os N componentes; **você** encontra onde eles se contradizem.

Próximo passo prático: leve esta lente para o [playbook de classes de bug](classes-de-bug.md) e para as [técnicas avançadas e cadeias](tecnicas-avancadas-e-cadeias.md), e conecte cada superfície ao loop [CTF ↔ bug bounty](../02-trilha/03-ctf-cve-carreira-f4-f7.md#-f4--ctf--labs-a-serviço-do-bug-bounty-velocidade-cadeias-e-escalonamento) da F4. Boa caçada. 🎯

---

⬅️ [Anterior: Técnicas Avançadas e Cadeias](tecnicas-avancadas-e-cadeias.md) · [⬆️ Topo](#) · [Próximo: Caçador Manual — Autorização e Lógica ➡️](manual-autorizacao-e-logica.md)
