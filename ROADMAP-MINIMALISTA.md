# ⚡ ROADMAP MINIMALISTA — Bug Bounty na veia (manual-first)

[🏠 Início](README.md) · *o caminho enxuto. Zero pipeline gigante, zero enrolação.*

> **4 entradas, foco no que scanner não pega, Burp + DevTools + cabeça.** O [roadmap completo](README.md) fica ali pra quando você quiser fundo — **este aqui é o que você faz todo dia.**

---

## A regra (uma frase)

**Bug bounty é o objetivo (a renda). PortSwigger + CTF + HTB + writeups é como você fica bom.** Você caça o que **scanner nenhum acha** — falha lógica, IDOR/BOLA, DOM XSS, race, auth quebrada. A máquina só te leva até a porta; **você arromba, à mão.**

---

## ONLY isto — as 4 entradas

**1. 🎓 [PortSwigger Web Security Academy](https://portswigger.net/web-security)** — a espinha. Faça **todos** os labs (Apprentice → Practitioner → Expert). Prioridade absoluta: **[Access control](https://portswigger.net/web-security/access-control) · [Business logic](https://portswigger.net/web-security/logic-flaws) · [Authentication](https://portswigger.net/web-security/authentication) · [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)**. De graça, e é o melhor treino que existe. Prova final: **[BSCP](https://portswigger.net/web-security/certification)**.

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

## Recon (o mínimo — só pra achar superfície)

Recon **não é o jogo**; é só o que te leva ao alvo manual. Um objetivo: **subdomínios vivos e esquecidos/distantes** (staging, legado, aquisições — o que ninguém olha).

```bash
# vivos + tech + título, numa linha:
subfinder -d alvo.com -all -silent | httpx -silent -sc -title -td | tee live.txt

# olhe os "distantes": dev. / staging. / old. / api-v1. / *.empresa-adquirida.com
# puxe URLs e JS pra ler à mão (endpoints, params, segredos):
katana -u https://sub.alvo.com -jc -silent | tee urls.txt      # ou: gau alvo.com
```

Achou os vivos + esquecidos → **fecha o terminal e vai pro Burp.** O resto é manual.

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
