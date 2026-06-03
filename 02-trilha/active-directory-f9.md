# 13 — Active Directory (F9)

> **Por que esta é A fase mais importante para a vaga:** ~95% das empresas médias/grandes rodam **Active Directory** como espinha dorsal de identidade. Todo pentest interno e quase todo engajamento de red team **vira um exercício de AD**: você entra com um usuário de baixo privilégio e o objetivo é **Domain Admin** (ou Enterprise Admin). Se você domina AD, você é contratável como pentester/red teamer no Brasil e lá fora. Esta fase é densa de propósito — reserve tempo.

> **A tese, aplicada ao AD de forma quase literal:** o **BloodHound** é a personificação de "máquina mapeia, humano explora" — o coletor (SharpHound) varre o domínio e desenha o **grafo de caminhos de ataque** (amplitude); **você lê o grafo, escolhe o caminho e executa o abuso** (profundidade). A ferramenta te mostra que "do grupo X dá pra chegar em DA"; *como* percorrer cada aresta é raciocínio humano.

> ⚠️ **Ética (releia o `00`):** ataques de AD são **barulhentos e perigosos** — DCSync, coerção, relay e despejo de LSASS podem disparar EDR, travar contas e quebrar serviços. **Só em lab próprio ou engajamento contratado com RoE.** Despejar credenciais de um domínio real sem autorização é crime grave. Em engajamento, tudo é com impacto mínimo, deconfliction com o cliente e OPSEC para não causar dano.

---

# 🏰 F9 — Ataque a Active Directory

**Duração:** 3–5 meses · **Carga:** ~15–20h/sem · **Badge:** 🏰 *Conquistador de Domínio*

**Foco:** dominar o ciclo completo de comprometimento de AD: **enumerar → ganhar credencial inicial → mover lateralmente → escalar privilégio → dominar o domínio → persistir**, entendendo a teoria (Kerberos, NTLM, ACLs, ADCS) por trás de cada técnica.

> **O loop de F9:** SharpHound/NetExec coletam e o BloodHound desenha o grafo → você identifica o caminho mais curto até DA → executa cada abuso (roasting, relay, ACL, delegação, ADCS) com a ferramenta certa (Impacket/Rubeus/Certipy) → valida o acesso → documenta a **attack path**. IA ajuda a explicar uma aresta do BloodHound ou um erro de Kerberos; **a decisão e a execução são suas**.

<details>
<summary>📚 <strong>Estuda</strong> — fundamentos de AD, autenticação, e cada família de ataque</summary>

