# ⭐ 01 — Recon Engine: IA + Automação (o motor de 2026)

[🏠 Início](../README.md) › [01 · Recon](README.md) › Recon Engine: IA + Automação

**Sumário:**
- [🎛️ Filosofia do recon: o funil](#-filosofia-do-recon-o-funil)
- [1️⃣ Descoberta de domínios & ASN](#1-descoberta-de-domínios--asn)
- [2️⃣ Enumeração de subdomínios](#2-enumeração-de-subdomínios) — [2.1 Passiva](#21-passiva-não-toca-no-alvo) · [2.2 Ativa / bruteforce](#22-ativa--bruteforce-resolve-contra-dns) · [2.3 Permutações](#23-permutações-variações-de-nomes-encontrados)
- [3️⃣ Resolução DNS & probing HTTP](#3-resolução-dns--probing-http)
- [4️⃣ Fingerprinting & triagem visual](#4-fingerprinting--triagem-visual)
- [5️⃣ Descoberta de conteúdo & endpoints](#5-descoberta-de-conteúdo--endpoints)
- [6️⃣ Análise de JavaScript & descoberta de parâmetros](#6-análise-de-javascript--descoberta-de-parâmetros)
- [7️⃣ Scanning de templates (Nuclei)](#7-scanning-de-templates-nuclei--o-limite-saudável-da-automação)
- [🤖 8️⃣ IA aplicada ao recon e à caça](#-8-ia-aplicada-ao-recon-e-à-caça-o-diferencial-de-2026) — [8.1 Modelos](#81-modelos-e-onde-rodam) · [8.2 Casos de uso](#82-casos-de-uso-de-alto-valor) · [8.3 Padrões de prompt](#83-padrões-de-prompt-para-trabalho-de-segurança) · [8.4 Pipeline de exemplo](#84-um-pipeline-de-exemplo-ponta-a-ponta) · [8.5 Limites e riscos](#85--limites-e-riscos-da-ia-não-pule-isto)

---

> **Este é o arquivo mais importante da edição 2026.** Aqui mora a **Fase A** da metodologia: coleta em escala. A regra que atravessa tudo: **a máquina mapeia a superfície; você (humano) valida e explora.** Releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase) antes de apontar qualquer ferramenta para qualquer alvo.

---

## 🎛️ Filosofia do recon: o funil

Recon não é "rodar ferramenta". É um **funil** que parte do nome de uma empresa e termina em uma lista priorizada de coisas para um humano olhar. Cada estágio reduz ruído e aumenta sinal.

```
ESCOPO / EMPRESA
   │
   ├─▶ Descoberta de domínios & ASN        (qual é o império do alvo?)
   │
   ├─▶ Enumeração de subdomínios            (passiva + ativa + permutações)
   │
   ├─▶ Resolução DNS & probing HTTP         (o que está vivo, em que porta?)
   │
   ├─▶ Fingerprinting & triagem visual      (que tecnologia? o que parece interessante?)
   │
   ├─▶ Descoberta de conteúdo & endpoints   (crawling + fuzzing + JS)
   │
   ├─▶ Análise de JS & descoberta de params (segredos, rotas de API, sinks)
   │
   └─▶ PRIORIZAÇÃO (IA + filtros)  ─────────▶  ENTREGA AO HUMANO (Fase B)
```

> ⚠️ **Ética do recon automatizado:** subdomain bruteforce e fuzzing geram **muito** tráfego. Respeite `rate-limit`, evite horários de pico, **nunca** rode fuzzing destrutivo em produção, e fique **estritamente** dentro do escopo. DNS bruteforce contra resolvers públicos é OK; flood contra a origem do alvo **não é**.

---

## 1️⃣ Descoberta de domínios & ASN

Antes de subdomínios, descubra **todos os domínios e blocos de IP** que pertencem ao alvo. Programas grandes têm aquisições, marcas secundárias e ranges inteiros.

| Ferramenta | O que faz |
|---|---|
| [amass](https://github.com/owasp-amass/amass) (`amass intel`) | Inteligência de domínios por organização, ASN e reverse whois |
| [asnmap](https://github.com/projectdiscovery/asnmap) | ASN → ranges de IP (CIDR) de uma organização |
| [bgp.he.net](https://bgp.he.net) | Busca manual de ASN/prefixos por nome de empresa |
| [whoxy](https://www.whoxy.com) / reverse whois | Domínios registrados pelo mesmo e-mail/organização |
| [crt.sh](https://crt.sh) | Certificate Transparency — domínios e subdomínios em certificados |

```bash
# ASN e ranges de IP de uma organização
asnmap -org "ExemploCorp" -silent | tee asn_ranges.txt

# Inteligência de domínios via amass
amass intel -org "ExemploCorp" -o intel_domains.txt
amass intel -asn 13335 -o asn_domains.txt

# Domínios em Certificate Transparency (crt.sh) via curl + jq
curl -s "https://crt.sh/?q=%25.exemplo.com&output=json" \
  | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u | tee ct_domains.txt
```

> 🎯 **Saída deste estágio:** uma lista de **domínios raiz** e **ranges de IP** confirmadamente do alvo (cruze sempre com o escopo do programa). Tudo daqui pra frente parte disso.

---

## 2️⃣ Enumeração de subdomínios

Três camadas que se complementam: **passiva** (fontes públicas, sem tocar no alvo), **ativa/bruteforce** (resolve nomes contra DNS) e **permutações** (gera variações de nomes já encontrados).

### 2.1 Passiva (não toca no alvo)

| Ferramenta | Observação |
|---|---|
| [subfinder](https://github.com/projectdiscovery/subfinder) | O padrão de mercado; agrega dezenas de fontes (configure API keys!) |
| [amass](https://github.com/owasp-amass/amass) (`enum -passive`) | Profundo; ótimo combinado com subfinder |
| [assetfinder](https://github.com/tomnomnom/assetfinder) | Rápido e simples (tomnomnom) |
| [findomain](https://github.com/Findomain/Findomain) | Veloz, multiplataforma |
| [chaos](https://github.com/projectdiscovery/chaos-client) | Dataset de subdomínios da ProjectDiscovery (free key) |
| [github-subdomains](https://github.com/gwen001/github-subdomains) | Garimpa subdomínios em código no GitHub |
| [gau](https://github.com/lc/gau) / [waymore](https://github.com/xnl-h4ck3r/waymore) | URLs históricas (Wayback/CommonCrawl) — extrai subdomínios também |

```bash
# Coleta passiva multi-fonte e consolidação
subfinder -d exemplo.com -all -silent -o subs_subfinder.txt
amass enum -passive -d exemplo.com -o subs_amass.txt
assetfinder --subs-only exemplo.com > subs_assetfinder.txt
chaos -d exemplo.com -silent > subs_chaos.txt

# Junta tudo, deduplica (anew mantém só novidades em arquivos incrementais)
cat subs_*.txt | sort -u | tee all_subs_passive.txt | wc -l
```

> 🔑 **Configure suas API keys** em `subfinder` (`~/.config/subfinder/provider-config.yaml`): Censys, SecurityTrails, Shodan, VirusTotal, GitHub, etc. Sem keys, a coleta passiva fica anêmica.

### 2.2 Ativa / bruteforce (resolve contra DNS)

| Ferramenta | Função |
|---|---|
| [puredns](https://github.com/d3mondev/puredns) | Bruteforce + resolução em massa com validação anti-wildcard |
| [massdns](https://github.com/blechschmidt/massdns) | Resolver DNS de altíssima vazão (backend do puredns) |
| [shuffledns](https://github.com/projectdiscovery/shuffledns) | Wrapper do massdns (bruteforce + resolve) |
| [dnsx](https://github.com/projectdiscovery/dnsx) | Toolkit DNS: resolve, filtra wildcard, tipos de registro |

```bash
# Bruteforce de subdomínios com wordlist + resolvers confiáveis
puredns bruteforce wordlist-subdominios.txt exemplo.com \
  -r resolvers.txt -w subs_bruteforce.txt

# Resolver uma lista grande e ficar só com os que respondem
cat all_subs.txt | dnsx -silent -a -resp-only | sort -u > subs_resolvidos.txt
```

> Wordlists de subdomínio boas: `SecLists/Discovery/DNS/` (ex.: `subdomains-top1million-110000.txt`, `dns-Jhaddix.txt`). Veja IA + wordlists adiante para gerar listas **contextuais ao alvo**.

### 2.3 Permutações (variações de nomes encontrados)

| Ferramenta | Ideia |
|---|---|
| [gotator](https://github.com/Josue87/gotator) | Gera permutações (`api`, `dev`, `stg`, `-v2`…) a partir de subs conhecidos |
| [alterx](https://github.com/projectdiscovery/alterx) | Engine de permutações com padrões customizáveis |
| [dnsgen](https://github.com/AlephNullSK/dnsgen) | Gera nomes candidatos a partir de uma lista |

```bash
# Gera permutações a partir dos subs já achados e resolve
gotator -sub subs_resolvidos.txt -perm palavras.txt -depth 1 -numbers 3 \
  | puredns resolve -r resolvers.txt -w subs_permutados.txt
```

> 🎯 **Saída:** lista única e resolvida de subdomínios (passiva + bruteforce + permutações), pronta para probing.

---

## 3️⃣ Resolução DNS & probing HTTP

Da lista de nomes, descubra **o que está vivo, em que portas, com que status**.

| Ferramenta | Função |
|---|---|
| [httpx](https://github.com/projectdiscovery/httpx) | Probe HTTP/HTTPS: status, título, tech, redirect, CDN |
| [naabu](https://github.com/projectdiscovery/naabu) | Port scan rápido (SYN/CONNECT) |
| [masscan](https://github.com/robertdavidgraham/masscan) | Port scan de internet em escala |
| [nmap](https://nmap.org) | Scan detalhado de serviços/versões (depois de naabu/masscan) |

```bash
# Quais hosts respondem em HTTP/HTTPS, com metadados úteis
cat subs_resolvidos.txt \
  | httpx -silent -status-code -title -tech-detect -web-server -cdn \
          -follow-redirects -o hosts_vivos.txt

# Portas abertas (rápido) e depois detalhe com nmap só no que importa
naabu -list subs_resolvidos.txt -top-ports 1000 -silent -o portas.txt
nmap -sV -sC -iL <(cut -d: -f1 portas.txt | sort -u) -oA nmap_detalhe
```

> ⚠️ Port scanning pesado pode disparar alertas e violar termos. Cheque a política do programa; muitos pedem para **não** fazer scan agressivo de portas.

---

## 4️⃣ Fingerprinting & triagem visual

Agora classifique a superfície: **que tecnologia roda** e **o que parece interessante visualmente** (painéis de login, admin, apps antigos, erros expostos).

| Ferramenta | Função |
|---|---|
| [httpx](https://github.com/projectdiscovery/httpx) `-tech-detect` | Stack por host (Wappalyzer engine embutido) |
| [whatweb](https://github.com/urbanadventurer/WhatWeb) | Fingerprint de tecnologias web |
| [fingerprintx](https://github.com/praetorian-inc/fingerprintx) | Fingerprint de serviços (não só HTTP) |
| [gowitness](https://github.com/sensepost/gowitness) | Screenshots em massa (Chrome headless) |
| [aquatone](https://github.com/michenriksen/aquatone) | Screenshots + clustering visual por similaridade |
| [eyewitness](https://github.com/RedSiege/EyeWitness) | Screenshots + categorização |

```bash
# Screenshots de tudo que está vivo, para triagem visual rápida
cat hosts_vivos.txt | cut -d' ' -f1 | gowitness scan file -f - --screenshot-path shots/

# Abra a galeria e marque visualmente: logins, admin, dev/stg, erros 500, apps legados
```

> 🎯 A triagem visual é onde o humano entra cedo: em 10 minutos olhando screenshots você separa "100 páginas de marketing" de "3 painéis de admin esquecidos" — e é nos 3 que você vai caçar.

---

## 5️⃣ Descoberta de conteúdo & endpoints

Para cada host interessante, encontre **caminhos, arquivos e endpoints**. Combine **crawling** (segue links/JS), **fontes históricas** (Wayback) e **fuzzing** (adivinha caminhos).

| Ferramenta | Tipo |
|---|---|
| [katana](https://github.com/projectdiscovery/katana) | Crawler moderno (headless, parseia JS) |
| [gau](https://github.com/lc/gau) / [waymore](https://github.com/xnl-h4ck3r/waymore) | URLs históricas (Wayback, CommonCrawl, URLScan, AlienVault) |
| [gospider](https://github.com/jaeles-project/gospider) / [hakrawler](https://github.com/hakluke/hakrawler) | Crawlers rápidos |
| [ffuf](https://github.com/ffuf/ffuf) | Fuzzing de diretórios/arquivos/params (o canivete) |
| [feroxbuster](https://github.com/epi052/feroxbuster) | Fuzzing recursivo veloz |
| [dirsearch](https://github.com/maurosoria/dirsearch) | Brute de caminhos com listas próprias |

```bash
# Crawl + histórico, consolidando todas as URLs conhecidas do host
katana -u https://app.exemplo.com -jc -kf all -d 3 -silent -o katana_urls.txt
gau --subs exemplo.com | anew gau_urls.txt
cat katana_urls.txt gau_urls.txt | sort -u > all_urls.txt

# Fuzzing de conteúdo (ajuste -rate ao escopo; nunca martele a origem)
ffuf -u https://app.exemplo.com/FUZZ -w SecLists/Discovery/Web-Content/raft-large-words.txt \
     -mc 200,204,301,302,307,401,403,405 -rate 50 -o ffuf.json
```

> 💡 **Processamento de URLs** (extração de sinal): use [uro](https://github.com/s0md3v/uro) para deduplicar URLs parecidas, [qsreplace](https://github.com/tomnomnom/qsreplace) para trocar valores de parâmetros em massa, [unfurl](https://github.com/tomnomnom/unfurl) para extrair domínios/paths/params, e [gf](https://github.com/tomnomnom/gf) (padrões do [gf-patterns](https://github.com/1ndianl33t/Gf-Patterns)) para grepar URLs candidatas a XSS/SSRF/SQLi/redirect/IDOR.

```bash
# Pipeline de filtragem: tira ruído e separa candidatos por classe de bug
cat all_urls.txt | uro | tee urls_limpas.txt | gf ssrf  > cand_ssrf.txt
cat urls_limpas.txt | gf xss   > cand_xss.txt
cat urls_limpas.txt | gf redirect > cand_redirect.txt
cat urls_limpas.txt | gf idor  > cand_idor.txt
```

---

## 6️⃣ Análise de JavaScript & descoberta de parâmetros

Em apps modernas (SPAs), **o JS é o mapa do tesouro**: rotas de API escondidas, parâmetros, chaves, endpoints internos. Esse estágio é onde **IA brilha** (veja adiante).

| Ferramenta | Função |
|---|---|
| [getJS](https://github.com/003random/getJS) | Coleta todos os arquivos `.js` de um conjunto de URLs |
| [LinkFinder](https://github.com/GerbenJavado/LinkFinder) | Extrai endpoints de arquivos JS via regex |
| [JSluice](https://github.com/BishopFox/jsluice) | Extrai URLs, paths e segredos de JS (Bishop Fox) |
| [SecretFinder](https://github.com/m4ll0k/SecretFinder) | Caça chaves/segredos (API keys, tokens) em JS |
| [Mantra](https://github.com/MrEmpy/mantra) | Segredos em JS/HTML em massa |
| [trufflehog](https://github.com/trufflesecurity/trufflehog) | Segredos em repos/arquivos com verificação |
| [arjun](https://github.com/s0md3v/Arjun) | Descoberta de parâmetros HTTP escondidos |
| [x8](https://github.com/Sh1Yo/x8) | Descoberta de parâmetros (hidden params) veloz |
| [paramspider](https://github.com/devanshbatham/ParamSpider) | Extrai parâmetros de URLs históricas |

```bash
# Junta o JS de todos os hosts e extrai endpoints + segredos
cat hosts_vivos.txt | cut -d' ' -f1 | getJS --complete | sort -u > js_files.txt
cat js_files.txt | while read u; do python3 linkfinder.py -i "$u" -o cli; done | sort -u > js_endpoints.txt
cat js_files.txt | jsluice urls -R | sort -u >> js_endpoints.txt
cat js_files.txt | xargs -I{} python3 SecretFinder.py -i {} -o cli 2>/dev/null

# Descoberta de parâmetros escondidos em um endpoint
arjun -u https://app.exemplo.com/api/v2/profile -m GET,POST
```

> 🎯 **Saída:** lista de endpoints "ocultos" (de API e web), parâmetros não documentados e **possíveis segredos** vazados — todos para **validação manual** (confirme antes de reportar; muitos "segredos" são keys públicas/inertes).

---

## 7️⃣ Scanning de templates (Nuclei) — o limite saudável da automação

[Nuclei](https://github.com/projectdiscovery/nuclei) roda **templates** (YAML) para detectar misconfigs, exposições e CVEs conhecidos em escala. É a peça onde "scanner" entra de forma **disciplinada**: como **detector de pistas**, não como gerador de report.

```bash
# Roda nuclei nos hosts vivos, com severidade e rate controlados
nuclei -list hosts_vivos.txt -severity low,medium,high,critical \
       -rate-limit 50 -o nuclei_findings.txt

# Foco cirúrgico: só exposições/misconfig (menos ruído)
nuclei -list hosts_vivos.txt -tags exposure,misconfig -o nuclei_exposicoes.txt
```

> **Onde fica a linha:** Nuclei te aponta *"isto parece exposto/vulnerável"*. **Você confirma manualmente**, entende o impacto no contexto e só então decide reportar. Output bruto de Nuclei **não** é um report — é uma pista. (Logo abaixo, como usar IA para escrever templates Nuclei sob medida.)

---

## 🤖 8️⃣ IA aplicada ao recon e à caça (o diferencial de 2026)

Aqui está o salto da edição 2026. IA (LLMs) não é gimmick — é uma **camada de inteligência** que acelera coleta, processamento e até a exploração. Mas com regras rígidas (veja "Limites e riscos" no fim).

### 8.1 Modelos e onde rodam

| Opção | Quando usar |
|---|---|
| **Claude / GPT (API ou chat)** | Raciocínio forte: análise de JS, geração de scripts, code review, redação de report |
| **Modelos locais** ([Ollama](https://ollama.com), LM Studio) | Quando você **precisa** processar dados sensíveis sem mandar para fora (roda na sua máquina) |
| **API Anthropic em scripts** | Automatizar triagem/sumarização dentro do seu pipeline (ex.: classificar milhares de URLs) |

> 🔒 **Regra de ouro de IA:** dados sensíveis de alvo (PII real, credenciais, código sob NDA) → **modelo local** ou **não usar IA**. Conceitos, JS público, padrões genéricos → modelo na nuvem é OK.

### 8.2 Casos de uso de alto valor

**a) Deobfuscar e entender JS minificado.** Cole um trecho de JS minificado/ofuscado e peça para a IA explicar o que faz, mapear rotas de API e identificar *sources/sinks* de DOM XSS. Economiza horas de leitura. *(Sempre confira o resultado no código real.)*

**b) Extrair endpoints e parâmetros de JS gigante.** Quando LinkFinder/JSluice deixam dúvida, IA reconstrói a lógica de roteamento e lista endpoints + métodos + parâmetros prováveis. Ótima para mapear superfície de API a partir do front.

**c) Gerar wordlists contextuais ao alvo.** Em vez de só usar listas genéricas, peça à IA wordlists baseadas no domínio do negócio (ex.: fintech → `kyc`, `ledger`, `payout`, `chargeback`, `webhook`, `settlement`). Listas contextuais acham caminhos que `raft` nunca acha.

**d) Resumir e priorizar output massivo de recon.** Jogue milhares de linhas de `httpx`/`nuclei`/URLs e peça: *"agrupe por tecnologia, destaque painéis de admin, APIs versionadas antigas e qualquer coisa que cheire a debug/staging; priorize o que olhar primeiro e por quê."* Transforma um paredão de texto em um plano de ataque.

**e) Escrever scripts de recon sob medida.** "Me faça um script Python que lê `hosts_vivos.txt`, bate em `/api/v1` e `/api/v2`, compara respostas e sinaliza endpoints só presentes na v1 (legados)." IA escreve, você lê, entende e ajusta.

**f) Gerar templates Nuclei customizados.** Descreva uma misconfig/CVE específica e peça o YAML do template Nuclei. Você revisa a lógica do matcher e roda. Multiplica sua capacidade de detecção dirigida.

**g) Triagem inicial de achados.** "Aqui estão 30 saídas de Nuclei — quais são prováveis falsos-positivos, quais merecem validação manual urgente, e qual o provável impacto de cada um?" *(A decisão final é sempre sua.)*

**h) Entender stacks e tecnologias desconhecidas.** Caiu num framework que você nunca viu? IA explica o modelo de auth, padrões comuns de bug daquela stack e onde procurar. Encurta a curva.

**i) Gerar hipóteses de ataque.** Descreva uma funcionalidade ("upload de avatar que gera thumbnail server-side") e peça vetores: SSRF via URL de imagem, RCE via ImageMagick/ffmpeg, path traversal no nome, XXE em SVG, etc. IA é excelente *brainstormer* de superfície.

**j) Code review assistido (turbina a F5 — CVE hunting).** Cole funções de um OSS e peça revisão de segurança: sinks perigosos, validação ausente, fluxos de auth quebrados. Acelera muito achar bug em código alheio. *(Detalhe em [CTF · CVE · Carreira (F4–F7)](../02-trilha/03-ctf-cve-carreira-f4-f7.md).)*

**k) Acelerar PoC e report.** IA ajuda a escrever PoC limpo e a redigir o report (resumo, impacto, passos, remediação) em tom profissional. Você garante a precisão técnica e o **impacto mínimo**.

### 8.3 Padrões de prompt para trabalho de segurança

- **Dê papel e contexto:** *"Você é um pesquisador de AppSec. Estou testando, com autorização, um alvo de bug bounty. Analise este JS público e liste endpoints de API, métodos e parâmetros."*
- **Peça raciocínio, não só resposta:** *"Explique o porquê de cada endpoint ser interessante."*
- **Force estrutura:** *"Responda em tabela: endpoint | método | parâmetros | por que é interessante."*
- **Itere:** comece amplo ("o que essa app faz?"), depois afunile ("onde está a lógica de autorização?").
- **Desconfie e valide:** trate toda saída como **hipótese a confirmar**, nunca como fato.

### 8.4 Um pipeline de exemplo, ponta a ponta

```
1. asnmap + amass intel + crt.sh        →  domínios e ranges do alvo
2. subfinder + amass + chaos + permut.  →  todos os subdomínios
3. dnsx + httpx                          →  hosts vivos + tech + títulos
4. gowitness                             →  screenshots p/ triagem visual (humano marca alvos)
5. katana + gau + ffuf                   →  todas as URLs + caminhos
6. getJS + JSluice + LinkFinder + arjun  →  endpoints ocultos + params + segredos
   └─▶ IA: deobfusca JS, extrai rotas de API, gera wordlist contextual
7. nuclei (low→critical)                 →  pistas de misconfig/CVE
   └─▶ IA: resume tudo, prioriza, aponta os 5 alvos mais promissores e o porquê
═════════════════════ ENTREGA AO HUMANO ═════════════════════
8. Burp + DevTools + cabeça              →  VALIDA e EXPLORA os alvos priorizados
   └─▶ IA copiloto: hipóteses, deobfuscação, PoC, redação do report
9. Disclosure coordenado + writeup       →  reputação e conteúdo
```

### 8.5 ⚠️ Limites e riscos da IA (não pule isto)

- **Alucinação:** IA inventa endpoints, CVEs e "fatos" com confiança. **Tudo** que ela diz é hipótese até você confirmar na ferramenta/no alvo.
- **Vazamento de dados:** **nunca** cole PII real, credenciais, tokens vivos ou código proprietário sob NDA em modelos de terceiros. Use modelo local ou não use. Considere retenção de dados do provedor.
- **Escopo e rate limit valem para código gerado por IA também:** um script que a IA escreveu pode martelar a origem — você é responsável por respeitar limites e escopo.
- **IA não substitui entender o bug:** se você não sabe explicar por que o PoC funciona, você não está pronto para reportá-lo. IA acelera o caçador competente; ela não cria competência.
- **Reports gerados por IA sem revisão = lixo:** triagers reconhecem (e penalizam) report genérico de IA. Use IA para estruturar; a precisão técnica e o impacto são seus.

> **Resumo do arquivo:** a máquina (automação + IA) te entrega a **superfície inteira, priorizada**. A partir daqui é **você**: Burp, DevTools e cabeça. Siga para [02 — Fundação + Web](../02-trilha/01-fundacao-e-web-f0-f2.md), onde essa superfície vira bug.

---

⬅️ [Anterior: Visão geral e metodologia](../README.md) · [⬆️ Topo](#) · [Próximo: Fundação + Web (F0·F1·F2) ➡️](../02-trilha/01-fundacao-e-web-f0-f2.md)
