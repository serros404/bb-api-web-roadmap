# 08 — Apêndices Práticos

[🏠 Início](../README.md) › [05 · Referência](README.md) › Apêndices Práticos

**Sumário:**
- [📖 A — Glossário de termos](#-a--glossário-de-termos)
- [🗓️ B — Plano concreto dos primeiros 90 dias](#-b--plano-concreto-dos-primeiros-90-dias)
- [⏱️ C — Template de cronograma semanal](#-c--template-de-cronograma-semanal)
- [❓ D — FAQ](#-d--faq)
- [📝 E — Modelo de report](#-e--modelo-de-report)
- [✅ F — Checklist de pré-submissão de report](#-f--checklist-de-pré-submissão-de-report)

---

> Glossário, plano dos primeiros 90 dias, cronograma semanal, FAQ e modelo de report. A parte "operacional" que transforma o roadmap em rotina. Releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase) para a ética que atravessa tudo.

---

## 📖 A — Glossário de termos

Os termos que aparecem nos arquivos e no dia a dia da caça. Termos técnicos ficam em inglês (padrão de mercado).

### Recon & automação
- **Recon (reconnaissance):** fase de coleta/mapeamento da superfície de ataque.
- **ASN (Autonomous System Number):** identificador de um conjunto de redes/IPs de uma organização.
- **Subdomain enumeration:** descobrir subdomínios de um domínio (passiva, ativa, permutação).
- **Passive recon:** coleta de fontes públicas, sem tocar no alvo.
- **Active recon:** interações que tocam o alvo (resolução DNS, probing, fuzzing).
- **Permutation/alteration:** gerar variações de nomes conhecidos (`api`, `dev`, `-v2`...).
- **Probing:** verificar o que está vivo e como (status, porta, tech).
- **Fingerprinting:** identificar tecnologia/stack de um host.
- **Content discovery:** descobrir caminhos/arquivos/endpoints (crawl + histórico + fuzz).
- **Fuzzing:** enviar muitas entradas para descobrir comportamento (caminhos, params, payloads).
- **Crawling/spidering:** seguir links/JS para mapear a aplicação.
- **Wordlist:** lista de palavras usada em brute/fuzz (genérica ou contextual).
- **Oneliner:** comando único encadeando ferramentas via pipe.
- **Pipeline:** sequência automatizada de etapas de recon/teste.
- **Diff de recon:** comparar execuções para investigar só o que é novo.
- **OOB (Out-of-Band):** confirmação via canal externo (DNS/HTTP) quando não há retorno direto — ex.: Collaborator.
- **Collaborator:** servidor da PortSwigger para detectar interações OOB (SSRF/XXE/RCE blind).

### Classes de bug (siglas)
- **XSS:** Cross-Site Scripting (reflected/stored/DOM).
- **SQLi:** SQL Injection.
- **SSRF:** Server-Side Request Forgery.
- **XXE:** XML External Entity.
- **SSTI:** Server-Side Template Injection.
- **LFI/RFI:** Local/Remote File Inclusion.
- **IDOR:** Insecure Direct Object Reference.
- **BOLA:** Broken Object Level Authorization (IDOR em terminologia de API).
- **BFLA:** Broken Function Level Authorization.
- **BOPLA:** Broken Object Property Level Authorization (inclui mass assignment + excessive data exposure).
- **BAC:** Broken Access Control (guarda-chuva de autorização).
- **CSRF:** Cross-Site Request Forgery.
- **CSPT:** Client-Side Path Traversal.
- **mXSS:** Mutation XSS.
- **SSPP:** Server-Side Prototype Pollution.
- **RCE:** Remote Code Execution.
- **ATO:** Account Takeover.
- **TOCTOU:** Time-Of-Check to Time-Of-Use (base das race conditions).

### Auth & protocolos
- **JWT:** JSON Web Token (token assinado de 3 partes).
- **OAuth 2.0 / OIDC:** frameworks de autorização/identidade (login social/SSO).
- **SSO:** Single Sign-On.
- **redirect_uri / state / code:** parâmetros do fluxo OAuth.
- **CORS:** Cross-Origin Resource Sharing.
- **CSP:** Content Security Policy.
- **SameSite:** atributo de cookie que mitiga CSRF.
- **Host header poisoning:** abusar do header `Host`/`X-Forwarded-Host` (ex.: link de reset).

### Programa & processo
- **BB (Bug Bounty):** programa que recompensa relatos de vulnerabilidade.
- **VDP (Vulnerability Disclosure Program):** programa sem recompensa financeira (reconhecimento).
- **Scope/escopo:** o que o programa autoriza testar (e como).
- **Safe Harbor:** cláusula que protege legalmente o pesquisador que age dentro das regras.
- **Triage/triagem:** validação inicial do report pela equipe do programa.
- **Duplicate/duplicata:** bug já reportado por outra pessoa (sem recompensa).
- **N/A (Not Applicable):** report considerado sem impacto/válido.
- **Bounty/payout:** recompensa financeira.
- **CVSS:** sistema de pontuação de severidade.
- **PoC (Proof of Concept):** demonstração mínima de que o bug existe.
- **Disclosure coordenado:** reportar primeiro, divulgar publicamente só após o fix.
- **CVE:** identificador público de uma vulnerabilidade conhecida.
- **GHSA:** GitHub Security Advisory.
- **CNA:** entidade que atribui CVEs.

### Ferramentas (apelidos comuns)
- **PD/ProjectDiscovery:** suíte subfinder/httpx/nuclei/katana/dnsx/naabu...
- **Burp/Repeater/Intruder:** o proxy de interceptação e seus módulos.
- **DOM Invader:** módulo do navegador do Burp para DOM XSS/prototype pollution/postMessage.
- **Autorize:** extensão do Burp para testar autorização (IDOR/BAC).
- **Turbo Intruder:** extensão para ataques de alta velocidade (race conditions).

---

## 🗓️ B — Plano concreto dos primeiros 90 dias

Um plano de arranque para ~15h/semana, assumindo que você está na transição F0→F2. **Ajuste à sua realidade** — é trilho de partida, não camisa de força.

### Semanas 1–2 — Fundação de ambiente (F0)
- **Dia 1–3:** instalar Burp/Caido + DevTools; subir Juice Shop e DVWA no Docker.
- **Dia 4–7:** instalar a suíte de recon (`pdtm`, tomnomnom tools); configurar API keys do subfinder; rodar o primeiro `subfinder → dnsx → httpx → gowitness` num alvo VDP autorizado.
- **Dia 8–10:** criar blog + perfis (X/LinkedIn/GitHub); criar repos `recon-scripts`, `ctf-writeups`, `notes`.
- **Dia 11–14:** publicar o 1º post ("meu setup e metodologia"); ler 6–8 writeups e anotar padrões.

> ✅ **Marco:** pipeline mínimo de recon rodando + presença pública no ar.

### Semanas 3–6 — Fundação técnica (F1) + primeiros labs
- **Toda semana:** PortSwigger Academy — comece pelos labs **Apprentice** de Access Control, IDOR, SQLi e XSS.
- **Semana 3–4:** revisar HTTP/headers/cookies/CORS; reler JS de uma SPA real e mapear endpoints.
- **Semana 5–6:** escrever o 1º script de recon próprio (wrapper subfinder→httpx); começar o 2º (parser de JS).
- **Contínuo:** 3–5 writeups/semana; 1 sessão de CTF (picoCTF/Hacker101) a cada 2 semanas.

> ✅ **Marco:** ~15 labs Apprentice fechados + 1–2 scripts no GitHub.

### Semanas 7–10 — Entrada na caça real (início de F2)
- **Semana 7:** escolher 1 programa autorizado de escopo amplo; rodar o funil completo do [motor de recon](../01-recon/recon-engine-ia-e-automacao.md); fazer triagem visual + priorização (com IA) e escolher 3–5 alvos.
- **Semana 8–9:** caçar nos alvos priorizados — foco em **IDOR/BOLA e access control** (use Autorize, duas contas). Labs **Practitioner** de Access Control em paralelo.
- **Semana 10:** aprofundar em uma classe (ex.: DOM XSS lendo JS, ou logic flaws num fluxo de negócio).
- **Contínuo:** writeups + CTF + 3º script.

> ✅ **Marco:** loop híbrido (recon→Burp→validação manual) virou seu fluxo padrão; primeiros candidatos a bug surgindo.

### Semanas 11–12 — Consolidação e primeiro report
- **Semana 11:** lapidar um achado real em report de qualidade (modelo na seção E); PoC de impacto mínimo; revisar com a IA antes de enviar.
- **Semana 12:** enviar o report; escrever um writeup metodológico (sem violar termos); revisar o plano e definir a cadência das próximas semanas.

> ✅ **Marco dos 90 dias:** 1º report enviado (idealmente triado) + portfolio inicial (posts, scripts, notas) consistente.

> **Realismo:** o 1º bug aceito pode levar mais de 90 dias — e tudo bem. O objetivo dos 90 dias é instalar o **sistema** (pipeline + estudo + cadência), não garantir o bug. Quem mantém o sistema, acha bug.

---

## ⏱️ C — Template de cronograma semanal

Um molde para ~15h/semana. Distribua como sua vida permitir; **consistência > intensidade**.

| Bloco | Horas/sem | Atividade |
|---|---|---|
| 🎯 Caça real (Fase A + B) | 6–8h | Recon automatizado + validação/exploração manual em alvo autorizado |
| 🎓 Estudo deliberado | 3–4h | PortSwigger Academy / classe de bug do [playbook de classes de bug](../03-playbooks/classes-de-bug.md) |
| 📖 Leitura de writeups | 2h | 3–5 writeups, com anotação ativa no `notes` |
| 🚩 CTF / 🧬 CVE | 1–2h | Alternar: 1 CTF quinzenal OU code review de OSS |
| ✍️ Conteúdo / pipeline | 1h | Post, atualização de script, organização de notas |

**Exemplo de semana (15h):**
- **Seg (2h):** estudo (1 classe de bug + lab correspondente).
- **Ter (3h):** caça — rodar/revisar recon e atacar 1 alvo priorizado.
- **Qua (2h):** caça — continuar; usar Autorize para IDOR/BAC.
- **Qui (2h):** writeups (1h) + CTF/CVE (1h).
- **Sex (2h):** caça — explorar lógica de negócio de um fluxo.
- **Sáb (3h):** caça profunda + lapidar achados.
- **Dom (1h):** conteúdo + organizar notas + planejar a semana.

> Travou num alvo? Troque de frente (CTF/CVE/conteúdo) — o paralelismo das 4 frentes existe justamente para você nunca ficar 100% bloqueado.

---

## ❓ D — FAQ

**"Preciso saber programar para fazer bug bounty?"**
Para *rodar ferramenta*, não. Para ser bom em 2026, sim — pelo menos ler JS e escrever Python/bash. A metodologia deste roadmap (automação + IA na coleta) pressupõe que você **entende** o que seus scripts e pipelines fazem. Sem isso, você vira "automation runner" ([anti-padrão nº 1 e 2](arsenal-labs-checklists-etica.md#-anti-padrões--o-que-não-fazer)).

**"Usar IA e ferramentas não é 'trapaça' / não me deixa pior caçador?"**
Não — desde que a competência seja sua. IA e automação te dão **amplitude** (mapear tudo, rápido). A **profundidade** (achar logic flaws, IDOR, cadeias) continua 100% sua. O risco não é usar a máquina; é *parar* nela sem validar, ou *confiar* nela sem entender. A máquina aponta; você valida e explora.

**"Por que não focar 100% no manual, como os veteranos faziam?"**
Porque a superfície explodiu. Um programa grande tem milhares de subdomínios e dezenas de milhares de endpoints, e centenas de caçadores competindo. Sem automação na coleta, você nem chega aos alvos certos antes deles serem duplicados. Manual-only em 2026 é deixar 90% da superfície sem olhar.

**"Quanto tempo até o primeiro bug / até viver disso?"**
Primeiro bug aceito: tipicamente meses (3–9), muito variável. Viver de BB: anos de consistência, e nem todo mundo chega (renda é volátil). Por isso o roadmap trata BB como **meio**, com 4 rotas de carreira no F6 — inclusive emprego AppSec e PJ em USD.

**"Burp Community serve ou preciso do Pro?"**
Community serve para começar e aprender. Para caçar a sério, o **Pro** vale (Intruder sem throttle, scanner, salvar projetos). Caido é uma alternativa moderna que muitos usam junto.

**"Posso rodar meus scanners/recon em qualquer site para praticar?"**
**Não.** Só em **labs locais seus** e em **alvos com escopo autorizado** (BB/contrato). Rodar recon/scan em site de terceiro sem autorização é potencialmente crime (Lei 12.737/2012 no Brasil, CFAA nos EUA). Pratique em Juice Shop/DVWA/crAPI e em programas VDP de escopo amplo.

**"O que faço se achar dados pessoais reais (PII) durante um teste?"**
Pare imediatamente. **Não copie, não baixe, não compartilhe.** Documente o mínimo necessário para provar o bug (ex.: "consegui acessar o registro do usuário X, evidenciado por 1 screenshot redigido") e reporte. Exfiltrar PII em massa, mesmo "para provar", é dano real e pode te tirar do Safe Harbor.

**"Quando uso modelo de IA na nuvem vs local?"**
Conceitos, JS público, padrões genéricos, redação → nuvem (Claude/GPT) é OK. Qualquer dado **sensível do alvo** (PII real, credenciais, código sob NDA) → **modelo local** (Ollama/LM Studio) ou não use. Sempre considere a política de retenção do provedor.

**"Devo me especializar logo em web ou em API?"**
Faça os dois — a fronteira entre eles é cada vez mais borrada (toda SPA fala com uma API). Domine web na F2, aprofunde API na F3. A maioria dos melhores caçadores cobre web+API e nunca precisa sair disso.

**"Como evito burnout?"**
Metas de **processo** (horas, alvos olhados, writeups lidos), não só de resultado (bugs/dinheiro); cadência sustentável (nada de maratona de fim de semana); e o paralelismo das 4 frentes para trocar quando travar. Burnout é o fim de carreira nº 1 em BB ([F7 — Consolidação](../02-trilha/03-ctf-cve-carreira-f4-f7.md#-f7--consolidação-contínuo)).

**"Vale a pena tirar a BSCP?"**
Sim, como meta de fim de F2. É barata, prática (você hackeia de verdade na prova) e prova domínio de web. Não substitui caçar de verdade, mas valida sua base e ajuda no F6 (carreira).

---

## 📝 E — Modelo de report

Um bom report aumenta a chance de aceite e o valor da recompensa. Triagers leem dezenas por dia — clareza e impacto bem demonstrado fazem a diferença. Estrutura recomendada:

```markdown
## Título
[Classe] em [endpoint/funcionalidade] permite [impacto conciso]
Ex.: IDOR em GET /api/v1/invoices/{id} permite ler faturas de outros usuários

## Resumo (1 parágrafo)
Descreva, em linguagem direta, o que é o bug e por que importa.
Um triager deve entender o problema só lendo isto.

## Severidade & Impacto
- Severidade sugerida (CVSS, se o programa usar): [ex.: High / 7.5]
- Impacto de negócio concreto: o que um atacante consegue?
  (ex.: "ler dados financeiros de qualquer usuário enumerando IDs sequenciais")

## Passos para reproduzir
1. Autentique-se como Usuário A (conta de teste fornecida/criada).
2. Capture o request GET /api/v1/invoices/1001 (fatura do próprio A).
3. Troque o ID para 1002 (pertencente ao Usuário B).
4. Observe que a resposta retorna a fatura completa de B (status 200).
[inclua requests/responses relevantes em blocos de código — redija PII]

## Prova de Conceito (PoC)
[Comando/captura mínima que evidencia o bug, com impacto mínimo.
Mostre UM registro alheio, redigido — nunca um dump em massa.]

## Remediação sugerida
Valide que o objeto solicitado pertence ao usuário autenticado
(checagem de ownership server-side em todos os endpoints de objeto).

## Referências
- OWASP API Security Top 10 (2023) — API1: BOLA
- [links relevantes]
```

### Princípios de um bom report
- **Impacto > severidade nominal:** mostre o dano real ao negócio, não só a categoria.
- **Reprodutível por qualquer um:** passos numerados, contas de teste, requests completos.
- **Impacto mínimo na PoC:** prove com o menor dano possível; redija qualquer PID/PII.
- **Sem ruído:** um bug por report (salvo cadeia que só faz sentido junta); nada de output bruto de scanner.
- **Tom profissional:** objetivo, sem hype, sem ameaça, sem "me paguem bem". A IA ajuda a redigir — você garante a precisão técnica.
- **Respeite o disclosure:** nada público até o fix (ou prazo combinado).

---

## ✅ F — Checklist de pré-submissão de report

Antes de clicar em "enviar", passe por esta lista:

- [ ] O bug está **dentro do escopo** do programa (revisei a política)?
- [ ] **Reproduzi do zero** seguindo meus próprios passos (funciona limpo)?
- [ ] A **PoC tem impacto mínimo** (nenhum dano real, nenhum dump de PII)?
- [ ] **Redigi** qualquer dado pessoal/sensível nas evidências?
- [ ] O **impacto de negócio** está claro e é convincente?
- [ ] Os **passos** são numerados e reproduzíveis por um terceiro?
- [ ] Incluí **requests/responses** relevantes (não a sessão inteira)?
- [ ] **Não é duplicata óbvia** (pesquisei disclosures públicos do programa)?
- [ ] **Não estou mandando output bruto** de scanner como se fosse report?
- [ ] A **severidade** sugerida é honesta (nem inflada, nem subestimada)?
- [ ] O **tom** é profissional e objetivo?
- [ ] **Validei tecnicamente** tudo (nada de afirmação que veio só da IA sem confirmar)?
- [ ] Sei **explicar o bug** se o triager perguntar (entendo por que funciona)?

> Se você marcou todos com honestidade, está pronto. Um report assim respeita o programa, te diferencia da enxurrada de submissões ruins, e constrói a reputação que abre programas privados e portas de carreira.

---

> **Fim do conjunto.** Você tem agora: a metodologia 2026 ([README](../README.md)), o motor de recon com IA + automação ([01](../01-recon/recon-engine-ia-e-automacao.md)), as fases F0–F7 ([02](../02-trilha/01-fundacao-e-web-f0-f2.md)–[04](../02-trilha/03-ctf-cve-carreira-f4-f7.md)), a referência completa ([05](arsenal-labs-checklists-etica.md)), o playbook de cada classe de bug ([06](../03-playbooks/classes-de-bug.md)), o recon avançado/contínuo + API ([07](../01-recon/recon-continuo-e-api-avancado.md)) e estes apêndices (este). O mapa está completo. O que falta é só uma coisa — **fazer**. Sobe um lab, roda teu primeiro funil num alvo autorizado, lê um writeup hoje. Boa caçada. 🎯

---

⬅️ [Anterior: Recon Contínuo + API Avançado](../01-recon/recon-continuo-e-api-avancado.md) · [⬆️ Topo](#) · [Próximo: Cenários Práticos ➡️](../04-pratica/cenarios-praticos.md)
