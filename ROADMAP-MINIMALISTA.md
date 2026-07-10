# ⚡ ROADMAP MINIMALISTA — Bug Bounty na veia (manual-first)

[🏠 Início](README.md) · *o caminho enxuto. Zero pipeline gigante, zero enrolação.*

> **4 entradas, foco no que scanner não pega, Burp + DevTools + cabeça.** O [roadmap completo](README.md) fica ali pra quando você quiser fundo — **este aqui é o que você faz todo dia.**

---

## A regra (uma frase)

**Bug bounty é o objetivo (a renda). PortSwigger + CTF + HTB + writeups é como você fica bom.** Você caça o que **scanner nenhum acha** — falha lógica, IDOR/BOLA, DOM XSS, race, auth quebrada. A máquina só te leva até a porta; **você arromba, à mão.**

---

## ONLY isto — as 4 entradas

**1. 🎓 [PortSwigger Web Security Academy](https://portswigger.net/web-security)** — a espinha. Faça **todos** os labs (Apprentice → Practitioner → Expert). Prioridade absoluta: **[Access control](https://portswigger.net/web-security/access-control) · [Business logic](https://portswigger.net/web-security/logic-flaws) · [Authentication](https://portswigger.net/web-security/authentication) · [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)**. De graça, e é o melhor treino que existe. Prova final: **[BSCP](https://portswigger.net/web-security/certification)**. 👉 **trilha semana a semana (31 topics, ordem por ROI): [plano-portswigger.md](plano-portswigger.md)**.

**2. 🚩 CTF (web)** — criatividade, classes raras, velocidade. [picoCTF](https://picoctf.org) + [Hacker101 CTF](https://www.hacker101.com) *(destrava convites na HackerOne)* → [Intigriti challenges](https://www.intigriti.com) + eventos web no [CTFtime](https://ctftime.org) (NahamCon, corCTF...). 1 a cada 1–2 semanas.

**3. 📦 [Hack The Box](https://www.hackthebox.com)** — o músculo de **cadeia** (foothold → escala → pivot = o mesmo que encadear bugs). Máquinas web-heavy + challenges web. Burp **na veia** o tempo todo.

**4. 📖 Writeups — 3 a 5 por semana, inegociável.** Lendo pra entender o **mecanismo**, sempre com *"eu teria achado? como?"*. [Pentester.land](https://pentester.land/list-of-bug-bounty-writeups.html) · [HackerOne Hacktivity](https://hackerone.com/hacktivity) · [PortSwigger Research](https://portswigger.net/research).

---

## O que você caça (o scanner é cego aqui)

- **Falhas lógicas + IDOR/BOLA + BAC/BFLA** → o dinheiro. 100% manual, **2 contas + Autorize** (extensão do Burp). *(a sua praia)*
- **DOM XSS** → lendo o **JS**, source → sink, no DevTools + **DOM Invader**.
- **Race conditions · mass assignment · OAuth/JWT/reset logic** → contexto, não payload.
- Regra de bolso: **se um scanner acharia, já foi achado.** Cace o que exige cabeça.

---

## As armas (na veia)

- **Burp** → Repeater é sua **casa**. + Intruder, **Autorize** (IDOR/BAC com 2 sessões), **DOM Invader** (DOM XSS / prototype pollution / postMessage), Collaborator (blind).
- **DevTools** → Network, Sources, Debugger, Console. É aqui que você **lê JS** e traça source→sink.
- **Cabeça** → a pergunta *"o que isto deveria impedir — e como eu quebro essa regra?"*.

---

## Recon (o mínimo — domínios, subs, JS, APIs, keys)

Recon **não é o jogo**; é o que te leva ao alvo manual. Objetivo: **subs vivos + esquecidos**, os **endpoints/APIs escondidos no JS**, e **keys vazadas**. O funil, do amplo ao fino:

```
DOMÍNIOS → SUBDOMÍNIOS → VIVOS → URLs/JS → ENDPOINTS/APIs + KEYS → Burp
```

**1) Domínios (raiz).** Vêm do **escopo** do programa. Pra achar outras raízes da mesma org sem instalar nada (certificados):
```bash
curl -s 'https://crt.sh/?q=%25.alvo.com&output=json' | jq -r '.[].name_value' | sort -u
```

**2) Subdomínios** — 1 domínio **ou** vários (lista com `-dL`):
```bash
subfinder -d alvo.com -all -silent | tee subs.txt              # 1 domínio
subfinder -dL roots.txt -all -silent | anew subs.txt           # vários
```

**3) Vivos** (quais respondem + tech/título). Guarde também só as **URLs limpas** pra alimentar o resto:
```bash
httpx -l subs.txt -silent -sc -title -td -location | tee live.txt
httpx -l subs.txt -silent > live_urls.txt
```
> 👀 Caça os **esquecidos/distantes**: `dev.` `staging.` `old.` `api-v1.` `internal.` `*.empresa-adquirida.com` — é onde a authz é fraca e ninguém olha.

**4) URLs + endpoints** — histórico (`gau`, passivo) **+** crawl ao vivo que entra no JS (`katana`):
```bash
# 1 host:
gau sub.alvo.com | anew urls.txt                               # Wayback/CommonCrawl/URLScan/OTX
katana -u https://sub.alvo.com -jc -d 3 -silent | anew urls.txt
# vários (lista):
echo alvo.com | gau --subs | anew urls.txt
katana -list live_urls.txt -jc -d 3 -silent | anew urls.txt
```

