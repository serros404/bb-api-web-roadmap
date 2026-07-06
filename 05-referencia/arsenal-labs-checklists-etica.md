# 05 — Arsenal · Labs · Reading List · Checklist · Ética · Anti-padrões

[🏠 Início](../README.md) › [05 · Referência](README.md) › Arsenal · Labs · Checklists · Ética

**Sumário:**
- [🧰 Arsenal completo](#-arsenal-completo)
- [🧪 Laboratórios recomendados](#-laboratórios-recomendados)
- [📚 Reading list](#-reading-list)
- [✅ Checklist final de qualidade](#-checklist-final-de-qualidade)
- [⚖️ Ética e compromissos](#-ética-e-compromissos)
- [🚫 Anti-padrões — o que NÃO fazer](#-anti-padrões--o-que-não-fazer)

---

> O arquivo de consulta permanente. Releia a Ética e os Anti-padrões de tempos em tempos — é fácil cair em vários sem perceber.

---

## 🧰 Arsenal completo

Organizado por função. **Domine o core antes de colecionar o exótico** — Burp + DevTools + cabeça acham mais bug que qualquer arsenal.

### Core (a mão direita)
| Ferramenta | Uso |
|---|---|
| [Burp Suite](https://portswigger.net/burp) | Proxy/Repeater/Intruder — o instrumento da profundidade |
| [Caido](https://caido.io) | Proxy moderno e leve; ótimo complemento/alternativa ao Burp |
| DevTools (Chrome/Firefox) | Network/Sources/Console/Debugger — sua lupa de front-end |

### Burp — extensões essenciais (BApp Store)
| Extensão | Uso |
|---|---|
| Autorize | Testa autorização (IDOR/BOLA/BAC) com duas sessões |
| Param Miner | Descobre headers/params escondidos |
| Turbo Intruder | Race conditions (single-packet attack) |
| JWT Editor | Manipula e ataca JWT |
| InQL | GraphQL no Burp (introspection, queries) |
| Logger++ | Log avançado e filtragem |
| Collaborator | OOB para SSRF/XXE/RCE blind |

### Descoberta de domínios & ASN
[amass](https://github.com/owasp-amass/amass) · [asnmap](https://github.com/projectdiscovery/asnmap) · [bgp.he.net](https://bgp.he.net) · [whoxy](https://www.whoxy.com) · [crt.sh](https://crt.sh)

### Subdomínios (passiva / ativa / permutação)
[subfinder](https://github.com/projectdiscovery/subfinder) · [amass](https://github.com/owasp-amass/amass) · [assetfinder](https://github.com/tomnomnom/assetfinder) · [findomain](https://github.com/Findomain/Findomain) · [chaos](https://github.com/projectdiscovery/chaos-client) · [github-subdomains](https://github.com/gwen001/github-subdomains) · [puredns](https://github.com/d3mondev/puredns) · [massdns](https://github.com/blechschmidt/massdns) · [shuffledns](https://github.com/projectdiscovery/shuffledns) · [dnsx](https://github.com/projectdiscovery/dnsx) · [gotator](https://github.com/Josue87/gotator) · [alterx](https://github.com/projectdiscovery/alterx) · [dnsgen](https://github.com/AlephNullSK/dnsgen)

### Probing / portas / fingerprint
[httpx](https://github.com/projectdiscovery/httpx) · [naabu](https://github.com/projectdiscovery/naabu) · [masscan](https://github.com/robertdavidgraham/masscan) · [nmap](https://nmap.org) · [whatweb](https://github.com/urbanadventurer/WhatWeb) · [fingerprintx](https://github.com/praetorian-inc/fingerprintx)

### Triagem visual
[gowitness](https://github.com/sensepost/gowitness) · [aquatone](https://github.com/michenriksen/aquatone) · [eyewitness](https://github.com/RedSiege/EyeWitness)

### Conteúdo & endpoints (crawl / histórico / fuzz)
[katana](https://github.com/projectdiscovery/katana) · [gau](https://github.com/lc/gau) · [waymore](https://github.com/xnl-h4ck3r/waymore) · [gospider](https://github.com/jaeles-project/gospider) · [hakrawler](https://github.com/hakluke/hakrawler) · [ffuf](https://github.com/ffuf/ffuf) · [feroxbuster](https://github.com/epi052/feroxbuster) · [dirsearch](https://github.com/maurosoria/dirsearch)

### Análise de JS & parâmetros & segredos
[getJS](https://github.com/003random/getJS) · [LinkFinder](https://github.com/GerbenJavado/LinkFinder) · [JSluice](https://github.com/BishopFox/jsluice) · [SecretFinder](https://github.com/m4ll0k/SecretFinder) · [Mantra](https://github.com/MrEmpy/mantra) · [trufflehog](https://github.com/trufflesecurity/trufflehog) · [arjun](https://github.com/s0md3v/Arjun) · [x8](https://github.com/Sh1Yo/x8) · [paramspider](https://github.com/devanshbatham/ParamSpider)

### API & GraphQL
[kiterunner](https://github.com/assetnote/kiterunner) · [InQL](https://github.com/doyensec/inql) · [graphql-cop](https://github.com/dolevf/graphql-cop) · [graphw00f](https://github.com/dolevf/graphw00f) · [GraphQL Voyager](https://github.com/IvanGoncharov/graphql-voyager)

### Auth / JWT
[jwt_tool](https://github.com/ticarpi/jwt_tool) · JWT Editor (Burp) · [jwt.io](https://jwt.io)

### Templates / scanning (uso disciplinado)
[nuclei](https://github.com/projectdiscovery/nuclei) (+ templates customizados gerados com IA)

### Processamento de pipeline (o "encanamento")
[anew](https://github.com/tomnomnom/anew) · [qsreplace](https://github.com/tomnomnom/qsreplace) · [gf](https://github.com/tomnomnom/gf) + [gf-patterns](https://github.com/1ndianl33t/Gf-Patterns) · [uro](https://github.com/s0md3v/uro) · [unfurl](https://github.com/tomnomnom/unfurl) · [jq](https://stedolan.github.io/jq/)

### Payloads & referência
[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) · [SecLists](https://github.com/danielmiessler/SecLists) · [HackTricks](https://book.hacktricks.xyz) · [DOM XSS Wiki](https://github.com/wisec/domxsswiki/wiki)

### IA (a camada de inteligência — [motor de recon](../01-recon/recon-engine-ia-e-automacao.md))
Claude / GPT (raciocínio, JS, scripts, code review, report) · [Ollama](https://ollama.com) / LM Studio (modelos **locais** para dados sensíveis) · API Anthropic (triagem/sumarização automatizada em pipeline)

> **Nota sobre scanners:** Nuclei e afins são **detectores de pista**, não geradores de report. A linha é sempre: máquina aponta → **humano valida e explora**.

---

## 🧪 Laboratórios recomendados

Sequência sagrada: **lab → writeup → caça autorizada**.

### Treino guiado por classe de bug
| Lab | Por que |
|---|---|
| [PortSwigger Web Security Academy](https://portswigger.net/web-security) | A referência. Faça todos Apprentice + Practitioner. Grátis. |
| [PentesterLab](https://pentesterlab.com) | Exercícios isolados por tema, com badges |
| [Root-Me](https://www.root-me.org) | Desafios web curtos e variados |
| [Hacker101 + CTF](https://www.hacker101.com) | Curso da HackerOne + CTF que destrava convites privados |

### Labs locais (Docker) — quebre à vontade
| Lab | Foco | Subir |
|---|---|---|
| [OWASP Juice Shop](https://github.com/juice-shop/juice-shop) | Web moderno (SPA + REST) | `docker run -p 3000:3000 bkimminich/juice-shop` |
| [DVWA](https://github.com/digininja/DVWA) | Clássicos com níveis | `docker run -p 80:80 vulnerables/web-dvwa` |
| [WebGoat](https://owasp.org/www-project-webgoat/) | Lições OWASP | `docker run -p 8080:8080 webgoat/webgoat` |
| [crAPI](https://github.com/OWASP/crAPI) | API vulnerável realista | `docker compose` (repo) |
| [VAmPI](https://github.com/erev0s/VAmPI) | API REST liga/desliga vulns | `docker run -p 5000:5000 erev0s/vampi` |
| [DVGA](https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application) | GraphQL vulnerável | `docker run -p 5013:5013 dolevf/dvga` |

### Plataformas de caça / treino
[BugBountyHunter.com](https://www.bugbountyhunter.com) (zseano) · [Hack The Box](https://www.hackthebox.com) · [TryHackMe](https://tryhackme.com) · [APIsec University](https://academy.apisec.ai/api-pentesting-fundamentals) · [PortSwigger BSCP](https://portswigger.net/web-security/certification) (a "prova final" de F2)

---

## 📚 Reading list

### Blogs / research (assine o RSS)
[PortSwigger Research](https://portswigger.net/research) · [Orange Tsai (blog.orange.tw)](https://blog.orange.tw) · [DEVCORE](https://devco.re/blog/) · [Assetnote](https://blog.assetnote.io) · [Doyensec](https://blog.doyensec.com) · [intigriti](https://blog.intigriti.com) · [Bugcrowd](https://blog.bugcrowd.com) · [watchTowr](https://watchtowr.com/blog/) · [SonarSource](https://www.sonarsource.com/blog/) · [Snyk](https://snyk.io/blog/) · [Sam Curry](https://samcurry.net) · [Krzysztof Kotowicz](https://blog.kotowicz.net) · [Securitum research](https://research.securitum.com/postmessage-xss/)

> 🧭 **Panteão completo de pesquisadores** (nome · foco · onde seguir), curado e verificável, na [lente Orange Tsai — seção 4](../03-playbooks/orange-tsai-decodificado-js.md#4--panteão-de-pesquisadores-de-web-security-seguir-ler-imitar). Meta-fonte anual: **"Top 10 Web Hacking Techniques"** da PortSwigger.

### Curadorias / feeds
[Pentester.land](https://pentester.land/list-of-bug-bounty-writeups.html) · [InfoSec Writeups](https://infosecwriteups.com) · [HackerOne Hacktivity](https://hackerone.com/hacktivity) · [tl;dr sec](https://tldrsec.com)

### Referência técnica
[OWASP API Security](https://owasp.org/API-Security/) · [OWASP WSTG](https://owasp.org/www-project-web-security-testing-guide/) · [HackTricks](https://book.hacktricks.xyz) · [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) · [MDN HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)

### JS / Node — pesquisa ofensiva (o pivô 2026)
A base da [lente Orange Tsai/JS](../03-playbooks/orange-tsai-decodificado-js.md). Fontes primárias:
- **Prototype pollution:** [origem (Arteau, NorthSec 2018)](https://github.com/HoLyVieR/prototype-pollution-nsec18) · [SSPP→RCE — Silent Spring (USENIX 2023)](https://arxiv.org/abs/2207.11171) · [gadgets (KTH-LangSec)](https://github.com/KTH-LangSec/server-side-prototype-pollution) · [SSPP black-box (PortSwigger)](https://portswigger.net/web-security/prototype-pollution/server-side).
- **Confusão de parser/rota:** [Next.js middleware bypass CVE-2025-29927 (Vercel postmortem)](https://vercel.com/blog/postmortem-on-next-js-middleware-bypass) · [Confusion Attacks / Apache (Orange)](https://blog.orange.tw/posts/2024-08-confusion-attacks-en/).
- **Sanitizer/mXSS:** [Bypassing DOMPurify again (PortSwigger)](https://portswigger.net/research/bypassing-dompurify-again-with-mutation-xss).
- **Sandbox/deser:** [vm2 sandbox escape (advisory)](https://github.com/advisories/GHSA-cchq-frgv-rjh5) · node-serialize CVE-2017-5941.
- **Supply chain:** [Snyk](https://snyk.io/blog/) · [Socket.dev](https://socket.dev) · [OWASP npm security](https://cheatsheetseries.owasp.org/cheatsheets/NPM_Security_Cheat_Sheet.html).

### Livros
- *The Web Application Hacker's Handbook* (Stuttard & Pinto) — base clássica.
- *Real-World Bug Hunting* (Peter Yaworski) — mentalidade, bugs reais.
- *Hacking APIs* (Corey J. Ball) — a referência prática de API.
- *Bug Bounty Bootcamp* (Vickie Li) — panorama moderno.
- *Alice and Bob Learn Application Security* (Tanya Janca) — fundamentos AppSec.

### Fundamentos (F1)
[JavaScript.info](https://javascript.info) · [Eloquent JavaScript](https://eloquentjavascript.net/) · [MDN JS](https://developer.mozilla.org/en-US/docs/Web/JavaScript) · [Automate the Boring Stuff](https://automatetheboringstuff.com/) · [SQLBolt](https://sqlbolt.com) · [web.dev](https://web.dev/learn)

### YouTube / podcasts
[NahamSec](https://www.youtube.com/c/NahamSec) · [STÖK](https://www.youtube.com/c/STOKfredrik) · [InsiderPhD](https://www.youtube.com/c/InsiderPhD) · [PinkDraconian](https://www.youtube.com/c/PinkDraconian) · [Intigriti](https://www.youtube.com/c/Intigriti) · [Hussein Nasser](https://www.youtube.com/c/HusseinNasser-software-engineering) · **Podcast:** *Critical Thinking – Bug Bounty Podcast* (Justin Gardner / Rhynorater)

### Comunidades / plataformas BB
[HackerOne](https://hackerone.com) · [Bugcrowd](https://bugcrowd.com) · [Intigriti](https://intigriti.com) · [YesWeHack](https://yeswehack.com) · [Synack](https://www.synack.com) (convite) · [huntr](https://huntr.com) (OSS) · [Patchstack](https://patchstack.com) (WordPress)

---

## ✅ Checklist final de qualidade

Você "concluiu" este roadmap quando consegue marcar, com honestidade, a maioria destes. É sobre **capacidade demonstrada**, não tempo decorrido.

### Pipeline & automação (a base 2026)
- [ ] Roda o funil de recon completo (domínio → endpoints) de forma automatizada
- [ ] Usa IA para deobfuscar JS, gerar wordlists contextuais e priorizar superfície
- [ ] Sabe a linha: máquina aponta, humano valida — nunca manda output bruto

### Técnico — web
- [ ] Todos os labs Apprentice + Practitioner da PortSwigger feitos
- [ ] Traça DOM XSS source-to-sink lendo JS
- [ ] Acha IDOR/BOLA e BAC/BFLA manualmente (Autorize)
- [ ] Reproduziu race condition (single-packet/Turbo Intruder)
- [ ] Domina ≥1 vetor client-side avançado ponta a ponta
- [ ] Sabe testar SSRF, SQLi, XXE, deser, smuggling

### Técnico — API
- [ ] OWASP API Top 10 (2023) de cor e testável
- [ ] crAPI + VAmPI + DVGA fechados
- [ ] Introspection + batching/aliases em GraphQL
- [ ] Detecta mass assignment (BOPLA) e BOLA/BFLA em REST

### Resultados concretos
- [ ] 1º bug aceito (triado/pago) em programa real
- [ ] 1º CVE próprio publicado
- [ ] Repo `ctf-writeups` ativo (≥3 CTFs)
- [ ] Pipeline pública (blog + perfil + GitHub) funcionando
- [ ] Pelo menos um writeup seu citado/curtido pela comunidade

### Carreira / sustentabilidade
- [ ] Currículo + LinkedIn refletem portfolio real
- [ ] 1ª conversa real de oportunidade
- [ ] Clareza sobre qual das 4 rotas perseguir
- [ ] Cadência semanal sustentável (sem maratona)

---

## ⚖️ Ética e compromissos

Não é burocracia — é o que separa pesquisador de criminoso, legal e moralmente.

### Regras inegociáveis
1. **Escopo é lei.** Só testa o que o programa autoriza, exatamente como autoriza. Sem programa/contrato, não há autorização.
2. **Lab para aprender, escopo para aplicar.** Toda técnica destrutiva (fuzzing pesado, payloads que corrompem dados) se aprende e ensaia em **labs locais**.
3. **Automação ≠ DoS.** Recon massivo respeita rate limit, evita horário de pico, **nunca** derruba o alvo. DNS brute contra resolvers públicos é OK; flood contra a origem **não**.
4. **IA não recebe dados sensíveis de alvo.** Nunca cole PII, credenciais, tokens vivos ou código sob NDA em modelos de terceiros — use modelo **local** ou não use. Considere a retenção do provedor.
5. **PoC de impacto mínimo.** Provou um IDOR? Mostre **um** registro que não é seu e pare. **Nunca** exfiltre dados de terceiros em massa, não pivote, não escale além da prova.
6. **Disclosure coordenado.** Reporte primeiro, divulgue depois — e só **após a correção** (ou prazo combinado).
7. **Dados de terceiros são sagrados.** Topou com PII? Não copie, não guarde, não compartilhe. Documente o mínimo e siga.
8. **Não é porque dá que deve.** Acesso técnico **não** é permissão. A linha é a **autorização** — nunca sua habilidade.

### Nota legal (leia de verdade)
Acesso não autorizado é **crime**, e habilidade técnica não é defesa.
- 🇧🇷 **Brasil:** **Lei 12.737/2012** ("Carolina Dieckmann") tipifica invasão de dispositivo informático; **LGPD** rege dados pessoais.
- 🇺🇸 **EUA:** **CFAA** criminaliza acesso "sem autorização ou excedendo a autorização".
- 🇬🇧 **Reino Unido:** **Computer Misuse Act 1990**.

> Programas de BB e contratos de pentest são uma **autorização explícita e limitada** — definem *o que*, *como* e *até onde*. Fora do perímetro, a lei não pergunta se sua intenção era boa. Na dúvida sobre escopo, **pergunte ao programa antes de agir** e siga o *Safe Harbor* quando existir. Tudo aqui é para uso **autorizado**; qualquer uso fora disso é responsabilidade exclusiva de quem praticar.

---

## 🚫 Anti-padrões — o que NÃO fazer

A lista de erros que mais trava (ou queima) caçadores — atualizada para a era da automação + IA.

1. **Parar na automação ("automation runner").** Mandar output bruto de Nuclei/scanner sem validar → duplicata, N/A, reputação queimada. Máquina aponta, **humano valida e explora**.
2. **Automatizar sem entender.** Rodar um pipeline que você não compreende. Se não sabe o que `httpx`/`nuclei`/`katana` fazem por baixo, você não controla o resultado.
3. **Confiar na IA cegamente.** Alucinação é real. Endpoint, CVE ou "fato" que a IA deu é **hipótese até confirmar**. IA acelera o caçador competente; não cria competência.
4. **Vazar dados de alvo para IA.** Colar PII/credenciais/código sob NDA em modelo de terceiro. Use modelo local ou não use.
5. **Não ler writeups.** Sem os 3–5/semana, você estagna e reinventa a roda.
6. **Pular labs e ir direto a produção.** Você só queima escopo, gera report ruim e arrisca causar dano. Lab primeiro, sempre.
7. **Testar fora de escopo.** O pecado capital. Mata conta, reputação e pode virar caso legal.
8. **Caçar só "critical".** Ignorar low/medium. Volume de bugs sólidos paga contas e abre programas privados.
9. **Copiar payload sem entender.** Colar de PayloadsAllTheThings sem saber por que funciona te trava quando ele falha — e ele vai falhar.
10. **Maratonar até o burnout.** BB é maratona. Cadência sustentável > heroísmo de fim de semana.
11. **Colecionar ferramentas em vez de dominar o core.** 50 tools mal usadas perdem para Burp + DevTools + cabeça bem usados.
12. **Não documentar.** Achar um padrão e esquecer. Seu "segundo cérebro" (`notes`) é vantagem competitiva.
13. **Divulgar antes do fix.** Postar por ego/pressa expõe usuários e queima o programa.
14. **Recon eterno sem testar.** Meses montando pipeline e nunca abrindo o Burp para *olhar* a aplicação. Recon serve à caça, não a substitui.
15. **Ignorar a lógica de negócio.** Procurar só payload mágico. As logic flaws e IDOR/BOLA — o dinheiro de verdade — vivem na pergunta *"o que essa função deveria impedir, e como quebro essa regra?"*.

---

> **Última palavra.** Este roadmap é mapa, não trilho. Adapte os prazos; não negocie os princípios: **a máquina coleta em escala, você valida e explora com profundidade** — e tudo dentro do escopo. A diferença entre quem lê isto e quem vira caçador de verdade é uma só: **fazer**. Suba um lab, rode seu primeiro funil de recon em um alvo autorizado e leia um writeup hoje. Boa caçada. 🎯

---

⬅️ [Anterior: CTF · CVE · Carreira (F4–F7)](../02-trilha/03-ctf-cve-carreira-f4-f7.md) · [⬆️ Topo](#) · [Próximo: Playbook de Classes de Bug ➡️](../03-playbooks/classes-de-bug.md)
