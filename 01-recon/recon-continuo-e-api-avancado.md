# 07 — API Avançado + Recon Contínuo & Automação em Escala

[🏠 Início](../README.md) › [01 · Recon](README.md) › Recon Contínuo + API Avançado

**Sumário:**
- [PARTE I — Recon contínuo e monitoramento](#parte-i--recon-contínuo-e-monitoramento)
- [PARTE II — Frameworks de automação de recon](#parte-ii--frameworks-de-automação-de-recon)
- [PARTE III — Descoberta de ativos em cloud](#parte-iii--descoberta-de-ativos-em-cloud)
- [PARTE IV — Templates Nuclei customizados](#parte-iv--templates-nuclei-customizados-detecção-dirigida)
- [PARTE V — API: técnicas avançadas (além do F3)](#parte-v--api-técnicas-avançadas-além-do-f3)
- [PARTE VI — Coletânea de oneliners úteis](#parte-vi--coletânea-de-oneliners-úteis)

---

> A camada "industrial" da metodologia 2026: transformar recon pontual em um **sistema que roda sozinho e te avisa quando algo novo aparece**, mais técnicas avançadas de API. Tudo para uso **autorizado** (releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase)) — automação em escala exige ainda mais disciplina de escopo e rate limit.

---

## PARTE I — Recon contínuo e monitoramento

A diferença entre o caçador casual e o profissional em 2026: o profissional **não faz recon uma vez** — ele monta um pipeline que monitora os alvos **continuamente** e dispara um alerta quando surge um subdomínio novo, um endpoint novo, uma tecnologia nova. Você chega primeiro no que acabou de ser exposto.

### 1. Por que monitoramento contínuo vence

- **Superfície muda toda semana:** deploys criam subdomínios, APIs novas, ambientes de staging esquecidos.
- **Quem chega primeiro, ganha:** um subdomínio recém-exposto com bug é um bug ainda não duplicado. Monitoramento te dá vantagem temporal.
- **Escala sem esforço manual:** depois de montado, o pipeline trabalha enquanto você dorme; você só investiga os **diffs**.

### 2. O padrão "diff de recon"

A ideia central é simples: rode o recon periodicamente, **compare com a execução anterior** e investigue só o que é **novo**.

```bash
#!/usr/bin/env bash
# monitor.sh — recon incremental com alerta de novidades (alvo AUTORIZADO)
# Requer: subfinder, dnsx, httpx, anew, notify (ProjectDiscovery)

DOMINIO="$1"
mkdir -p "recon/$DOMINIO"
cd "recon/$DOMINIO" || exit 1

# 1) coleta subdomínios e guarda só os NOVOS em new_subs.txt
subfinder -d "$DOMINIO" -all -silent \
  | dnsx -silent \
  | anew all_subs.txt | tee new_subs.txt

# 2) probing dos novos e alerta
if [ -s new_subs.txt ]; then
  httpx -silent -status-code -title -tech-detect < new_subs.txt \
    | anew all_hosts.txt \
    | notify -silent -bulk    # manda pro seu Discord/Telegram/Slack
fi
```

> [`anew`](https://github.com/tomnomnom/anew) é a peça-chave: adiciona a um arquivo só linhas inéditas **e** as imprime no stdout — perfeito para "o que mudou desde a última vez". [`notify`](https://github.com/projectdiscovery/notify) entrega o diff onde você quiser.

### 3. Agendamento

```bash
# crontab -e  → roda o monitor todo dia às 6h (ajuste à carga e ao escopo!)
0 6 * * * /home/voce/recon/monitor.sh exemplo.com >> /home/voce/recon/cron.log 2>&1
```

> ⚠️ **Ética em escala:** agendamento amplifica volume. Espace execuções, respeite rate limits do alvo e nunca rode brute force diário pesado contra a origem. Monitorar **mudança de superfície** (passivo + probing leve) é educado; martelar a origem todo dia **não é**.

### 4. Notificação multi-canal

[`notify`](https://github.com/projectdiscovery/notify) suporta Slack, Discord, Telegram, e-mail e mais. Configure em `~/.config/notify/provider-config.yaml` e canalize qualquer pipeline para um alerta:

```bash
echo "novo painel admin: https://admin-stg.exemplo.com" | notify -silent
nuclei -list new_hosts.txt -severity high,critical -silent | notify -silent -bulk
```

---

## PARTE II — Frameworks de automação de recon

Você pode encadear tudo do [motor de recon](recon-engine-ia-e-automacao.md) à mão (recomendado para **entender** o pipeline) ou usar frameworks que já orquestram o funil. **Aprenda manual primeiro; use framework para escalar** — e leia o que cada um faz por baixo.

| Framework | O que é | Observação |
|---|---|---|
| [reconFTW](https://github.com/six2dez/reconftw) | Orquestra subdomínios→probing→fuzzing→nuclei→relatório, ponta a ponta | O mais completo; muitos botões — entenda antes de soltar |
| [Osmedeus](https://github.com/j3ssie/osmedeus) | Engine de recon com workflows e UI | Bom para rodar em servidor |
| [Axiom](https://github.com/pry0cc/axiom) | Distribui recon em fleets de VMs efêmeras na nuvem | Para escala real (paraleliza ffuf/nuclei em dezenas de VMs) |
| [reNgine](https://github.com/yogeshojha/rengine) | Plataforma web de recon com banco e diff | Interface amigável, scans agendados |
| [nuclei + templates](https://github.com/projectdiscovery/nuclei) | A peça de detecção comum a quase todos | Customize com seus próprios templates (Parte IV) |

```bash
# reconFTW: recon completo de um alvo autorizado (-r = reconhecimento completo)
./reconftw.sh -d exemplo.com -r

# Axiom: dispara um fleet e distribui um scan de ffuf entre as VMs
axiom-fleet reconfleet -i 10
axiom-scan urls.txt -m ffuf -wordlist=raft-large.txt -o resultados/
```

> **Princípio:** framework é multiplicador de força para quem **já sabe** o que está acontecendo. Para o iniciante vira caixa-preta que cospe ruído. Domine o [funil manual do motor de recon](recon-engine-ia-e-automacao.md), depois acelere.

---

## PARTE III — Descoberta de ativos em cloud

Programas modernos vivem em cloud; buckets, funções e registries expostos são uma superfície inteira que o funil web padrão não cobre.

| Alvo | Ferramenta | O que procura |
|---|---|---|
| Buckets S3/GCS/Azure | [cloud_enum](https://github.com/initstring/cloud_enum), [S3Scanner](https://github.com/sa7mon/S3Scanner) | Buckets públicos, listáveis, graváveis |
| Ranges/ASN de cloud | [asnmap](https://github.com/projectdiscovery/asnmap), [cidr](https://github.com/projectdiscovery/cidr) | IPs do alvo na nuvem |
| Subdomain takeover | [subzy](https://github.com/PentestPad/subzy), [nuclei takeover templates](https://github.com/projectdiscovery/nuclei-templates) | CNAMEs apontando para serviços não reclamados |
| Segredos em git/CI | [trufflehog](https://github.com/trufflesecurity/trufflehog), [gitleaks](https://github.com/gitleaks/gitleaks) | Chaves vazadas em repositórios |

```bash
# Enumeração de buckets a partir de uma "keyword" da empresa
python3 cloud_enum.py -k exemplocorp -k exemplo-corp

# Subdomain takeover: checa todos os subs achados
subzy run --targets all_subs.txt --hide_fails
```

> 🎯 **Subdomain takeover** é um clássico de alto valor e fácil de monitorar: um CNAME apontando para um serviço desativado pode ser "reclamado" por você. Inclua um check de takeover no seu pipeline contínuo.

---

## PARTE IV — Templates Nuclei customizados (detecção dirigida)

A automação só te dá vantagem real quando você **escreve seus próprios detectores** para padrões específicos do alvo (um endpoint característico, uma misconfig recorrente, uma resposta de erro reveladora). É aqui que a **IA acelera muito**: descreva o padrão e peça o YAML.

### Anatomia de um template

```yaml
id: painel-admin-exposto
info:
  name: Painel admin exposto sem auth
  author: voce
  severity: high
  tags: exposure,custom

http:
  - method: GET
    path:
      - "{{BaseURL}}/admin/dashboard"
    matchers-condition: and
    matchers:
      - type: status
        status:
          - 200
      - type: word
        words:
          - "Admin Dashboard"
          - "User Management"
        condition: and
```

```bash
# Roda só o seu template contra os hosts vivos
nuclei -list hosts_vivos.txt -t meus-templates/painel-admin-exposto.yaml -silent
```

### Padrões úteis para escrever (peça à IA o YAML e revise)

- Detector de uma **versão de API legada** (`/api/v1` que ainda responde).
- Detector de **mensagens de erro verbose** específicas da stack do alvo.
- Detector de **headers reveladores** (debug, framework, versão).
- Detector de **endpoint característico** que você viu num subdomínio e quer caçar em todos.

> **Workflow IA→template:** "Escreva um template Nuclei que detecta respostas contendo `X-Debug-Token` com status 200; severidade medium; tag custom." → você revisa a lógica do matcher → roda. Multiplica sua capacidade de detecção dirigida sem virar "scanner runner" (porque **você** definiu o que procurar e **valida** cada hit).

---

## PARTE V — API: técnicas avançadas (além do F3)

Com a base de [API Bug Bounty (F3)](../02-trilha/02-api-f3.md) dominada, aprofunde nestes vetores que separam o caçador de API competente do avançado.

### 1. Versionamento e inventário (API9)

O vetor mais subestimado. APIs antigas não recebem os mesmos controles das novas.

**Metodologia:**
- Para todo endpoint `/api/v2/x`, teste `/api/v1/x`, `/api/v3/x`, `/api/beta/x`, `/api/internal/x`.
- Compare respostas entre versões: a v1 frequentemente **não tem** as checagens de autorização da v2.
- Procure docs de versões antigas (Swagger/OpenAPI versionado) no recon.

```bash
# Compara presença/resposta entre versões para um conjunto de paths
for p in $(cat paths.txt); do
  for v in v1 v2 v3 internal beta; do
    code=$(curl -s -o /dev/null -w "%{http_code}" "https://api.exemplo.com/$v/$p")
    echo "$v/$p -> $code"
  done
done | sort
```

### 2. Mass assignment / BOPLA aprofundado (API3)

- Compare o objeto retornado por um `GET` com os campos aceitos num `PUT/PATCH/POST` — campos lidos mas "não editáveis" são candidatos.
- Tente injetar campos de outras entidades (`"organization_id"`, `"is_staff"`, `"verified"`, `"balance"`).
- Teste em **criação** (signup) e em **atualização** (perfil) — o controle costuma diferir.

### 3. Excessive Data Exposure (lado de BOPLA)

- A API retorna o objeto **inteiro** e o front "esconde" campos no JS? Olhe o JSON cru no Burp — tokens, hashes, PII, flags internas costumam vir a mais.
- Endpoints de listagem (`/users`, `/orders`) frequentemente vazam campos sensíveis em massa.

### 4. Rate limiting e abuso de fluxo (API4/API6)

- Rate limit por IP é burlável com `X-Forwarded-For`/rotação; por conta, com múltiplas contas.
- **GraphQL:** aliases/batching para multiplicar operações numa request (brute de OTP, scraping, voto múltiplo).
- Avalie o **impacto de negócio** do abuso (não só "não tem rate limit").

### 5. JWT e auth em API (API2)

Veja o [playbook C6 (JWT Attacks)](../03-playbooks/classes-de-bug.md#c6-jwt-attacks). Em API, atenção extra a: tokens que não expiram, refresh tokens reutilizáveis, `alg confusion`, e segredos fracos (brute offline com jwt_tool).

### 6. GraphQL avançado

- **Introspection desabilitada?** Tente *field suggestion* (erros que sugerem campos), ferramentas como [clairvoyance](https://github.com/nikitastupin/clairvoyance) para reconstruir o schema às cegas.
- **DoS por profundidade/complexidade:** queries aninhadas profundas (avalie com cautela; pode derrubar — **não** faça em prod sem autorização explícita).
- **Mutations sensíveis:** mapeie criação de usuário, mudança de papel, transferência — e teste BOLA/BFLA em cada.

```graphql
# Batching para brute de OTP (em LAB / escopo autorizado)
[
  { "query": "mutation { verifyOtp(code:\"0001\"){ token } }" },
  { "query": "mutation { verifyOtp(code:\"0002\"){ token } }" }
]
```

### 7. Webhooks e integrações (SSRF/API10)

- Configuração de webhook = SSRF quase de graça (aponte a URL para interno/Collaborator).
- Dados vindos de terceiros (API10) processados sem validação → injeções de segunda ordem.

---

## PARTE VI — Coletânea de oneliners úteis

Linha de comando é onde a velocidade mora. Adapte sempre ao escopo e ao rate limit.

```bash
# Subdomínios vivos com título e tecnologia, num passo
subfinder -d exemplo.com -all -silent | dnsx -silent | httpx -silent -title -tech-detect

# Todas as URLs históricas com parâmetros, limpas e separadas por classe
gau --subs exemplo.com | uro | grep '=' | tee params.txt \
  | gf xss > cand_xss.txt; gf ssrf < params.txt > cand_ssrf.txt

# Extrai todos os JS de hosts vivos e garimpa endpoints
cat hosts_vivos.txt | getJS --complete | sort -u \
  | xargs -I{} sh -c 'jsluice urls -R {} 2>/dev/null' | sort -u

# Procura segredos em todos os JS coletados
cat js_files.txt | xargs -I{} sh -c 'curl -s {} | grep -aoE "(api[_-]?key|secret|token)[\"'\'' :=]+[A-Za-z0-9_\-]{16,}"'

# Subdomain takeover check em massa
cat all_subs.txt | dnsx -cname -resp-only | sort -u | subzy run --targets - --hide_fails

# Probing de portas web comuns além de 80/443
cat all_subs.txt | httpx -silent -ports 80,443,8080,8443,8000,3000,5000 -title

# Diff diário de subdomínios com alerta
subfinder -d exemplo.com -all -silent | anew subs.txt | notify -silent
```

> Cada oneliner é um tijolo. O caçador de 2026 monta os seus, versiona no repo `recon-scripts`, e os encadeia em pipelines reaproveitáveis — sempre entendendo o que cada estágio faz e validando o resultado à mão.

---

> **Fechamento:** com o [motor de recon](recon-engine-ia-e-automacao.md) (funil manual), o [playbook de classes de bug](../03-playbooks/classes-de-bug.md) (como quebrar cada classe) e este documento (escala + continuidade + API avançado), você tem o motor **e** a profundidade. O que falta é prática deliberada e disciplina. Os [apêndices](../05-referencia/apendices.md) fecham com plano concreto, cronograma, glossário, FAQ e modelo de report.

---

⬅️ [Anterior: Playbook de Classes de Bug](../03-playbooks/classes-de-bug.md) · [⬆️ Topo](#) · [Próximo: Apêndices Práticos ➡️](../05-referencia/apendices.md)
