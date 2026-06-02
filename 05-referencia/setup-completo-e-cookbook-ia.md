# 10 — Setup Completo do Ambiente + Cookbook de Prompts de IA

[🏠 Início](../README.md) › [05 · Referência](README.md) › Setup Completo + Cookbook de IA

**Sumário:**
- [PARTE I — Setup completo do ambiente](#parte-i--setup-completo-do-ambiente)
- [PARTE II — Cookbook de prompts de IA para segurança](#parte-ii--cookbook-de-prompts-de-ia-para-segurança)
- [PARTE III — Mini cheat-sheet: o primeiro movimento por classe](#parte-iii--mini-cheat-sheet-o-primeiro-movimento-por-classe)

---

> A parte que tira você do zero: instalar e configurar todo o arsenal (recon + Burp + labs + IA local), e um **cookbook de prompts** prontos para usar IA com segurança e eficácia. Releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase) — tudo aqui pressupõe uso autorizado.

---

## PARTE I — Setup completo do ambiente

### 1. Sistema base e dependências

A maioria dos caçadores usa **Linux** (Kali, Ubuntu, ou Debian) ou WSL2 no Windows. Você vai precisar de **Go**, **Python 3** e **git**.

```bash
# Atualiza e instala dependências base (Debian/Ubuntu)
sudo apt update && sudo apt install -y golang-go python3 python3-pip git jq curl wget unzip

# Garante que binários Go fiquem no PATH (adicione ao ~/.bashrc ou ~/.zshrc)
echo 'export PATH=$PATH:$(go env GOPATH)/bin:$HOME/.local/bin' >> ~/.bashrc
source ~/.bashrc
```

### 2. Suíte ProjectDiscovery (o núcleo do recon)

A forma mais fácil é via **pdtm** (ProjectDiscovery Tool Manager), que instala e mantém tudo atualizado.

```bash
go install -v github.com/projectdiscovery/pdtm/cmd/pdtm@latest
pdtm -ia          # instala TODAS as tools PD (subfinder, httpx, nuclei, katana, dnsx, naabu, asnmap, alterx...)
pdtm -update      # atualiza tudo depois

# Atualiza os templates do nuclei (faça com frequência)
nuclei -update-templates
```

### 3. Utilitários de pipeline (tomnomnom & cia.)

```bash
# Os "tijolos" de encanamento e filtragem
go install github.com/tomnomnom/assetfinder@latest
go install github.com/tomnomnom/anew@latest
go install github.com/tomnomnom/qsreplace@latest
go install github.com/tomnomnom/unfurl@latest
go install github.com/tomnomnom/gf@latest
go install github.com/tomnomnom/waybackurls@latest
go install github.com/lc/gau/v2/cmd/gau@latest

# Padrões do gf (instala em ~/.gf)
git clone https://github.com/1ndianl33t/Gf-Patterns ~/.gf
git clone https://github.com/tomnomnom/gf /tmp/gf && cp /tmp/gf/examples/*.json ~/.gf/

# uro (deduplica URLs) e outras via pip
pip install uro --break-system-packages
```

### 4. Resolvers e wordlists (qualidade importa muito)

```bash
# SecLists — o repositório de wordlists de referência
git clone https://github.com/danielmiessler/SecLists ~/wordlists/SecLists

# Resolvers DNS confiáveis (atualize periodicamente; resolvers ruins arruínam o bruteforce)
wget -q https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt -O ~/wordlists/resolvers.txt
```

> 🎯 **Wordlists que importam:** `SecLists/Discovery/DNS/` (subdomínios), `SecLists/Discovery/Web-Content/` (caminhos), e listas **contextuais** que você gera com IA (Parte II). Resolvers ruins geram falsos positivos em massa — mantenha a lista limpa e atual.

### 5. Configuração de API keys do subfinder

Sem keys, a coleta passiva fica pobre. Edite `~/.config/subfinder/provider-config.yaml`:

```yaml
# ~/.config/subfinder/provider-config.yaml (exemplo — use SUAS chaves)
virustotal: ["SUA_KEY"]
securitytrails: ["SUA_KEY"]
shodan: ["SUA_KEY"]
github: ["SEU_TOKEN_GITHUB"]
censys: ["ID:SECRET"]
```

> Muitas dessas têm tier gratuito (VirusTotal, SecurityTrails, GitHub, Shodan limitado). Comece pelas grátis; o ganho de cobertura é enorme. A `chaos` da ProjectDiscovery também tem key gratuita.

### 6. Ferramentas complementares (JS, params, GraphQL, takeover)

```bash
# Análise de JS e parâmetros
go install github.com/003random/getJS@latest
pip install arjun --break-system-packages
git clone https://github.com/GerbenJavado/LinkFinder && cd LinkFinder && pip install -r requirements.txt --break-system-packages; cd ..
go install github.com/BishopFox/jsluice/cmd/jsluice@latest

# Triagem visual
go install github.com/sensepost/gowitness@latest

# Takeover & cloud
go install github.com/PentestPad/subzy@latest

# GraphQL
pip install graphql-cop --break-system-packages
git clone https://github.com/dolevf/graphw00f

# JWT
git clone https://github.com/ticarpi/jwt_tool && cd jwt_tool && pip install -r requirements.txt --break-system-packages; cd ..

# Notificação (alertas de recon contínuo)
go install github.com/projectdiscovery/notify/cmd/notify@latest
```

### 7. Burp Suite — instalação e configuração

1. Baixe em [portswigger.net/burp](https://portswigger.net/burp) (Community para começar; Pro recomendado para caçar a sério).
2. Configure o navegador para usar o proxy do Burp (porta 8080) — use o **navegador embutido** do Burp ou **FoxyProxy** no Firefox/Chrome.
3. Instale o certificado CA do Burp (`http://burp` → "CA Certificate") para interceptar HTTPS.
4. **Extensões essenciais** (Extensions → BApp Store): Autorize, Param Miner, Turbo Intruder, JWT Editor, InQL, Logger++, Collaborator Everywhere, Backslash Powered Scanner.

> O **DOM Invader** já vem no navegador embutido do Burp — ative-o nas configurações do navegador para acelerar DOM XSS / prototype pollution / postMessage.

### 8. Labs locais (Docker)

```bash
# Instale o Docker, depois suba os labs
docker run -d -p 3000:3000 bkimminich/juice-shop          # web moderno
docker run -d -p 80:80 vulnerables/web-dvwa               # clássicos
docker run -d -p 8080:8080 webgoat/webgoat                # lições OWASP
docker run -d -p 5000:5000 erev0s/vampi                   # API REST vulnerável
docker run -d -p 5013:5013 dolevf/dvga                    # GraphQL vulnerável
git clone https://github.com/OWASP/crAPI && cd crAPI && docker compose up -d; cd ..   # API realista
```

### 9. IA local (para dados sensíveis) — Ollama

Quando precisar processar **qualquer coisa sensível do alvo** sem mandar para fora, rode um modelo localmente.

```bash
# Instala o Ollama e baixa um modelo capaz
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.1          # ou outro modelo de sua preferência
ollama run llama3.1           # chat local; nada sai da sua máquina
```

> 🔒 Regra do [motor de recon](../01-recon/recon-engine-ia-e-automacao.md): PII real, credenciais, código sob NDA → **modelo local**. Conceitos e JS público → modelo na nuvem é OK.

### 10. Estrutura de diretórios sugerida

```
~/bb/
├── recon/<programa>/        # saídas de recon por alvo (subs, hosts, urls, js, shots)
├── notes/                   # seu "segundo cérebro" (markdown por programa/classe)
├── scripts/                 # seus scripts e oneliners (versionados no GitHub)
├── wordlists/               # SecLists + resolvers + listas contextuais
└── reports/                 # rascunhos de report por achado
```

> Organização não é firula: quando você está caçando em 5 programas, achar de novo aquele endpoint suspeito de 3 semanas atrás é o que separa o profissional do amador. Documente tudo no `notes/`.

---

## PARTE II — Cookbook de prompts de IA para segurança

Prompts prontos, organizados por tarefa. **Adapte e copie.** Antes de cada um, releia a regra de ouro: **nunca cole dados sensíveis do alvo (PII, credenciais, código sob NDA) em modelo de terceiro** — use modelo local (Parte I.9) ou não use. E **tudo que a IA responde é hipótese até você confirmar** na ferramenta/no alvo.

> **Como tirar o máximo:** (1) dê papel e contexto de autorização; (2) peça raciocínio, não só resposta; (3) force formato (tabela/lista); (4) itere do amplo ao específico; (5) desconfie e valide.

### 1. Análise e deobfuscação de JavaScript

```
Aja como um pesquisador de AppSec. Estou testando, com autorização, um alvo de
bug bounty. Abaixo está um trecho de JavaScript público da aplicação (minificado).
Tarefas:
1. Explique em alto nível o que esse código faz.
2. Liste todos os endpoints/rotas de API que ele chama (caminho + método + parâmetros).
3. Aponte sources e sinks de DOM XSS (ex.: location → innerHTML).
4. Sinalize qualquer chave, token ou segredo aparente (e diga se parece público/inerte).
Responda em tabelas separadas por tarefa.

[COLE O JS PÚBLICO AQUI]
```

### 2. Geração de wordlists contextuais

```
Estou fazendo content discovery autorizado em uma aplicação de [SETOR: ex. fintech].
Gere uma wordlist de ~80 caminhos/endpoints prováveis, específicos do domínio de
negócio (ex.: para fintech: kyc, ledger, payout, chargeback, settlement, webhook...).
Inclua variações com versões (v1, v2), verbos comuns e prefixos internos
(internal, admin, debug). Saída: uma palavra por linha, sem comentários.
```

### 3. Sumarização e priorização de recon

```
Abaixo está a saída de httpx de um recon autorizado (host, status, título, tecnologia).
Tarefas:
1. Agrupe os hosts por tecnologia/stack.
2. Destaque painéis administrativos, ambientes dev/staging, APIs versionadas antigas
   e qualquer coisa que cheire a debug.
3. Priorize os 5 alvos mais promissores para teste manual e explique o porquê de cada.
Lembre-se: isto é só priorização; vou validar tudo manualmente.

[COLE A SAÍDA DE httpx/nuclei/urls AQUI]
```

### 4. Reconstrução de superfície de API a partir de Swagger/JS

```
Aqui está a especificação OpenAPI/Swagger de uma API que estou testando com autorização.
Tarefas:
1. Liste os endpoints mais sensíveis (auth, usuários, pagamentos, admin) com método.
2. Para cada um, gere hipóteses de BOLA, BFLA e mass assignment (BOPLA).
3. Sugira, para cada endpoint, um teste manual concreto que eu poderia fazer no Burp.
Saída em tabela: endpoint | método | hipótese de bug | teste manual.

[COLE O swagger.json / TRECHO RELEVANTE]
```

### 5. Geração de scripts de recon/automação

```
Escreva um script Python (use requests) que:
- Lê uma lista de hosts de hosts.txt;
- Para cada host, faz GET em /api/v1/<X> e /api/v2/<X> (X vindo de paths.txt);
- Compara status e tamanho das respostas;
- Imprime os endpoints presentes só na v1 (legados, candidatos a falta de controle).
Respeite um delay configurável entre requests (rate limit). Comente o código.
```

### 6. Geração de templates Nuclei

```
Escreva um template Nuclei (YAML) que detecta [DESCREVA O PADRÃO: ex. respostas
contendo o header X-Debug-Token com status 200]. Severidade: medium. Tags: custom.
Explique a lógica dos matchers para eu revisar antes de rodar.
```

### 7. Triagem de achados (com validação humana)

```
Abaixo estão N saídas de uma ferramenta de detecção (nuclei) de um teste autorizado.
Para cada item: (a) probabilidade de falso-positivo e por quê; (b) impacto provável
se for real; (c) qual seria meu próximo passo manual para confirmar.
Deixe claro que a decisão final e a confirmação são minhas.

[COLE OS ACHADOS]
```

### 8. Code review assistido (CVE hunting — só código PÚBLICO)

```
Aja como auditor de segurança de aplicações. O código abaixo é de um projeto OPEN
SOURCE público. Tarefas:
1. Aponte sinks perigosos (eval/exec/query concatenada/desserialização/etc.).
2. Para cada sink, rastreie se há um caminho de input do usuário até ele (input → sink).
3. Descreva como um atacante exploraria, e que validação está faltando.
4. Sugira um PoC mínimo que eu rodaria LOCALMENTE na minha instância.
Vou validar e reproduzir tudo localmente antes de qualquer disclosure.

[COLE AS FUNÇÕES PÚBLICAS]
```

### 9. Brainstorm de vetores por funcionalidade

```
Estou testando, com autorização, uma funcionalidade que [DESCREVA: ex. recebe a URL
de uma imagem e gera um thumbnail no servidor]. Liste vetores de ataque plausíveis,
do mais ao menos provável, e como eu testaria cada um manualmente (ex.: SSRF via URL
interna/Collaborator, RCE via ImageMagick/ffmpeg, path traversal no nome, XXE em SVG).
```

### 10. Entender uma stack/tecnologia desconhecida

```
Caí num alvo autorizado rodando [TECNOLOGIA/FRAMEWORK que não conheço]. Explique:
1. O modelo típico de autenticação/autorização dessa stack.
2. As classes de bug mais comuns historicamente associadas a ela.
3. Onde eu deveria olhar primeiro e que comportamentos suspeitos procurar.
```

### 11. Redação e revisão de report (você garante a precisão)

```
Ajude a redigir um report de bug bounty profissional a partir destes fatos
(eu confirmei tecnicamente tudo). Estrutura: título, resumo, severidade/impacto de
negócio, passos para reproduzir (numerados), PoC de impacto mínimo, remediação,
referências. Tom objetivo, sem hype. Redija qualquer PII como [REDIGIDO].

Fatos: [DESCREVA o bug, endpoint, passos, impacto — sem PII real]
```

### 12. Explicar um payload/PoC que você não entende 100%

```
Explique, passo a passo, por que o payload/PoC abaixo funciona contra [classe de bug],
o que cada parte faz, e em que condições ele falharia. Quero entender o mecanismo,
não só usar. [COLE O PAYLOAD/PoC — de fonte pública/lab]
```

---

## PARTE III — Mini cheat-sheet: o primeiro movimento por classe

Quando o recon te entrega um endpoint, qual o **primeiro teste manual** por classe? (Detalhes completos no [playbook de classes de bug](../03-playbooks/classes-de-bug.md).)

| Classe | Primeiro movimento manual |
|---|---|
| IDOR/BOLA | Duas contas; trocar o ID; rodar Autorize |
| BFLA | Chamar endpoint de admin como user; trocar verbo HTTP |
| Mass assignment | Injetar `"role":"admin"` / `"verified":true` no body |
| SQLi | `'` → observar erro; `' AND SLEEP(5)-- -` (blind) |
| SSRF | Apontar parâmetro de URL para o Collaborator |
| XSS | Marcador único → ver contexto de reflexão → quebrar contexto |
| DOM XSS | Tracear source→sink no JS; DOM Invader |
| XXE | Enviar XML com entidade externa → ver arquivo/Collaborator |
| SSTI | `{{7*7}}` / `${7*7}` → ver se avalia para 49 |
| Path traversal | `../../etc/passwd` + variações encodadas |
| Open redirect | `redirect=//evil.com` → ver se sai do domínio |
| CORS | `Origin: https://evil.com` → ver reflexão + credentials |
| JWT | Testar `alg:none`; brute do segredo HMAC (jwt_tool) |
| Race condition | Turbo Intruder, single-packet attack |
| Logic flaw | Perguntar: "que regra essa função impõe? como eu quebro?" |
| Subdomain takeover | CNAME órfão → confirmar reivindicação (subzy) |

---

> **Fim do setup + cookbook.** Com o ambiente montado (Parte I), os prompts em mãos (Parte II) e o primeiro movimento por classe (Parte III), você tem tudo para operar a metodologia 2026 desde o primeiro dia. Volte ao [README](../README.md) para o mapa geral. O resto é prática deliberada — e disciplina de escopo. Boa caçada. 🎯

---

⬅️ [Anterior: Cenários Práticos](../04-pratica/cenarios-praticos.md) · [⬆️ Topo](#) · [Próximo: Técnicas Avançadas e Cadeias ➡️](../03-playbooks/tecnicas-avancadas-e-cadeias.md)
