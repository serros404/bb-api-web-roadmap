# 12 — Rede & Infraestrutura (F8)

> **Por que esta fase existe:** o roadmap original (web/API) te faz um excelente caçador de aplicação — mas a vaga de **pentest/red team CLT** quase sempre pede **pentest interno de rede**. Cliente contrata para responder: *"se um atacante cair na minha rede (phishing, insider, VPN vazada), até onde ele vai?"* Web é uma porta; **rede + AD + cloud é a casa inteira**. Esta é a primeira das frentes novas (F8 → F9 → F10 → F11).

> **A mesma tese, agora em infra:** *amplitude por máquina, profundidade por humano.* Scanner enumera 65k portas em centenas de hosts em minutos (amplitude); **exploração, escalonamento de privilégio e pivoting são raciocínio humano** (profundidade). Nenhum scanner pivota sozinho de um host comprometido para o resto da rede — isso é você.

> ⚠️ **Ética (releia o `00`):** as técnicas aqui são **muito mais destrutivas** que as de web. Pentest de rede roda **exclusivamente** em (1) seus próprios labs, (2) engajamentos contratados com escopo e *Rules of Engagement* assinados. Acesso a rede de terceiros sem contrato é crime (Lei 12.737/2012, CFAA). **Exploit em produção pode derrubar serviço** — em engajamento real você combina janela, evita DoS e tem botão de pânico. Nada disto se testa "no alvo pra ver".

---

# 🌐 F8 — Pentest de Rede & Infraestrutura

**Duração:** 2–4 meses · **Carga:** ~15–20h/sem · **Badge:** 🌐 *Operador de Rede*

**Foco:** sair do navegador. Você vai aprender a **enumerar uma rede inteira**, identificar e explorar serviços, **escalar privilégio** em Linux e Windows, e — o mais importante para o trabalho real — **pivotar** de um host para o resto da rede. É a base sobre a qual o Active Directory (F9) e o Red Team (F11) se apoiam.

> **O loop de F8:** descoberta automatizada (nmap/autorecon) entrega a superfície → você escolhe o serviço/host mais promissor → explora manualmente → ganha o primeiro shell → enumera privesc (LinPEAS/WinPEAS) → escala → **pivota** para a próxima rede → repete. IA é copiloto (parsear output, explicar um serviço desconhecido, gerar script de enum), **nunca** o piloto.

<details>
<summary>📚 <strong>Estuda</strong> — fundamentos de rede ofensiva, enumeração, exploração, privesc e pivoting</summary>

