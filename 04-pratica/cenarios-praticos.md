# 09 — Cenários Práticos: a metodologia 2026 ponta a ponta

[🏠 Início](../README.md) › [04 · Prática](README.md) › Cenários Práticos

**Sumário:**
- [🎯 Cenário 1 — Programa grande de escopo amplo](#-cenário-1--programa-grande-de-escopo-amplo-exemplocom)
- [🎯 Cenário 2 — API REST com Swagger exposto](#-cenário-2--api-rest-com-swagger-exposto)
- [🎯 Cenário 3 — SPA moderna (React/Vue) com backend GraphQL](#-cenário-3--spa-moderna-reactvue-com-backend-graphql)
- [🎯 Cenário 4 — App com login social (OAuth / SSO)](#-cenário-4--app-com-login-social-oauth--sso)
- [🎯 Cenário 5 — Ativos esquecidos & subdomain takeover](#-cenário-5--ativos-esquecidos--subdomain-takeover-recon-contínuo)
- [🎯 Cenário 6 — CVE hunting num OSS](#-cenário-6--cve-hunting-num-oss-ex-pluginprojeto-self-hosted)
- [🔁 O padrão por trás dos seis cenários](#-o-padrão-por-trás-dos-seis-cenários)

---

> Aqui tudo se junta. Seis cenários reais mostram o fluxo completo: **Fase A (coleta com IA + automação) → priorização → Fase B (validação e exploração manual) → report**. São playbooks de *como pensar*, não receitas para copiar cegamente. Tudo para uso **autorizado** (releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase)). Nomes e dados são fictícios.

> **Como ler cada cenário:** observe sempre o mesmo padrão — a máquina mapeia rápido e amplo; um humano olha screenshots e prioriza; e a exploração de autorização/lógica é manual. É a tese do roadmap aplicada.

---

## 🎯 Cenário 1 — Programa grande de escopo amplo (`*.exemplo.com`)

**Contexto:** programa público com escopo wildcard, centenas de subdomínios, muitos olhos competindo. Objetivo: chegar rápido a um alvo negligenciado e achar o primeiro bug.

### Fase A — Coleta em escala
```bash
# 1) domínios e ranges do alvo
asnmap -org "ExemploCorp" -silent | tee asn.txt
curl -s "https://crt.sh/?q=%25.exemplo.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u > ct.txt

# 2) subdomínios (passiva + permutação) e resolução
subfinder -d exemplo.com -all -silent | anew subs.txt
gotator -sub subs.txt -perm palavras.txt -depth 1 | puredns resolve -r resolvers.txt | anew subs.txt
cat subs.txt | dnsx -silent > resolvidos.txt

# 3) hosts vivos + tech + screenshots
cat resolvidos.txt | httpx -silent -title -tech-detect -status-code | tee hosts.txt
cat hosts.txt | cut -d' ' -f1 | gowitness scan file -f - --screenshot-path shots/
```

### Priorização (humano + IA)
- Abra a galeria de screenshots: em 10 min separe **painéis de admin, ambientes dev/stg, apps legados, telas de erro 500** das centenas de páginas de marketing.
- Jogue `hosts.txt` na IA: *"agrupe por tecnologia, destaque painéis administrativos, APIs versionadas antigas e qualquer coisa que cheire a staging/debug; priorize o que olhar primeiro e por quê."*
- Resultado típico: 3–5 alvos suculentos (ex.: `admin-legacy.exemplo.com`, `api-v1.exemplo.com`, `staging-app.exemplo.com`).

### Fase B — Validação e exploração manual
- Abra `api-v1.exemplo.com` no Burp. APIs `v1` legadas raramente têm os controles da `v2` → teste **BOLA** com duas contas (Autorize).
- Em `admin-legacy`, faça force browsing dos paths achados no recon → teste **BFLA** (chamar função de admin como user).
- No app de staging, procure **logic flaws** (dados de produção em ambiente sem as mesmas validações).

### Report
- Achou IDOR/BOLA em `api-v1`? Report com duas contas, PoC de impacto mínimo (um registro alheio, redigido), impacto de negócio claro. Modelo em [Apêndices — Modelo de report](../05-referencia/apendices.md#-e--modelo-de-report).

> **Lição:** a automação não achou o bug — ela te levou em minutos ao subdomínio esquecido onde o bug estava. A validação de autorização foi 100% sua.

---

## 🎯 Cenário 2 — API REST com Swagger exposto

**Contexto:** você encontrou `https://api.exemplo.com/swagger.json` (ou `/openapi.json`, `/api-docs`). É um mapa do tesouro.

### Fase A — Coleta
```bash
# Baixa o schema e extrai todos os endpoints + métodos
curl -s https://api.exemplo.com/swagger.json | jq -r '.paths | keys[]' > endpoints.txt
# Descobre parâmetros escondidos em endpoints sensíveis
arjun -u https://api.exemplo.com/v2/account -m GET,POST
# Brute de rotas não documentadas
kr scan https://api.exemplo.com -w routes-large.kite -x 20
```

### Priorização (IA)
- Cole o `swagger.json` na IA: *"liste os endpoints mais sensíveis (auth, usuários, pagamentos, admin), o método de cada um, e gere hipóteses de BOLA/BFLA/mass assignment por endpoint."*
- A IA monta um "plano de ataque" priorizado a partir da própria documentação.

### Fase B — Exploração manual
- **BOLA (API1):** em cada endpoint `/{id}`, troque o ID entre duas contas.
- **Mass assignment (API3/BOPLA):** num `PATCH /users/me`, injete `"role":"admin"`, `"verified":true`, `"balance":99999` e veja o que cola.
- **Inventário (API9):** teste `/v1` para cada `/v2` documentado; a v1 esquecida costuma estar sem controle.
- **Excessive data exposure:** olhe o JSON cru — campos que o front esconde (tokens, hashes, PII) vêm a mais.

### Report
- Mass assignment que vira escalonamento de privilégio → severidade alta. Mostre o request com o campo injetado e a resposta refletindo o privilégio.

> **Lição:** a doc + IA te deram a superfície inteira da API instantaneamente. A quebra de autorização e o mass assignment foram testados request a request, à mão.

---

## 🎯 Cenário 3 — SPA moderna (React/Vue) com backend GraphQL

**Contexto:** app single-page; quase nada aparece no HTML; toda a lógica e os endpoints vivem no JavaScript e numa API GraphQL.

### Fase A — Coleta (JS é o mapa)
```bash
# Coleta todo o JS e extrai endpoints/rotas/segredos
cat hosts.txt | cut -d' ' -f1 | getJS --complete | sort -u > js.txt
cat js.txt | jsluice urls -R | sort -u > js_endpoints.txt
cat js.txt | xargs -I{} python3 SecretFinder.py -i {} -o cli 2>/dev/null

# Identifica o endpoint GraphQL e faz fingerprint
python3 graphw00f.py -d -f -t https://exemplo.com/graphql
```

### Priorização (IA)
- Cole um arquivo JS minificado/grande na IA: *"deobfusque mentalmente e liste rotas de API, endpoints GraphQL, mutations e quaisquer chaves/segredos; aponte sources e sinks de DOM XSS."*
- A IA reconstrói a superfície que LinkFinder/JSluice deixaram ambígua.

### Fase B — Exploração manual
- **GraphQL introspection:** `{ __schema { types { name fields { name } } } }` → schema completo. Se desabilitada, tente *field suggestion* / clairvoyance.
- **BOLA/BFLA em resolvers:** teste as mutations sensíveis (criar usuário, mudar papel, transferir) com papéis diferentes.
- **Batching/aliases:** abuse para brute de OTP ou para burlar rate limit.
- **DOM XSS:** trace os sinks que a IA apontou no JS, confirme com DOM Invader, explore à mão.

### Report
- BOLA em mutation de GraphQL (ex.: `updateUser(id:...)` aceita ID alheio) → report com a query, as duas sessões e o impacto.

> **Lição:** numa SPA, sem ler o JS você não enxerga nada. Automação + IA extraem a superfície do JS; você quebra a autorização nos resolvers.

---

## 🎯 Cenário 4 — App com login social (OAuth / SSO)

**Contexto:** "Entrar com Google/GitHub". Fluxos OAuth concentram bugs de alto impacto (roubo de conta).

### Fase A — Coleta
- Mapeie o fluxo inteiro no Burp: requisição de autorização, `redirect_uri`, `state`, `code`, callback.
- Note todos os parâmetros e para onde o token/`code` trafega.
- Procure **open redirects** no recon (`gf redirect`) — são o gadget que torna o OAuth explorável.

### Priorização (IA)
- Descreva o fluxo capturado para a IA: *"aqui está o fluxo OAuth desta app; liste os pontos de falha clássicos (redirect_uri, state, response_type, pre-account-takeover) e como eu testaria cada um."*

### Fase B — Exploração manual
- **`redirect_uri` mal validado:** tente apontar para um domínio seu (ou um open redirect do próprio alvo) e exfiltrar o `code`/token.
- **`state` ausente/fraco:** CSRF no fluxo (account linking forçado).
- **Vazamento via Referer:** veja se `code`/token vaza para terceiros.
- **Pre-account takeover:** crie conta via SSO antes da vítima confirmar e-mail.
- Encadeie: **open redirect + redirect_uri → roubo de token → ATO** (playbook [C5](../03-playbooks/classes-de-bug.md#c5-oauth--sso-misconfiguration)/[C8](../03-playbooks/classes-de-bug.md#c8-account-takeover-ato--encadeamento)).

### Report
- Cadeia que resulta em ATO → severidade crítica. Documente cada elo e o resultado final (tomar a conta de outro usuário, demonstrado com contas de teste).

> **Lição:** nenhum scanner "entende" um fluxo OAuth. Aqui a máquina só ajudou a achar o open redirect; a cadeia até o ATO é raciocínio humano puro.

---

## 🎯 Cenário 5 — Ativos esquecidos & subdomain takeover (recon contínuo)

**Contexto:** você quer vantagem temporal — ser o primeiro a ver o que acabou de ser exposto. Aqui o **monitoramento contínuo** de [Recon Contínuo + API Avançado](../01-recon/recon-continuo-e-api-avancado.md) brilha.

### Fase A — Pipeline contínuo
```bash
# monitor.sh agendado (cron) — guarda só novidades e alerta
subfinder -d exemplo.com -all -silent | dnsx -silent | anew subs.txt | tee novos.txt
[ -s novos.txt ] && httpx -silent -title -tech-detect < novos.txt | notify -silent -bulk
# checa takeover em todos os subs
cat subs.txt | dnsx -cname -resp-only | subzy run --targets - --hide_fails | notify -silent
```

### Priorização (automática + humano)
- O alerta chega no seu Discord/Telegram. Você investiga **só o diff**: um subdomínio novo, um CNAME órfão, uma tecnologia nova que apareceu.

### Fase B — Validação manual
- **Subdomain takeover:** CNAME aponta para um serviço desativado (S3 bucket, Heroku, GitHub Pages não reclamado)? Confirme manualmente que dá para reivindicar — e prove com impacto mínimo (uma página inócua sua, sem hospedar conteúdo malicioso).
- **Subdomínio novo:** rode o funil completo nele e ataque como nos cenários anteriores.

### Report
- Subdomain takeover é alto valor e fácil de demonstrar: mostre o CNAME órfão e a prova controlada de reivindicação.

> **Lição:** a automação não só mapeia — ela **vigia**. Quem chega primeiro no ativo recém-exposto acha o bug antes da duplicata. A confirmação do takeover, porém, é manual e cuidadosa.

---

## 🎯 Cenário 6 — CVE hunting num OSS (ex.: plugin/projeto self-hosted)

**Contexto:** você quer um CVE próprio (F5). Alvo: um projeto open source com adoção e código auditável.

### Fase A — Coleta no código
```bash
git clone https://github.com/org/projeto && cd projeto
# Grep por sinks perigosos (PHP de exemplo)
grep -rnE '\b(eval|system|exec|unserialize|include|require)\s*\(' .
grep -rnE '(query|execute)\s*\(.*(\+|%|\$).*' .   # SQL concatenado
```

### Priorização (IA como copiloto de code review)
- Cole as funções suspeitas (código **público**, nunca sob NDA): *"Aja como auditor de AppSec. Esta função processa upload/entrada do usuário — aponte sinks perigosos, validação ausente, e como um atacante exploraria. Rastreie o fluxo do input até o sink."*
- A IA acelera a triagem de centenas de funções; **você valida cada hipótese**.

### Fase B — Confirmação manual
- Suba o projeto **localmente** (Docker/dev). Reproduza o bug com PoC mínima **na sua instância** — jamais contra instância de terceiro.
- Entenda a causa-raiz a fundo (você precisa explicá-la no advisory).

### Report / disclosure
- Reporte de forma **coordenada** ao mantenedor (ou via huntr/Patchstack/GHSA). Peça CVE. Só publique o writeup **após o fix**.

> **Lição:** IA + grep varrem o código rápido; a confirmação roda no seu lab, e o disclosure é coordenado. Mesma filosofia da caça web: máquina amplia, humano valida.

---

## 🔁 O padrão por trás dos seis cenários

Repare que, mude o alvo que mudar, a estrutura é idêntica:

| Etapa | Quem faz | O que acontece |
|---|---|---|
| **Coleta** | 🤖 Máquina (automação + IA) | Mapeia a superfície inteira, rápido e amplo |
| **Priorização** | 🤝 Humano + IA | Triagem visual + IA apontam onde vale a energia humana |
| **Validação/exploração** | 🧠 Humano | Autorização, lógica, cadeias — onde estão os bugs que pagam |
| **Report/disclosure** | 🧠 Humano | PoC de impacto mínimo, coordenado, profissional |

> **Essa é a tese do roadmap inteiro, em ação:** *amplitude por máquina, profundidade por humano.* Em 2026, quem só faz manual perde a amplitude; quem só faz automação perde a profundidade (e a reputação). O caçador que vence faz os dois — nesta ordem.

---

> **Fim dos cenários.** Volte ao [README](../README.md) para o mapa, ao [motor de recon](../01-recon/recon-engine-ia-e-automacao.md) para o motor, ao [playbook de classes](../03-playbooks/classes-de-bug.md) para as classes e aos [apêndices](../05-referencia/apendices.md) para a operação. Agora é prática deliberada e disciplina. Sobe um lab, roda teu primeiro funil num alvo autorizado, e começa. Boa caçada. 🎯

---

⬅️ [Anterior: Apêndices Práticos](../05-referencia/apendices.md) · [⬆️ Topo](#) · [Próximo: Setup Completo + Cookbook de IA ➡️](../05-referencia/setup-completo-e-cookbook-ia.md)
