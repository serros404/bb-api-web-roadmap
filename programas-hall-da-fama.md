# 🏅 Programas de Bug Bounty com Hall da Fama (e onde reportar)

[🏠 Início](README.md) · combustível do seu pipeline: **acha → reporta → disclosure liberado → pesquisa → blog**

> Programas onde um bug **aceito** vira **reconhecimento público** (Hall of Fame / acknowledgment / leaderboard) — a matéria-prima do seu blog e do seu nome. **Confira sempre a política/escopo atual** antes de testar; links e regras mudam.

## ⚠️ Estratégia antes da lista (leia isto)

- **Google/Apple HoF é difícil.** A superfície deles é a mais disputada do planeta — centenas de caçadores full-time. Entrar lá é troféu de anos, não de mês 1.
- **Seu 1º HoF sai mais fácil em VDP / "recognition only".** Programas **sem dinheiro** (NASA, governos, universidades, muitas empresas com "recognition") têm **muito menos concorrência** e existem exatamente pra te dar o **crédito público**. É o caminho realista pro primeiro nome no muro — e rende o **mesmo writeup**.
- **HoF ≠ prêmio de participação.** Precisa de um bug **válido e triado**. VDP não paga, mas o **reconhecimento + o writeup pós-disclosure** é o que constrói sua autoridade (que é o seu objetivo).

## 🏛️ Hall of Fame self-hosted — nomes de peso (troféu de blog/CV)

| Empresa | Onde reportar / HoF | Paga? | Nota |
|---|---|---|---|
| **Google** | [bughunters.google.com](https://bughunters.google.com) · [Honorable Mentions](https://bughunters.google.com/leaderboard/honorable-mentions) | ✅ VRP | HoF + leaderboard próprios; disputadíssimo |
| **Apple** | [security.apple.com](https://security.apple.com) · créditos nos security advisories | ✅ | Apple Security Bounty + crédito por advisory |
| **Microsoft (MSRC)** | [microsoft.com/msrc/bounty](https://www.microsoft.com/en-us/msrc/bounty) · Researcher Acknowledgments + leaderboard | ✅ | Acknowledgments públicos + leaderboard |
| **Meta / Facebook** | [bugbounty.meta.com](https://bugbounty.meta.com/leaderboard/) · [whitehat/thanks](https://www.facebook.com/whitehat/thanks/) | ✅ | Leaderboard + thanks page |
| **Mozilla** | [Hall of Fame](https://www.mozilla.org/en-US/security/bug-bounty/hall-of-fame/) | ✅ | HoF clássico (hoje via HackerOne) |

## 🏅 VDP / recognition — o caminho realista pro 1º HoF (pouca concorrência)

| Programa | Onde | Paga? | Nota |
|---|---|---|---|
| **NASA** | [Bugcrowd VDP](https://bugcrowd.com/engagements/nasa-vdp) · [HackerOne](https://hackerone.com/nasa) · [política](https://www.nasa.gov/vulnerability-disclosure-policy/) | 🏅 VDP | Sem dinheiro, mas "achei bug na NASA" é ouro de conteúdo |
| **US DoD** (Hack the Pentagon) | [hackerone.com/deptofdefense](https://hackerone.com/deptofdefense) | 🏅 VDP | Superfície gigante, recognition |
| **Governos / .gov / .edu** | via [HackerOne](https://hackerone.com/directory/programs) / [Bugcrowd](https://bugcrowd.com/programs) (US GSA, UK, vários países, universidades) | 🏅 | Muitos VDPs abertos — bom pra **volume** de HoF |
| **Adobe** | [hackerone.com/adobe](https://hackerone.com/adobe) | 🏅 pontos/HoF | Reconhecimento por pontos (histórico sem cash em web) |

## 🌐 Big tech via plataforma — reputação = HoF de fato

Aqui seu **perfil público** na plataforma (com bugs resolvidos) **é** o hall da fama, e muitos têm thanks/leaderboard próprio:

| Empresa | Programa |
|---|---|
| **X / xAI** | [hackerone.com/x](https://hackerone.com/x) |
| **OpenAI** | [Bugcrowd](https://openai.com/index/bug-bounty-program/) |
| **GitHub** | [hackerone.com/github](https://hackerone.com/github) · [bounty.github.com](https://bounty.github.com) |
| **GitLab** | [hackerone.com/gitlab](https://hackerone.com/gitlab) |
| **Shopify · Uber · PayPal · Dropbox · Yahoo · Snap · TikTok · Airbnb · Spotify · Netflix · Atlassian · Cloudflare · Valve · Sony · Nintendo** | via [HackerOne directory](https://hackerone.com/directory/programs) / [Bugcrowd programs](https://bugcrowd.com/programs) |

## 🔎 Onde achar MAIS (diretórios)

- [HackerOne — directory](https://hackerone.com/directory/programs) (filtra "offers bounties" vs VDP)
- [Bugcrowd — programs](https://bugcrowd.com/programs) · [Intigriti](https://www.intigriti.com/programs) · [YesWeHack](https://yeswehack.com/programs)
- [disclose.io](https://disclose.io) — diretório de VDPs + Safe Harbor
- [awesome-google-vrp-writeups](https://github.com/xdavidhu/awesome-google-vrp-writeups) — estude **como** entraram no VRP

> **Fluxo pro blog:** pega um VDP de recognition (NASA/gov/edu) → acha um bug de **authz/lógica** (a sua praia) → reporta → espera o disclosure/permissão → escreve o **"How I…"**. Primeiro o HoF fácil; Google/Apple vêm quando você já tiver repertório. Como caçar a superfície certa nesses alvos grandes: **[recon-big-tech.md](recon-big-tech.md)**.

---

⬅️ [ROADMAP MINIMALISTA](ROADMAP-MINIMALISTA.md) · [🏠 Início](README.md)
