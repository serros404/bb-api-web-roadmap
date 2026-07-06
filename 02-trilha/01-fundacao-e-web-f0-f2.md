# 02 — Fundação + Web (F0 · F1 · F2)

[🏠 Início](../README.md) › [02 · Trilha](README.md) › Fundação + Web (F0·F1·F2)

**Sumário:**
- [🥚 F0 — Setup + Mindset + Pipeline pública + Base de automação](#-f0--setup--mindset--pipeline-pública--base-de-automação) — [Missão F0](#-missão-f0--pipeline-viva) · [Checklist F0](#-checklist-f0)
- [🐣 F1 — Fundação Técnica](#-f1--fundação-técnica) — [Missão F1](#-missão-f1--3-scripts-no-github) · [Checklist F1](#-checklist-f1)
- [🦅 F2 — Web BB Modern (★ o coração)](#-f2--web-bb-modern--o-coração) — [Missão F2](#-missão-f2--1º-bug-aceito-em-programa-real) · [Checklist F2](#-checklist-f2)

---

> A superfície vem do [motor de recon](../01-recon/recon-engine-ia-e-automacao.md). Aqui você constrói a base (F0/F1) e a capacidade central de transformar superfície em bug **com validação manual** (F2). Releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase).

---

# 🥚 F0 — Setup + Mindset + Pipeline pública + Base de automação

**Duração:** 2–3 semanas · **Carga:** ~15h/sem · **Badge:** 🥚 *Iniciado*

**Foco:** montar o ambiente, calibrar a mentalidade certa para 2026 e colocar no ar a **pipeline pública** + um **recon mínimo automatizado** já funcionando.

<details>
<summary>📚 <strong>Estuda</strong> — mentalidade e modelo de 2026</summary>

- Internalize a [metodologia 2026](../README.md#-a-metodologia-2026-amplitude-por-máquina-profundidade-por-humano): **máquina coleta, humano explora**. Você não é "manual" nem "scanner" — é os dois, em ordem.
- Entenda o ecossistema BB: plataformas ([HackerOne](https://hackerone.com), [Bugcrowd](https://bugcrowd.com), [Intigriti](https://intigriti.com), [YesWeHack](https://yeswehack.com)), tipos de programa (público/privado, VDP vs pago), e como funciona triagem, duplicata, severidade (CVSS) e payout.
- Leia 5–10 writeups variados para ver o "shape" de um bug real (não para copiar — para sentir o padrão). Comece em [Pentester.land](https://pentester.land/list-of-bug-bounty-writeups.html).
- Estude **escopo e Safe Harbor**: como ler a política de um programa e nunca sair do permitido.

</details>

<details>
<summary>🧪 <strong>Pratica</strong> — monte o lab e o recon mínimo</summary>

**Ambiente de trabalho (manual):**
- **Burp Suite** (Community serve para começar; Pro vale o investimento na F2) — proxy, Repeater, Intruder.
- **[Caido](https://caido.io)** — alternativa moderna e leve ao Burp; muitos caçadores rodam os dois.
- **DevTools** do navegador (Chrome/Firefox) — sua lupa de front-end. Aprenda Network, Sources, Console, Debugger.
- Navegador dedicado a teste + extensões (FoxyProxy, Wappalyzer, Cookie-Editor).

**Labs locais (Docker) para destruir à vontade:**
```bash
docker run -d -p 3000:3000 bkimminich/juice-shop      # web moderno
docker run -d -p 80:80 vulnerables/web-dvwa            # clássicos
```

**Base de automação (instale o essencial do [motor de recon](../01-recon/recon-engine-ia-e-automacao.md)):**
```bash
# Toolkit ProjectDiscovery via pdtm (instala subfinder, httpx, nuclei, katana, dnsx, naabu...)
go install -v github.com/projectdiscovery/pdtm/cmd/pdtm@latest && pdtm -ia
# Utilitários do tomnomnom
go install github.com/tomnomnom/{assetfinder,anew,qsreplace,unfurl,gf}@latest
# Atualize os templates do nuclei
nuclei -update-templates
```
- Configure as **API keys** do subfinder (Censys, SecurityTrails, GitHub, Shodan...).
- Faça um primeiro recon end-to-end **em um alvo autorizado** (ex.: um programa VDP público com escopo amplo) só para validar o pipeline.

</details>

<details>
<summary>✍️ <strong>Posta</strong> — coloque a pipeline pública no ar</summary>

- Crie **blog** (GitHub Pages, Hashnode ou Medium) — será onde moram seus writeups.
- Perfil profissional: **X/Twitter** (infosec vive lá), **LinkedIn**, e **GitHub** organizado.
- Crie os repositórios-base: `recon-scripts`, `ctf-writeups`, `notes` (seu segundo cérebro).
- Primeiro post: "começando em bug bounty web/API — meu setup e metodologia" (documentar a jornada já constrói audiência).

</details>

<details>
<summary>📖 <strong>Writeups da semana</strong></summary>

Leia 3–5/semana e anote: que classe de bug? como o autor *achou* (recon? leitura de JS? intuição de lógica?)? eu teria achado? Foque em writeups que mostram **metodologia de descoberta**, não só o payload final.

</details>

### 🎯 Missão F0 — "Pipeline viva"
Tenha simultaneamente no ar: **(1)** lab local rodando, **(2)** um recon automatizado mínimo que, a partir de um domínio, te entrega hosts vivos + screenshots + URLs, e **(3)** blog/perfis publicados com o primeiro post.

**Critério de aceitação:** você consegue rodar `subfinder → httpx → gowitness` em um alvo autorizado e abrir a galeria de screenshots; e seu blog tem 1 post no ar.

### ✅ Checklist F0
- [ ] Burp/Caido + DevTools configurados
- [ ] Juice Shop + DVWA rodando local
- [ ] Toolkit de recon instalado + API keys do subfinder
- [ ] Recon mínimo end-to-end validado em alvo autorizado
- [ ] Blog + X + LinkedIn + GitHub no ar
- [ ] Repos `recon-scripts`, `ctf-writeups`, `notes` criados
- [ ] 1º post publicado

---

# 🐣 F1 — Fundação Técnica

**Duração:** 4–8 semanas · **Carga:** ~15h/sem · **Badge:** 🐣 *Fundado*

**Foco:** os fundamentos sem os quais nada do resto faz sentido. Sem isso, você vira "automation runner" — roda ferramenta e não entende o output.

<details>
<summary>📚 <strong>Estuda</strong> — HTTP, web, JS, Python, SQL, arquitetura</summary>

**HTTP & protocolos web (a língua de tudo):**
- Métodos, status codes, headers (Host, Origin, Referer, Cookie, Authorization, CORS headers), cookies e atributos (`HttpOnly`, `Secure`, `SameSite`), cache, redirects.
- HTTP/1.1 vs HTTP/2 (importa para request smuggling, F2).
- Referência: [MDN HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP), [web.dev](https://web.dev/learn).

**JavaScript (ler E escrever):**
- *Ler:* entender JS de aplicações reais, inclusive minificado, é o que destrava DOM XSS e descoberta de endpoints.
- *Escrever:* DOM, eventos, `fetch`/XHR, `postMessage`, promises, `async/await`.
- *Modelo de protótipos:* entenda `__proto__`, `constructor.prototype` e herança — é a base de **prototype pollution**, a classe-assinatura do ecossistema (ver [F2](#-f2--web-bb-modern--o-coração) e a [lente Orange Tsai](../03-playbooks/orange-tsai-decodificado-js.md)).
- Referência: [JavaScript.info](https://javascript.info), [Eloquent JavaScript](https://eloquentjavascript.net/), [MDN JS](https://developer.mozilla.org/en-US/docs/Web/JavaScript).

**Node.js & runtime (a base do pivô 2026):**
- Como um servidor Node processa uma request: **event loop**, **streams**, `Buffer`, `child_process`, sistema de módulos (`require`/ESM) e `node_modules` (o source das libs — dá para lê-lo).
- Como os frameworks **roteiam e parseiam**: Express (`qs` na query, `body-parser`, ordem de middleware), Fastify, **Next.js** (middleware vs handler), e o parser HTTP `llhttp`. Saber *onde* cada entrada é interpretada é o que habilita achar [diferenciais de parser](../03-playbooks/tecnicas-avancadas-e-cadeias.md#h--diferenciais-de-parser-e-confusão-de-componentes-a-veia-orange-tsai).
- Referência: [Node.js docs](https://nodejs.org/en/learn), [Hussein Nasser (YouTube)](https://www.youtube.com/c/HusseinNasser-software-engineering).

**Python (sua linguagem de automação):**
- Scripts de recon/automação, `requests`, parsing (JSON/HTML), threading básico.
- Referência: [Automate the Boring Stuff](https://automatetheboringstuff.com/).

**SQL & dados:**
- SELECT/INSERT/UPDATE, JOINs, subqueries — base para entender SQLi.
- Referência: [SQLBolt](https://sqlbolt.com), [W3Schools SQL](https://www.w3schools.com/sql/).
- JSON e XML (base para API e XXE).

**Arquitetura web moderna:**
- SPA vs server-rendered, REST vs GraphQL, JWT/sessões, OAuth (fluxos), CORS, CSP, proxies/CDN, microsserviços, webhooks.

</details>

<details>
<summary>🧪 <strong>Pratica</strong> — escreva ferramentas, com IA como tutor</summary>

- Refaça exercícios de JS e Python até fluir.
- **Escreva 3 scripts próprios** de recon/automação (vão pro `recon-scripts`). Ideias:
  1. Wrapper que encadeia `subfinder → dnsx → httpx` e salva resultado organizado.
  2. Parser de JS que extrai URLs/endpoints (sua versão simples de LinkFinder).
  3. Enumerador de IDOR: itera IDs e compara respostas (exemplo abaixo).
- **Use IA como tutor de fundação**, não como muleta: peça para explicar um trecho de JS que você não entende, mas reescreva o entendimento com suas palavras.

```python
# Enumerador simples de IDOR (USE APENAS EM LAB OU ESCOPO AUTORIZADO)
import requests

BASE = "http://localhost:3000/api/users/"   # alvo de LAB
headers = {"Authorization": "Bearer SEU_TOKEN_DE_TESTE"}

for uid in range(1, 50):
    r = requests.get(f"{BASE}{uid}", headers=headers, timeout=5)
    # sinaliza quando você consegue ler dados de outro usuário
    if r.status_code == 200 and "email" in r.text:
        print(f"[+] Possível IDOR em id={uid}: {r.status_code} ({len(r.text)} bytes)")
```

</details>

<details>
<summary>✍️ <strong>Posta</strong></summary>

- Publique os 3 scripts no GitHub com `README` decente (o que faz, como rodar).
- Escreva 1 post técnico de fundação (ex.: "como o JS de uma SPA revela a superfície de API").

</details>

<details>
<summary>📖 <strong>Writeups da semana</strong></summary>

Agora leia entendendo a **camada técnica**: por que aquele header importava, por que aquele endpoint de JS era a porta. Comece a conseguir reproduzir mentalmente o bug.

</details>

### 🎯 Missão F1 — "3 scripts no GitHub"
Três scripts de recon/automação seus, públicos, funcionando, com README.

**Critério de aceitação:** outra pessoa consegue clonar, ler o README e rodar seus scripts.

### ✅ Checklist F1
- [ ] HTTP/headers/cookies/CORS dominados
- [ ] Lê e escreve JS (inclusive minificado, com esforço)
- [ ] Python para automação fluindo
- [ ] SQL + JSON/XML compreendidos
- [ ] Arquitetura web moderna clara (SPA, REST/GraphQL, JWT, OAuth)
- [ ] 3 scripts publicados com README
- [ ] 1 post técnico de fundação no ar

---

# 🦅 F2 — Web BB Modern (★ o coração)

**Duração:** 3–6 meses · **Carga:** ~15–20h/sem · **Badge:** 🦅 *Caçador Web*

**Foco:** a fase central. Aqui a **superfície que a máquina mapeou ([motor de recon](../01-recon/recon-engine-ia-e-automacao.md)) vira bug** através de **validação e exploração manual** com Burp + DevTools. É onde você se torna caçador de verdade.

> **O loop de F2:** recon automatizado entrega alvos priorizados → você abre no Burp → forma hipóteses → testa manualmente → confirma → escreve PoC mínima → reporta. IA é copiloto em cada passo, nunca o piloto.

<details>
<summary>📚 <strong>Estuda</strong> — recon aplicado + Burp + as classes de bug</summary>

**Recon aplicado à caça** (revise o [motor de recon](../01-recon/recon-engine-ia-e-automacao.md) e aplique de verdade):
- Pegue um programa autorizado, rode o funil completo, e pratique a **triagem visual** + **priorização com IA** para escolher onde caçar.

**Burp Suite — maestria** (o instrumento da profundidade):
- **Proxy** (interceptar/editar), **Repeater** (manipular requests à mão — onde você vai morar), **Intruder** (fuzzing manual), **Decoder/Comparer**, **Logger**.
- Extensões essenciais (BApp Store): **Autorize** (testa autorização automaticamente), **Param Miner**, **Turbo Intruder** (race conditions), **JWT Editor**, **InQL** (GraphQL), **Logger++**, **Collaborator** (OOB para SSRF/XXE blind).
- Curso/certificação: a **[PortSwigger Web Security Academy](https://portswigger.net/web-security)** é a referência — faça **todos** os labs Apprentice + Practitioner. Meta de fim de F2: **[Burp Suite Certified Practitioner (BSCP)](https://portswigger.net/web-security/certification)**.

**As classes de bug web** (estude e pratique cada uma na Academy):

**1. DOM XSS (source → sink) — leitura de JS na veia**
- Entenda *sources* (`location`, `document.referrer`, `postMessage`...) e *sinks* (`innerHTML`, `eval`, `document.write`, `setAttribute`...).
- Trace o fluxo no JS real; use o **DOM Invader** (embutido no navegador do Burp) para acelerar, mas saiba fazer à mão.
- Referência: [DOM XSS Wiki](https://github.com/wisec/domxsswiki/wiki), [PortSwigger XSS](https://portswigger.net/web-security/cross-site-scripting).

**2. ★★ Logic flaws + IDOR/BOLA + BAC/BFLA + race conditions — onde está o dinheiro**
- **IDOR / BOLA:** trocar IDs para acessar dados de outros. Confirme com **Autorize** (duas sessões). É o bug mais comum e mais pago.
- **BAC/BFLA:** quebra de controle de acesso por função (chamar endpoint de admin como user).
- **Logic flaws:** abusar das regras de negócio (preço negativo, pular etapa de pagamento, cupom infinito). Pergunte sempre: *"o que essa função deveria impedir, e como eu quebro essa regra?"*
- **Race conditions:** explorar janelas de concorrência (resgatar cupom 2x, sacar saldo duplicado). Use **Turbo Intruder** com *single-packet attack* (técnica de James Kettle, ["Smashing the state machine"](https://portswigger.net/research/smashing-the-state-machine)).

**3. Client-side avançado**
- **postMessage** inseguro ([Securitum research](https://research.securitum.com/postmessage-xss/)).
- **CSPT (Client-Side Path Traversal)** ([Doyensec](https://blog.doyensec.com/2023/06/13/cspt.html)).
- **Prototype pollution** (client e server), **DOM clobbering**.
- **OAuth/SSO** mal configurado, **JWT** (alg confusion, chave fraca).

**4. Demais classes (server-side)**
- **SSRF** (com Collaborator para blind), **SQLi**, **XXE**, **deserialization**, **HTTP request smuggling** (HTTP/1 vs HTTP/2 desync).

</details>

<details>
<summary>🧪 <strong>Pratica</strong> — Academy + alvo real, com o pipeline híbrido</summary>

- Feche os labs **Apprentice + Practitioner** de cada classe na PortSwigger Academy. Os Expert depois.
- **Caça real (escopo autorizado):** aplique o loop completo —
  1. Recon automatizado + IA priorizam alvos ([motor de recon](../01-recon/recon-engine-ia-e-automacao.md)).
  2. Abra os 3–5 alvos mais promissores no Burp.
  3. Para cada funcionalidade, **forme uma hipótese de lógica/autorização** e teste manualmente no Repeater.
  4. Rode Autorize em duas sessões para IDOR/BAC.
  5. Confirme, escreva **PoC de impacto mínimo**.
- Use IA para: deobfuscar JS dos alvos, gerar hipóteses de ataque por funcionalidade, e revisar seu PoC/report antes de enviar.

</details>

<details>
<summary>✍️ <strong>Posta</strong></summary>

- Writeups dos labs mais instrutivos (sem violar termos da Academy — explique a técnica).
- Quando tiver um bug aceito e divulgável (pós-fix, com permissão), escreva o writeup. Isso vale ouro para reputação.

</details>

<details>
<summary>📖 <strong>Writeups da semana</strong></summary>

Foque em writeups de **logic flaws, IDOR/BOLA e race conditions** — as classes que pagam. Estude **como o autor descobriu** (que pergunta de lógica ele fez), não só o resultado.

</details>

### 🎯 Missão F2 — "1º bug aceito em programa real"
Encontre, valide manualmente e reporte um bug **aceito (triado/pago)** em um programa real, partindo de uma superfície que você mapeou com automação.

**Critério de aceitação:** report aceito (mín. triado) em plataforma de BB, com PoC de impacto mínimo, achado por validação manual sobre recon automatizado.

### ✅ Checklist F2
- [ ] Todos os labs Apprentice + Practitioner da Academy concluídos
- [ ] Burp dominado (Repeater/Intruder/extensões) + DOM Invader
- [ ] Traça DOM XSS source-to-sink lendo JS
- [ ] Acha IDOR/BOLA e BAC/BFLA manualmente (confirma com Autorize)
- [ ] Reproduziu race condition (single-packet/Turbo Intruder) em lab
- [ ] Domina ≥1 vetor client-side avançado ponta a ponta
- [ ] Sabe quando/como testar SSRF, SQLi, XXE, deser, smuggling
- [ ] Loop híbrido (recon→Burp→validação manual) é seu fluxo padrão
- [ ] 1º bug aceito em programa real
- [ ] (Meta) BSCP conquistado

---

⬅️ [Anterior: Recon Engine — IA + Automação](../01-recon/recon-engine-ia-e-automacao.md) · [⬆️ Topo](#) · [Próximo: API Bug Bounty (F3) ➡️](02-api-f3.md)