**5) JS a fundo → endpoints escondidos + keys.** O JS é onde mora a superfície de API e, às vezes, **segredo vazado**:
```bash
grep -iE '\.js($|\?)' urls.txt | sort -u > js.txt
# baixa os JS e extrai paths/endpoints de dentro deles:
while read u; do curl -sk "$u"; done < js.txt | jsluice urls | anew urls.txt
# caça keys/segredos no JS (jsluice tem modo secrets):
while read u; do curl -sk "$u"; done < js.txt | jsluice secrets
```

**6) APIs** — ache a superfície no que você já tem:
```bash
grep -iE 'api|/v[0-9]|graphql|swagger|openapi|\.json' urls.txt | sort -u
# docs abertas = mapa do tesouro:  /swagger.json  /openapi.json  /api-docs  /graphql (tente introspection)
```

**7) Triagem — o que abrir no Burp:**
```bash
sort -u urls.txt -o urls.txt
grep -iE '\?|=' urls.txt                                        # params → IDOR / logic / XSS
grep -iE 'admin|internal|export|upload|account|invoice|user|order|api|graphql' urls.txt
```

Pegou os suculentos → **fecha o terminal, abre o Burp.** 2 contas + Autorize (IDOR/BOLA), lê o JS (DOM XSS), quebra a lógica. **Recon te leva à porta; você arromba à mão.**

> 🏙️ **Alvo é big tech?** Aí o recon vira outro jogo (ASN, aquisições, JS/mobile, legado/`actuator`, **funções pouco usadas**) → **[recon-big-tech.md](recon-big-tech.md)**. E **[onde reportar pra Hall of Fame](programas-hall-da-fama.md)** (NASA, Google, Apple, OpenAI, X...).

<details><summary>🔧 <strong>Instala tudo (uma vez)</strong></summary>

```bash
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/projectdiscovery/katana/cmd/katana@latest
go install github.com/lc/gau/v2/cmd/gau@latest
go install github.com/tomnomnom/anew@latest
go install github.com/BishopFox/jsluice/cmd/jsluice@latest
# keys em repositórios: trufflehog / gitleaks (via brew/apt)  ·  jq: brew/apt install jq
```
</details>

---

## O loop (a rotina)

```
LAB (PortSwigger)  → ganha o movimento da classe
      ↓
CTF / HTB          → velocidade + encadeamento
      ↓
WRITEUP            → vê como os bons acham
      ↓
CAÇA REAL          → no que scanner não pega, dentro de escopo
      ↓  ── repete, mais afiado ──┘
```

**Semana enxuta (~10–15h):** caça (metade do tempo) + labs PortSwigger + 3–5 writeups + 1 CTF/HTB. Travou? **Troca de entrada, não para.**

---

## Régua — quando você chegou

- [ ] Fechou **Access Control + Business Logic + Authentication + DOM XSS** da PortSwigger
- [ ] Acha **IDOR/BOLA/logic** à mão (2 contas + Autorize)
- [ ] Traça **DOM XSS** lendo JS no DevTools
- [ ] Encadeia (HTB/CTF): `low` + `low` → `critical`
- [ ] **1º bug real aceito** no que scanner não pega
- [ ] **BSCP** no bolso

---

## Anti-regras (o que te enrola)

- ❌ **Pipeline gigante de recon.** Você quer **vivos + esquecidos**, mais nada.
- ❌ **Rodar scanner e mandar o output.** Isso é ruído, não caça — e queima reputação.
- ❌ **Decorar payload.** Entenda o **mecanismo** (é pra isso que você lê writeup).
- ❌ **Recon eterno sem abrir o Burp.** Se não testou à mão, você não caçou.

---

> **Quer fundo depois (e só se quiser):** [Caçador Manual — Autz & Lógica](03-playbooks/manual-autorizacao-e-logica.md) · [DOM XSS / classes web (F2)](02-trilha/01-fundacao-e-web-f0-f2.md#-f2--web-bb-modern--o-coração) · [playbook de classes](03-playbooks/classes-de-bug.md). Mas o que muda o jogo é **fazer** o de cima — hoje. Boa caçada. 🎯