**1. Fundamentos de rede para o atacante (revise o que já viu na F1, com lente ofensiva)**
- Modelo TCP/IP, portas/serviços comuns (21 FTP, 22 SSH, 25/465/587 SMTP, 53 DNS, 80/443 HTTP, 88 Kerberos, 110/143 POP/IMAP, 111 RPC, 135/139/445 SMB/RPC, 161 SNMP, 389/636 LDAP/LDAPS, 1433 MSSQL, 3306 MySQL, 3389 RDP, 5432 PostgreSQL, 5985/5986 WinRM).
- Subnetting/CIDR (saber que `10.10.10.0/24` são 254 hosts), roteamento, NAT, VLANs, firewalls/segmentação — você precisa **ler** a topologia para saber por onde pivotar.
- TCP vs UDP (muita gente esquece de scanear UDP — SNMP, DNS, TFTP vivem lá).
- Referência: [TryHackMe — Network Fundamentals](https://tryhackme.com), [HackTricks — Pentesting Network](https://book.hacktricks.wiki).

**2. Descoberta e varredura (a parte "máquina")**
- **nmap** a fundo: host discovery (`-sn`), TCP SYN (`-sS`), version/service detection (`-sV`), OS detection (`-O`), scripts NSE (`-sC`, `--script`), portas (`-p-` para todas), timing/evasão (`-T`, `--max-rate`, fragmentação). Aprenda a **ler** o output, não só rodar.
- **Velocidade:** `masscan` e `rustscan` para varrer faixas grandes rápido (depois passa nmap `-sV` só nas portas abertas).
- **naabu** (já está na sua suíte ProjectDiscovery) integra bem ao seu pipeline.

**3. Enumeração de serviços (o que separa quem acha shell de quem não acha)**
A regra de ouro do pentest interno: **enumere até doer**. 80% do trabalho é enumeração; exploit é o final.
- **SMB (445):** o serviço mais rico em rede Windows. `enum4linux-ng`, `smbmap`, `smbclient`, `rpcclient`, `nxc smb` (NetExec). Procure shares legíveis, null sessions, versões vulneráveis (EternalBlue/MS17-010 em labs).
- **SNMP (161 UDP):** `snmpwalk`, `onesixtyone`, `snmp-check`. Community string `public` vaza um mapa inteiro (usuários, processos, rotas).
- **LDAP (389):** `ldapsearch` anônimo às vezes despeja todo o diretório (ponte para a F9).
- **NFS (2049):** `showmount -e`, montar shares mal configurados.
- **FTP/SSH/RDP/WinRM/bancos:** banners, default creds, versões.
- **DNS (53):** transferência de zona (`dig axfr`), enumeração de subdomínios internos.
- Referência: [HackTricks](https://book.hacktricks.wiki) tem uma página por porta — é a bíblia da enumeração.

**4. Identificação de vulnerabilidades**
- Scanners: **Nessus Essentials** (grátis até 16 IPs), **OpenVAS/Greenbone**, **nuclei** (você já domina). Eles dão amplitude; você valida.
- `searchsploit` (Exploit-DB local) para casar versão → exploit conhecido.
- **Sempre valide a versão** antes de jogar exploit — versão errada derruba serviço sem te dar nada.

**5. Exploração**
- **Metasploit Framework:** `msfconsole`, busca de módulos, `meterpreter`, `msfvenom` para gerar payloads. Aprenda, mas **não vire dependente** — o OSCP e os bons engajamentos exigem exploração manual.
- **Exploração manual:** ler um PoC público, adaptar, entender o que faz. É isso que prova senioridade.
- Classes comuns em infra: serviços desatualizados, default credentials, RCE em apps internas, deserialização, file upload em painéis internos.

**6. Escalonamento de privilégio — Linux**
- Metodologia de enumeração: **LinPEAS**, `linux-smart-enumeration` (lse.sh), `pspy` (monitora processos/cron sem root).
- Vetores: SUID/SGID mal configurado (cruze com [GTFOBins](https://gtfobins.github.io)), `sudo -l` (regras frouxas), cron jobs com path/permissão fraca, capabilities, kernel exploits (último recurso — pode dar pânico), credenciais em arquivos/histórico, montagens, NFS root squash.
- Referência: [HackTricks — Linux Privesc](https://book.hacktricks.wiki), [g0tmi1k checklist](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/).

**7. Escalonamento de privilégio — Windows**
- Enumeração: **WinPEAS**, **PrivescCheck.ps1**, **PowerUp.ps1**, **Seatbelt**, **SharpUp**, `accesschk`.
- Vetores: serviços com binário/permissão fraca, **unquoted service path**, `AlwaysInstallElevated`, **token impersonation / Potato attacks** (JuicyPotato, PrintSpoofer, RoguePotato, GodPotato), credenciais em registro/GPP/arquivos, tarefas agendadas, DLL hijacking, [LOLBAS](https://lolbas-project.github.io).
- `WES-NG` (Windows Exploit Suggester) e `Watson` para faltas de patch.
- Referência: [HackTricks — Windows Privesc](https://book.hacktricks.wiki), [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings).

**8. Pivoting & tunneling (⭐ a habilidade que o pentest interno exige e o CTF não ensina)**
- Conceito: comprometeu um host com duas interfaces? Ele é sua ponte para a rede que você não alcança.
- **Ligolo-ng** (o moderno, cria uma interface tun limpa — aprenda este primeiro), **chisel**, **sshuttle**, túneis SSH (`-L` local, `-R` remoto, `-D` dynamic/SOCKS), **proxychains**, `socat`, port forwarding nativo do Windows (`netsh`), `autoroute` do Metasploit.
- Referência: [Ligolo-ng](https://github.com/nicocha30/ligolo-ng), [The Hacker Recipes — Pivoting](https://www.thehacker.recipes).

**9. Post-exploitation básico**
- Looting (credenciais, chaves, configs), harvesting de hashes, conceitos de persistência (aprofundados em F9/F11). Sempre com **impacto mínimo** e dentro do escopo.

</details>

<details>
<summary>🧪 <strong>Pratica</strong> — instalação, comandos de referência e LABS (onde você de fato aprende)</summary>

**Arsenal de infra (instale no seu Kali/Ubuntu de ataque):**
```bash
# Kali já traz a maioria. Em Ubuntu/Debian, instale o essencial:
sudo apt update && sudo apt install -y nmap masscan smbclient smbmap \
  enum4linux snmp onesixtyone hydra medusa john hashcat proxychains4 metasploit-framework

# NetExec (sucessor do CrackMapExec) — canivete suíço de rede/SMB/WinRM
pipx install git+https://github.com/Pennyw0rth/NetExec

# rustscan (scan rápido)
# baixe o .deb do release oficial: github.com/RustScan/RustScan

# Ligolo-ng (pivoting) — baixe agent + proxy dos releases
# github.com/nicocha30/ligolo-ng/releases

# PEASS-ng (LinPEAS/WinPEAS) e ferramentas de enum
# github.com/peass-ng/PEASS-ng/releases   (linpeas.sh, winPEASx64.exe)
git clone https://github.com/peass-ng/PEASS-ng
```

**Fluxo de referência (LAB autorizado):**
```bash
# 1) Descoberta de hosts vivos numa faixa de LAB
nmap -sn 10.10.10.0/24 -oA hosts_vivos

# 2) Scan rápido de todas as portas, depois -sV/-sC só nas abertas
rustscan -a 10.10.10.10 --ulimit 5000 -- -sV -sC -oA full_10.10.10.10
# (ou puro nmap)
nmap -p- --min-rate 2000 -T4 10.10.10.10 -oA portas
nmap -p <portas_abertas> -sV -sC -oA detalhe 10.10.10.10

# 3) Enumeração SMB
nxc smb 10.10.10.10 -u '' -p '' --shares          # null session
enum4linux-ng -A 10.10.10.10
smbmap -H 10.10.10.10 -u guest

# 4) SNMP
snmpwalk -v2c -c public 10.10.10.10

# 5) Identificar exploit conhecido por versão
searchsploit <servico versao>

# 6) Privesc Linux (já com um shell no alvo do lab)
./linpeas.sh | tee linpeas.out
sudo -l ; find / -perm -4000 -type f 2>/dev/null    # SUID

# 7) Pivoting com Ligolo-ng (esquema: você compromete host-A com 2 redes)
#  - no SEU host:  ./proxy -selfcert
#  - no host-A:    ./agent -connect SEU_IP:11601
#  - no proxy:     session > start  → cria interface ligolo; adicione rota para a 2ª rede
```

**LABS — a parte que importa de verdade (pratica, pratica, pratica):**

| Plataforma | O que praticar | Custo |
|---|---|---|
| [TryHackMe](https://tryhackme.com) | Trilhas **Complete Beginner**, **Jr Penetration Tester**, **Offensive Pentesting**. Salas: Vulnversity, Kenobi, Blue (EternalBlue), Steel Mountain, Alfred. Redes com pivoting: **Wreath** e **Throwback** | Freemium / barato |
| [HackTheBox](https://www.hackthebox.com) | **Starting Point** → máquinas retiradas (com writeups oficiais) → **HTB Academy** (job-role path *Penetration Tester* / cert **CPTS**) | Freemium / VIP |
| [HTB Pro Labs](https://www.hackthebox.com/hacker/pro-labs) | **Dante** = rede multi-host com pivoting (o melhor primeiro Pro Lab) | Pago |
| [Proving Grounds](https://www.offsec.com/labs/) | **Play** (grátis, espelha VulnHub) e **Practice** (pago, estilo OSCP) | Free/Pago |
| [VulnHub](https://www.vulnhub.com) | VMs vulneráveis para baixar e quebrar offline (de graça, no seu hypervisor) | Grátis |

> 🎯 **Monte um home lab:** VirtualBox/VMware (grátis) ou **Proxmox**. Suba seu Kali de ataque + VMs vulneráveis do VulnHub. Para redes/pivoting sem gastar, **[Ludus](https://docs.ludus.cloud)** automatiza ranges inteiros. É o lab que você usa o resto da carreira.

**IA como copiloto em infra (regra do `01`/`10`):**
- Cole **output de nmap** e peça priorização: *"quais serviços/portas têm maior chance de RCE ou má configuração? por onde começo?"*
- Caiu num serviço/stack que não conhece: *"explique o modelo de auth deste serviço e as classes de bug históricas dele."*
- Peça um **script de enumeração** sob medida (varrer N hosts, comparar respostas).
- 🔒 **Limite:** dados de cliente (IPs internos reais, credenciais, output bruto de engajamento sob NDA) → **modelo local (Ollama)**, nunca nuvem. Conceito/lab público → nuvem OK. E **tudo é hipótese até você validar**.

</details>

<details>
<summary>✍️ <strong>Posta</strong> — alimente o pipeline público (o que te contrata)</summary>

- Writeups de máquinas **retiradas** do HTB e de salas do THM (permitido escrever sobre as retiradas/concluídas). Foque na **metodologia de enumeração e na decisão de pivoting**, não só no exploit.
- Um post sobre **"meu home lab de pentest interno"** (Proxmox + Ludus + GOAD na F9) — mostra maturidade e atrai recrutador.
- Repositório `notes` (seu segundo cérebro): cheat-sheet de enumeração por porta, comandos de pivoting que você usa sempre.

</details>

<details>
<summary>📖 <strong>Writeups da semana (3–5, inegociável)</strong></summary>

Agora além de bug bounty, leia **writeups de pentest interno e máquinas HTB/THM**. Anote: como o autor enumerou? qual foi o pivô? qual técnica de privesc? eu teria visto isso? Boas fontes: [0xdf](https://0xdf.gitlab.io) (writeups HTB de altíssima qualidade), [IppSec](https://www.youtube.com/@ippsec) (vídeo + a engine de busca [ippsec.rocks](https://ippsec.rocks)).

</details>

### 🎯 Missão F8 — "Rede comprometida ponta a ponta"
Comprometa uma **rede multi-host em lab** do zero ao controle total, **incluindo pelo menos um pivô** (host → rede interna), e escreva o report. Sugestão de entregável: **HTB Pro Lab Dante** ou as redes **Wreath/Throwback** do THM, completas.

**Critério de aceitação:** você documenta a cadeia inteira — foothold inicial → privesc → pivoting → host seguinte → objetivo final — com um report técnico legível (use o modelo do `08`, adaptado a infra).

### ✅ Checklist F8
- [ ] nmap dominado (descoberta, `-sV`/`-sC`, NSE, `-p-`, UDP, timing)
- [ ] Enumeração profunda de SMB/SNMP/LDAP/NFS/serviços comuns
- [ ] Valida versão → exploit (searchsploit) antes de disparar
- [ ] Exploração manual **e** com Metasploit/meterpreter
- [ ] Password attacks (hydra/medusa + spraying) e cracking offline (hashcat/john)
- [ ] Privesc Linux (LinPEAS + GTFOBins, SUID/sudo/cron/cap)
- [ ] Privesc Windows (WinPEAS/PowerUp, serviços, Potato, LOLBAS)
- [ ] **Pivoting** com Ligolo-ng/chisel/proxychains dominado
- [ ] Home lab montado (hypervisor + VMs vulneráveis)
- [ ] 1 rede multi-host de lab comprometida ponta a ponta + report
- [ ] (Meta) progresso na trilha **eJPT → PNPT/CPTS → OSCP** (ver `16`)

---

➡️ **Próximo:** `13-ACTIVE-DIRECTORY-F9.md` — onde 95% das redes corporativas vivem, e onde o red team realmente brilha.