**1. Fundamentos de AD (sem isto, o resto é decoreba)**
- Domínio, árvore, **floresta**, OUs, **GPOs**, sites; objetos: usuários, grupos, computadores; **Domain Controllers**; SYSVOL/NETLOGON; DNS integrado ao AD; **FSMO roles**; **trusts** (intra/inter-floresta).
- Grupos privilegiados: Domain Admins, Enterprise Admins, Administrators, **Account Operators**, **Backup Operators**, **DnsAdmins**, etc.
- **SPN** (Service Principal Name), **ACL/DACL/ACE** (o que permite ACL abuse), **SID**, **RID**.
- Referência: [The Hacker Recipes — AD](https://www.thehacker.recipes/ad/), [adsecurity.org (Sean Metcalf)](https://adsecurity.org), [HackTricks — AD Methodology](https://book.hacktricks.wiki).

**2. Autenticação: NTLM vs Kerberos (o conceito-base de quase todo ataque)**
- **NTLM:** challenge-response; de onde saem o NetNTLMv1/v2 (capturáveis e relayáveis) e o NT hash (usado em Pass-the-Hash).
- **Kerberos:** AS-REQ/AS-REP, TGT, TGS, KDC; o que são **AS-REP roasting** (usuários sem preauth), **Kerberoasting** (tickets de serviço para SPNs), **delegação**, **PAC**.
- Por que isto importa: cada ataque abaixo explora uma propriedade específica de NTLM ou Kerberos.

**3. Enumeração (a parte "máquina" — coleta o grafo)**
- **BloodHound CE** + coletores **SharpHound** (.exe/.ps1) ou **bloodhound-python** (do Linux). Coleta usuários, grupos, sessões, ACLs, GPOs, trusts, e desenha caminhos.
- **NetExec (nxc)**: enumera SMB/LDAP/WinRM/MSSQL em massa, lista usuários, políticas de senha, shares, valida credenciais pelo domínio inteiro.
- **PowerView/SharpView** (no host Windows), **ldapsearch/ldapdomaindump/windapsearch** (do Linux), **adidnsdump** (DNS do AD).
- **PingCastle** / **ADRecon**: visão "de saúde" do AD que revela achados rápidos.

**4. Acesso inicial / coleta de credenciais (do "nada" ao primeiro hash)**
- **LLMNR / NBT-NS / mDNS poisoning** com **Responder** → captura NetNTLMv2 → cracka offline (hashcat) ou faz relay.
- **mitm6** (DHCPv6/DNS IPv6) → posição de MITM em redes que esquecem o IPv6.
- **NTLM relay** com **ntlmrelayx** (Impacket): relay para SMB (exec), **LDAP/LDAPS** (RBCD, adicionar computador), e **ADCS** (ESC8). Combine com coerção (ver abaixo).
- **Password spraying** com **Kerbrute** / nxc (uma senha comum contra muitos usuários — cuidado com lockout).
- **AS-REP roasting** (`GetNPUsers.py`) e **Kerberoasting** (`GetUserSPNs.py`) → crack offline.

**5. Movimento lateral**
- **Pass-the-Hash / Pass-the-Ticket / Overpass-the-Hash** (Impacket, Rubeus, Mimikatz).
- Execução remota: `psexec.py`, `wmiexec.py`, `smbexec.py`, `dcomexec.py`, **`evil-winrm`**, `atexec.py`.
- Dump de credenciais local: **Mimikatz** (`sekurlsa::logonpasswords`), `secretsdump.py` (SAM/LSA/NTDS), LSASS dump (com cuidado de EDR).

**6. Escalonamento de privilégio dentro do domínio (⭐ onde o BloodHound paga)**
- **ACL/ACE abuse** (arestas do BloodHound): `GenericAll`, `GenericWrite`, `WriteDACL`, `WriteOwner`, `AddMember`, `ForceChangePassword`, `AddSelf`. Ex.: `GenericAll` sobre um usuário → trocar a senha dele; sobre um grupo → se adicionar.
- **Delegação Kerberos:**
  - **Unconstrained** → capturar TGT de quem se conecta (incl. DC via coerção).
  - **Constrained** (S4U2Proxy) → abusar de serviços permitidos.
  - **Resource-Based Constrained Delegation (RBCD)** → se você pode escrever `msDS-AllowedToActOnBehalfOfOtherIdentity`, vira comprometimento (combo clássico com relay).
- **Ataques de coerção** (forçam um host a autenticar em você → relay/unconstrained): **PetitPotam** (MS-EFSRPC), **PrinterBug/SpoolSample** (MS-RPRN), **DFSCoerce**, **Coercer** (faz tudo).
- **GPO abuse**, **LAPS** (ler senha de admin local se você tem permissão), **gMSA** (ler `msDS-ManagedPassword`).

**7. ⭐ ADCS — Active Directory Certificate Services (o vetor moderno mais quente)**
- Misconfigurações de template de certificado = caminho direto a DA. Enumere e explore com **Certipy** (ou **Certify**/Certutil).
- **ESC1–ESC8+**: ESC1 (template permite SAN arbitrário → pedir cert como DA), ESC8 (relay HTTP para o endpoint de inscrição), entre outros.
- É frequentemente o caminho **mais limpo e confiável** até Domain Admin em AD moderno. Estude o paper "[Certified Pre-Owned](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf)" da SpecterOps.

**8. Dominância de domínio & persistência**
- **DCSync** (`secretsdump -just-dc` / Mimikatz `lsadump::dcsync`) → puxa o hash de qualquer usuário, inclusive `krbtgt`.
- **Golden Ticket** (forja TGT com o hash do krbtgt), **Silver Ticket** (forja TGS para um serviço), **Diamond/Sapphire Ticket** (variações furtivas), **Skeleton Key**, **DSRM**, **AdminSDHolder**, abuso de **certificados** (persistência via cert válido por anos).
- ⚠️ Persistência só em lab/engajamento autorizado, documentada, e **removida** ao fim.

**9. Trusts (multi-domínio / multi-floresta)**
- Abuso intra-floresta (escalada child → parent via SID history / golden ticket com SID de Enterprise Admins) e cross-forest (trust key, foreign group membership). Avançado — depois do básico sólido.

**10. Hybrid (a ponte para a F10 — Cloud)**
- AD on-prem hoje quase sempre está sincronizado com **Entra ID (Azure AD)** via **Azure AD Connect**. Comprometer o on-prem frequentemente abre caminho para a nuvem (e vice-versa). Isto conecta diretamente com o arquivo `14`.

</details>

<details>
<summary>🧪 <strong>Pratica</strong> — arsenal, comandos de referência e LABS de AD</summary>

**Arsenal de AD (no Kali/atacante Linux + ferramentas Windows/.NET):**
```bash
# Impacket — a caixa de ferramentas nº1 de AD a partir do Linux
pipx install impacket
#  → GetNPUsers.py, GetUserSPNs.py, secretsdump.py, ntlmrelayx.py,
#    psexec.py, wmiexec.py, smbexec.py, getST.py, ticketer.py, addcomputer.py ...

# NetExec (CME successor)
pipx install git+https://github.com/Pennyw0rth/NetExec

# BloodHound CE (via Docker) + coletor python
#  github.com/SpecterOps/BloodHound  (docker compose up)
pipx install bloodhound        # bloodhound-python (coletor remoto)

# Responder, mitm6, Coercer, Certipy, Kerbrute
sudo apt install -y responder
pipx install mitm6
pipx install coercer
pipx install certipy-ad
#  Kerbrute: baixe binário em github.com/ropnop/kerbrute/releases

# Evil-WinRM (shell WinRM)
gem install evil-winrm

# Ferramentas .NET/PowerShell (rodam no host Windows comprometido):
#  Rubeus, Certify, PowerView/SharpView, Seatbelt, SharpHound, mimikatz
#  → github.com/GhostPack, github.com/PowerShellMafia/PowerSploit
```

**Fluxo de referência (LAB de AD autorizado — ex.: GOAD):**
```bash
# 0) Já com um foothold/credencial de baixo privilégio no lab:

# 1) COLETA para o BloodHound (do Linux)
bloodhound-python -d dominio.local -u user -p 'Senha123' -ns 10.10.10.10 -c all
#   → importe o .zip no BloodHound CE e procure "shortest path to Domain Admins"

# 2) Enumeração rápida pelo domínio
nxc smb 10.10.10.0/24 -u user -p 'Senha123' --shares --users
nxc ldap dc.dominio.local -u user -p 'Senha123' --password-policy

# 3) AS-REP roasting + Kerberoasting (crack offline com hashcat)
GetNPUsers.py dominio.local/ -usersfile users.txt -no-pass -dc-ip 10.10.10.10
GetUserSPNs.py dominio.local/user:'Senha123' -dc-ip 10.10.10.10 -request

# 4) Poisoning + relay (em rede de lab)
sudo responder -I eth0           # captura NetNTLMv2
#   (ou) coerção + relay para LDAP → RBCD:
ntlmrelayx.py -t ldaps://dc.dominio.local --delegate-access  # combinar com Coercer/PetitPotam

# 5) ADCS (Certipy) → encontrar e explorar template vulnerável
certipy find -u user@dominio.local -p 'Senha123' -dc-ip 10.10.10.10 -vulnerable
#   ESC1 exemplo: pedir cert com SAN de admin → autenticar com o cert

# 6) Movimento lateral / dump (com hash obtido)
psexec.py dominio.local/user@10.10.10.20 -hashes :NThash
secretsdump.py dominio.local/user@10.10.10.20 -hashes :NThash

# 7) Dominância (após chegar a privilégio de replicação)
secretsdump.py -just-dc dominio.local/admin@dc.dominio.local   # DCSync
```

**LABS de AD — onde você realmente vira perigoso (de graça e pago):**

| Lab | O que é | Custo |
|---|---|---|
| **[GOAD](https://github.com/Orange-Cyberdefense/GOAD)** (Game of Active Directory) | A **melhor** floresta AD vulnerável self-hosted (multi-domínio, todos os ataques). Variações: GOAD-Light, NHA, SCCM | Grátis (seu hypervisor) |
| **[vulnerable-AD](https://github.com/WazeHell/vulnerable-AD)** / **[BadBlood](https://github.com/davidprowe/BadBlood)** | Scripts que populam/vulnerabilizam um DC seu rapidamente | Grátis |
| [TryHackMe — trilha AD](https://tryhackme.com) | **Active Directory Basics**, **Attacking Kerberos**, **Breaching AD**, **Exploiting AD**, **Persisting AD**; redes **Throwback**/**Wreath**/**Holo** | Freemium |
| [HTB Academy](https://academy.hackthebox.com) | Módulos de AD + **Active Directory Enumeration & Attacks**; job-role **CPTS** | Pago |
| [HTB Pro Labs](https://www.hackthebox.com/hacker/pro-labs) | **Dante**(intro) → **Offshore**, **RastaLabs**, **Cybernetics**, **APTLabs** (florestas enterprise realistas) | Pago |
| [Altered Security](https://www.alteredsecurity.com) | Labs oficiais das certs **CRTP/CRTE/CRTO-equivalentes** (AD do básico ao avançado) | Pago |

> 🎯 **Comece pelo GOAD.** É gratuito, completo e cobre todos os ataques desta fase. Suba no Proxmox/VMware (ou use **[Ludus](https://docs.ludus.cloud)**, que tem template do GOAD pronto). Quando fechar o GOAD, vá para um Pro Lab do HTB ou direto para o lab do **CRTP**.

**IA como copiloto em AD (regra do `01`/`10`):**
- Cole a descrição de uma **aresta do BloodHound** (ex.: `WriteDACL` sobre um grupo) e peça: *"como abuso disso passo a passo, e com que ferramenta?"*
- Erro de Kerberos críptico (skew de relógio, `KRB_AP_ERR_*`): peça explicação.
- Peça para a IA **resumir um caminho de ataque** do BloodHound em linguagem de report.
- 🔒 **Limite duro:** nomes de domínio reais de cliente, hashes, dumps de NTDS, output de engajamento sob NDA → **modelo local (Ollama)**. Lab/conceito público → nuvem OK. **Valide tudo** — IA alucina nomes de flag e sintaxe de ferramenta.

</details>

<details>
<summary>✍️ <strong>Posta</strong> — AD writeups valem ouro para recrutador</summary>

- Writeup de **comprometimento do GOAD** (ou de um Pro Lab retirado, respeitando regras): conte a **attack path** completa, com o grafo do BloodHound, decisões e por quê. Isto demonstra exatamente a competência que a vaga procura.
- Posts conceituais: *"Kerberoasting explicado de verdade"*, *"ADCS ESC1 do zero"*, *"por que NTLM relay ainda funciona em 2026"*. Ensinar prova domínio.
- Mantenha um cheat-sheet de AD no `notes` (comandos Impacket/nxc/Certipy que você sempre esquece).

</details>

<details>
<summary>📖 <strong>Writeups da semana</strong></summary>

Leia writeups de **AD/red team interno** e research de ponta. Fontes: [SpecterOps blog](https://specterops.io/blog/), [The Hacker Recipes](https://www.thehacker.recipes), [ired.team](https://www.ired.team), [dirkjanm.io](https://dirkjanm.io) (relay/Azure), [Posts do Elad Shamir sobre delegação]. Pergunte sempre: que propriedade do AD foi abusada, e eu teria reconhecido?

</details>

### 🎯 Missão F9 — "Do zero a Domain Admin por 3 caminhos"
Comprometa um **lab de AD completo (GOAD recomendado)** do usuário de baixo privilégio até **Domain Admin / Enterprise Admin**, usando **pelo menos 3 caminhos de ataque distintos** (ex.: Kerberoasting+crack, ACL abuse via BloodHound, ADCS ESC1, relay→RBCD). Escreva a attack-path.

**Critério de aceitação:** writeup/relatório documentando 3+ caminhos independentes até o controle do domínio, com o grafo do BloodHound e os comandos de cada elo (em ambiente de lab).

### ✅ Checklist F9
- [ ] Fundamentos de AD claros (floresta, GPO, SPN, ACL, FSMO, trusts)
- [ ] Entende NTLM vs Kerberos e de onde vem cada ataque
- [ ] BloodHound CE + SharpHound/bloodhound-python: coleta e leitura de caminhos
- [ ] NetExec para enumeração/validação pelo domínio
- [ ] Responder/mitm6 + **ntlmrelayx** (relay para SMB/LDAP/ADCS)
- [ ] AS-REP roasting + Kerberoasting + crack offline (hashcat)
- [ ] Password spraying com controle de lockout (Kerbrute/nxc)
- [ ] PtH / PtT / OtH e execução remota (psexec/wmiexec/evil-winrm)
- [ ] **ACL abuse** (GenericAll/WriteDACL/etc.) na prática
- [ ] **Delegação** (unconstrained/constrained/**RBCD**) entendida e explorada
- [ ] **Coerção** (PetitPotam/PrinterBug/Coercer) → relay
- [ ] **ADCS** com Certipy (ESC1/ESC8 no mínimo)
- [ ] DCSync + Golden/Silver Ticket (em lab)
- [ ] GOAD comprometido ponta a ponta + writeup
- [ ] (Meta) **CRTP** no horizonte (ver `16`)

---

➡️ **Próximo:** `14-CLOUD-E-CONTAINERS-F10.md` — onde a infra moderna migrou, e onde híbrido AD→Azure fecha o ciclo.
