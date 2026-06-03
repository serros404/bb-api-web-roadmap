# 14 — Cloud & Containers (F10)

> **Por que esta fase fecha a lacuna de empregabilidade:** a infra do mundo migrou para **AWS, Azure e GCP** — e o pentest acompanhou. Vagas de pentest/red team em 2026 cada vez mais pedem "cloud security". O diferencial: a maioria dos caçadores web sabe achar um bucket S3 público, mas **poucos sabem escalar privilégio dentro do IAM, abusar de identidades gerenciadas, ou pivotar do AD on-prem para o Entra ID**. Quem sabe isso é raro e bem pago.

> **A tese, na nuvem:** *amplitude por máquina, profundidade por humano.* Ferramentas como **ScoutSuite/Prowler/Pacu** enumeram milhares de recursos e permissões e apontam misconfigs (amplitude); **a cadeia de escalonamento de privilégio do IAM — "essa permissão `iam:PassRole` + `lambda:CreateFunction` me leva a admin" — é raciocínio humano** (profundidade). Scanner lista permissões; só você as encadeia até o impacto.

> ⚠️ **Ética (releia o `00`):** teste **apenas** (1) suas próprias contas cloud, (2) labs intencionalmente vulneráveis, (3) escopo cloud explicitamente autorizado por contrato. Cloud tem nuances: enumerar recursos de terceiros, mesmo "só listando", pode violar ToS do provedor e a lei. Em engajamento, **escopo de cloud é por conta/assinatura/recurso** e exige autorização do dono da conta — não só do dono da aplicação. Cuidado redobrado com **custo** (um loop mal feito gera fatura) e com **dados** (PII em buckets/DBs).

---

# ☁️ F10 — Pentest de Cloud & Containers

**Duração:** 2–4 meses · **Carga:** ~15h/sem · **Badge:** ☁️ *Andarilho da Nuvem*

**Foco:** entender o modelo de segurança das três grandes nuvens, **enumerar e escalar privilégio no IAM**, abusar de serviços (storage, compute, serverless, secrets), explorar **identidade no Entra ID/Azure** (incluindo o elo híbrido com o AD da F9), e atacar **containers/Kubernetes**.

> **O loop de F10:** credencial/role inicial → enumeração automatizada (ScoutSuite/Prowler/Pacu/roadrecon) → mapeia permissões e recursos → você identifica a **cadeia de privesc** ou o recurso exposto → executa → demonstra impacto mínimo → documenta. IA ajuda a interpretar uma policy IAM ou um achado do ScoutSuite; **a cadeia é sua**.

<details>
<summary>📚 <strong>Estuda</strong> — fundamentos cloud, AWS, Azure/Entra, GCP e containers</summary>

**1. Fundamentos transversais (antes de qualquer provedor)**
- **Modelo de responsabilidade compartilhada** (o que é do provedor vs do cliente — define o que está em escopo).
- **IAM** é o coração de tudo: identidades, policies, roles, *trust relationships*, privilege escalation por permissão mal dada.
- Serviços-chave por categoria: **compute** (EC2/VM/Compute Engine), **storage** (S3/Blob/GCS), **serverless** (Lambda/Functions), **secrets** (Secrets Manager/Key Vault), **networking** (VPC), **containers** (ECS/EKS/AKS/GKE).
- **Metadata service** (IMDS) — o vetor que liga SSRF de web a credenciais cloud (você já viu no `11`); aqui você aprende a explorá-lo a fundo.
- Referência: [HackTricks Cloud](https://cloud.hacktricks.wiki), [PayloadsAllTheThings — Cloud](https://github.com/swisskyrepo/PayloadsAllTheThings).

**2. AWS (o mais cobrado em pentest cloud)**
- **Enumeração:** `aws` CLI, **ScoutSuite**, **Prowler**, **enumerate-iam**, **CloudMapper**.
- **IAM privilege escalation:** as ~20+ rotas clássicas (mapeadas pela Rhino Security Labs) — ex.: `iam:CreatePolicyVersion`, `iam:PassRole` + `ec2:RunInstances`/`lambda:CreateFunction`, `sts:AssumeRole`. Estude o porquê de cada uma.
- **S3:** buckets públicos/listáveis/graváveis (você já viu no recon — agora com lente de privesc/exfil).
- **EC2 + IMDS:** SSRF/foothold → `169.254.169.254` → credenciais temporárias da role (IMDSv1) → assumir a role.
- **Lambda, Secrets Manager, SSM, RDS, SNS/SQS:** vetores de exfil e privesc.
- **Pacu** (o "Metasploit da AWS"): framework de exploração pós-credencial.

**3. Azure & Entra ID (⭐ e o elo híbrido com o AD da F9)**
- **Entra ID (antigo Azure AD)** é identidade-as-a-service; muito do AD enterprise hoje é **híbrido**.
- **Enumeração:** **AzureHound** (BloodHound para Azure), **ROADtools/roadrecon** (dirkjanm), **AADInternals**, **MicroBurst**, **Stormspotter**.
- **Vetores de identidade:** **device code phishing**, **illicit consent grant** (app maliciosa), **PRT (Primary Refresh Token) abuse**, bypass de **Conditional Access**, abuso de **service principals / app registrations** e **managed identities**.
- **Híbrido (a ponte AD→Cloud):** **Azure AD Connect** (PHS/PTA/Seamless SSO) — comprometer o servidor de sync ou abusar da sincronização pode levar do on-prem à nuvem e vice-versa. Estude a research de **dirkjanm** e **SpecterOps** sobre isto.
- **Azure RBAC vs Entra roles** (dois modelos de permissão que se confundem e geram privesc).

**4. GCP**
- **Enumeração:** **ScoutSuite**, **GCPBucketBrute**, `gcloud` CLI, **gcp_scanner**.
- Privesc por IAM roles/service accounts, abuso de `iam.serviceAccounts.getAccessToken`, buckets GCS, metadata.

**5. Containers & Kubernetes (superfície inteira própria)**
- **Docker:** escape de container, sockets expostos (`/var/run/docker.sock`), imagens com segredos.
- **Kubernetes:** RBAC mal configurado, dashboards/API expostos, abuso de service accounts e tokens montados, escape de pod para nó, secrets. Ferramentas: **kube-hunter**, **kubeaudit**, `kubectl`, **peirates**.
- Referência: [HackTricks — K8s & Docker](https://cloud.hacktricks.wiki), [Kubernetes Goat docs](https://madhuakula.com/kubernetes-goat/).

**6. CI/CD & supply chain (conecta com o `07`)**
- Pipelines expostos, **OIDC** mal configurado (GitHub Actions → AWS sem chave estática mas explorável), segredos em CI, runners comprometidos. **trufflehog/gitleaks** (você já conhece) para segredos em repos.

</details>

<details>
<summary>🧪 <strong>Pratica</strong> — arsenal, comandos e LABS de cloud (muitos grátis)</summary>

**Arsenal de cloud:**
```bash
# AWS
pip install awscli
pip install scoutsuite          # multi-cloud audit (AWS/Azure/GCP)
pip install prowler             # auditoria/benchmark AWS (e mais)
pipx install pacu               # exploração pós-credencial AWS
#  enumerate-iam, CloudMapper, S3Scanner → github

# Azure / Entra
pipx install roadrecon          # ROADtools (dirkjanm) — enum Entra ID
#  AzureHound (binário, github.com/SpecterOps/AzureHound)
#  AADInternals (PowerShell): Install-Module AADInternals
#  MicroBurst (PowerShell, NetSPI): github.com/NetSPI/MicroBurst

# GCP
pip install gcp_scanner
#  GCPBucketBrute → github

# Kubernetes
pip install kube-hunter
#  kubeaudit, peirates, kubectl → github / package manager
```

**Fluxo de referência (sua conta / LAB autorizado):**
```bash
# AWS — com um par de credenciais (de lab) configurado:
aws sts get-caller-identity                     # quem sou eu?
scout aws                                        # relatório HTML de misconfig
prowler aws                                      # achados por benchmark
#  IMDS (de dentro de uma EC2/SSRF de lab):
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
#  Pacu: enumerar permissões e procurar rota de privesc
pacu  → run iam__enum_permissions ; run iam__privesc_scan

# Azure/Entra — com sessão de lab:
roadrecon auth -u user@tenant -p 'senha'        # autentica
roadrecon gather                                 # coleta o tenant
roadrecon gui                                    # explora o grafo
#  AzureHound → importa no BloodHound CE (arestas Azure)

# Kubernetes — contra cluster de lab:
kube-hunter --remote <ip>                        # scan
kubectl auth can-i --list                        # o que meu token pode?
```

**LABS de cloud — comece pelos gratuitos:**

| Lab | Provedor | O que é | Custo |
|---|---|---|---|
| **[flaws.cloud](http://flaws.cloud)** / **[flaws2.cloud](http://flaws2.cloud)** | AWS | Os clássicos didáticos (S3, IAM, IMDS) — **comece aqui** | Grátis |
| **[CloudGoat](https://github.com/RhinoSecurityLabs/cloudgoat)** | AWS | Cenários vulneráveis que você implanta na **sua** conta (privesc, etc.) | Grátis (paga só o uso AWS) |
| **[PwnedLabs](https://pwnedlabs.io)** | AWS/Azure/GCP | Labs modernos e realistas de cloud pentest (free + paid) — **excelente** | Freemium |
| **[AzureGoat](https://github.com/ine-labs/AzureGoat)** / **[XMGoat](https://github.com/XMCyber/XMGoat)** | Azure | Ambientes Azure/Entra vulneráveis | Grátis |
| **[GCPGoat](https://github.com/ine-labs/GCPGoat)** / [ThunderCTF](https://thunder-ctf.cloud) | GCP | GCP vulnerável | Grátis |
| **[Kubernetes Goat](https://madhuakula.com/kubernetes-goat/)** / [Bust-a-Kube](https://www.bustakube.com) | K8s | Cluster vulnerável guiado | Grátis |
| [Altered Security — CARTP lab](https://www.alteredsecurity.com) | Azure | Lab oficial da cert de Azure red team | Pago |
| [TryHackMe — cloud rooms](https://tryhackme.com) | Multi | Intro a AWS/Azure/cloud | Freemium |

> 🎯 **Trilha sugerida:** flaws.cloud → flaws2.cloud → CloudGoat (3–4 cenários) → PwnedLabs (AWS e Azure) → AzureGoat + roadrecon → Kubernetes Goat. Híbrido AD→Azure: combine o **GOAD** (F9) com um tenant Entra de teste (conta dev grátis da Azure) para praticar o pivô on-prem→cloud.

> ⚠️ **Conta-laboratório:** crie uma conta AWS/Azure/GCP **separada**, ative billing alerts, e **derrube os recursos** depois de cada lab. CloudGoat tem `destroy` — use sempre. Loop infinito + serverless = fatura surpresa.

**IA como copiloto em cloud (regra do `01`/`10`):**
- Cole uma **policy IAM** (de lab) e peça: *"esta policy permite escalonamento de privilégio? por qual cadeia de ações?"*
- Achado do **ScoutSuite/Prowler** que você não entende: peça o impacto e o próximo passo manual.
- Peça para explicar um serviço cloud desconhecido e seus vetores históricos.
- 🔒 **Limite duro:** ARNs/IDs de conta reais, chaves, dados de tenant de cliente, output de engajamento sob NDA → **modelo local**. Lab/conceito público → nuvem OK. **Valide tudo.**

</details>

<details>
<summary>✍️ <strong>Posta</strong></summary>

- Writeup de um **cenário do CloudGoat ou PwnedLabs**: a cadeia de privesc no IAM, com diagrama. Cloud pentest tem **muito menos** conteúdo em português — você se destaca rápido escrevendo isso.
- Post conceitual: *"do SSRF ao IAM: como o IMDS conecta web e cloud"* (liga o seu conhecimento de web ao novo) ou *"o caminho híbrido: do AD on-prem ao Entra ID"*.
- Cheat-sheet de enumeração cloud no `notes`.

</details>

<details>
<summary>📖 <strong>Writeups da semana</strong></summary>

Research e writeups de cloud: [dirkjanm.io](https://dirkjanm.io) (Azure/Entra/híbrido — referência mundial), [SpecterOps blog](https://specterops.io/blog/) (Azure + BloodHound), [Rhino Security Labs](https://rhinosecuritylabs.com/blog/) (AWS), [PwnedLabs blog](https://pwnedlabs.io), [NetSPI blog](https://www.netspi.com/blog/). Pergunte: que permissão/serviço foi abusado, e eu teria mapeado a cadeia?

</details>

### 🎯 Missão F10 — "Cadeia de privesc na nuvem documentada"
Complete **flaws.cloud + flaws2.cloud + pelo menos 3 cenários do CloudGoat** e **1 lab de Azure/Entra** (AzureGoat ou PwnedLabs), e documente **uma cadeia completa de escalonamento de privilégio no IAM** do acesso inicial ao "admin" da conta/tenant.

**Critério de aceitação:** writeup de uma cadeia de privesc cloud (AWS **ou** Azure) ponta a ponta, com as permissões/recursos abusados e diagrama, em ambiente de lab/sua conta.

### ✅ Checklist F10
- [ ] Modelo de responsabilidade compartilhada e IAM claros
- [ ] AWS: enumeração (ScoutSuite/Prowler/Pacu) + ≥3 rotas de IAM privesc
- [ ] S3 + IMDS + Lambda/Secrets como vetores de exfil/privesc
- [ ] Azure/Entra: enumeração (AzureHound/roadrecon/AADInternals)
- [ ] Device code phishing / illicit consent / PRT entendidos
- [ ] **Caminho híbrido AD→Entra** (Azure AD Connect/PHS/PTA) compreendido
- [ ] GCP: enumeração + privesc por service account (noções)
- [ ] Containers/K8s: RBAC, tokens montados, escape (Kubernetes Goat)
- [ ] flaws.cloud + flaws2 + CloudGoat (3 cenários) + 1 lab Azure feitos
- [ ] Conta-lab com billing alerts e destroy disciplinado
- [ ] 1 cadeia de privesc cloud documentada
- [ ] (Meta) **CARTP** (Azure) ou cert AWS no horizonte (ver `16`)

---

➡️ **Próximo:** `15-RED-TEAM-OPS-F11.md` — a camada que transforma "pentester" em "red teamer": C2, evasão, OPSEC e simulação de adversário.
