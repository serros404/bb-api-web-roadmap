# 15 — Red Team Ops (F11)

> **Por que esta fase é o que "fecha" o título de red teamer:** pentest responde *"o que está vulnerável?"*. **Red team** responde *"um adversário real conseguiria atingir um objetivo de negócio sem ser detectado, e a equipe de defesa pegaria?"*. A diferença é **furtividade, objetivo e simulação de ameaça** — não é achar todos os bugs, é emular um atacante específico contra uma organização que está se defendendo ativamente (Blue Team/SOC/EDR). Esta é a camada mais avançada e a mais sensível eticamente — faça-a **depois** de dominar F8/F9/F10.

> **A tese, no red team:** *amplitude por máquina, profundidade por humano* assume sua forma final. Automação faz recon, gera infraestrutura (C2/redirectors via IaC), dispara enumeração; mas **a estratégia do engajamento, a narrativa de ataque, as decisões de OPSEC e a emulação de TTPs de um adversário são puro raciocínio humano**. Nenhuma ferramenta decide "vale a pena fazer barulho aqui para atingir o objetivo, ou espero?".

> ⚠️ **Ética — leia com atenção (e releia o `00`):** esta fase trata de **evasão de defesa e operação adversária**. Tudo aqui é **exclusivamente** para (1) seu próprio lab, (2) engajamentos de red team contratados com Rules of Engagement formais, autorização escrita do nível executivo, janelas, deconfliction e *get-out-of-jail letter*. **Este material descreve conceitos, ferramentas públicas e caminhos de estudo — não escreve malware, packers ou bypasses de EDR prontos.** O valor de aprender evasão é **entender a defesa** e operar com OPSEC que **protege o cliente** (não vazar dados, não deixar backdoors, limpar tudo). Operar técnicas de red team fora de autorização explícita é crime grave. Em red team profissional, **proteger o cliente é parte do trabalho** — você simula o adversário sem se tornar um.

---

# 🥷 F11 — Operações de Red Team

**Duração:** contínuo / 3–6 meses de imersão · **Carga:** flexível · **Badge:** 🥷 *Operador Red Team*

**Foco:** entender o ciclo de uma operação adversária emulada — **acesso inicial → C2 → evasão → ações no objetivo → exfil simulada → relatório com narrativa e mapeamento ATT&CK** — e a disciplina de **OPSEC** que separa um profissional de um vândalo. O foco é **conceitual e de laboratório**: você aprende as TTPs para emulá-las com responsabilidade e para entender como a defesa as detecta.

> **O loop de F11 (assumed breach é o mais comum no mercado):** o cliente te dá um foothold (simula phishing bem-sucedido) → você estabelece C2 furtivo → enumera evitando detecção → executa a attack path de AD/cloud (F9/F10) com OPSEC → atinge o objetivo (acessar o "crown jewel" combinado) → documenta cada TTP, o que o Blue Team viu e o que não viu. **Purple teaming** (red+blue colaborando) é cada vez mais o formato — e ótimo para aprender.

<details>
<summary>📚 <strong>Estuda</strong> — ciclo adversário, C2, evasão (conceitual), OPSEC e frameworks</summary>

**1. Frameworks mentais (a linguagem do red team)**
- **MITRE ATT&CK** — o dicionário de TTPs (Tactics, Techniques, Procedures). Red teamer **pensa e reporta em ATT&CK**. Estude as táticas: Initial Access, Execution, Persistence, Privilege Escalation, Defense Evasion, Credential Access, Discovery, Lateral Movement, Collection, C2, Exfiltration, Impact.
- **Cyber Kill Chain** (Lockheed Martin) e **Unified Kill Chain**.
- **Threat-informed / adversary emulation:** emular um grupo específico (ex.: usar o [MITRE ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/) e relatórios de threat intel para reproduzir TTPs de um APT relevante ao setor do cliente).
- Frameworks de engajamento: **TIBER-EU**, **CBEST**, e o conceito de **RoE / scoping**.
- Referência: [MITRE ATT&CK](https://attack.mitre.org), [Red Team Development and Operations (Vincent/Cervini)](https://redteam.guide), [Red Team Field Manual (RTFM)].

**2. Acesso inicial & engenharia social (conceitual + autorizado)**
- **OSINT de alvo** para pretexting (você já tem base no recon): funcionários, e-mails, tecnologias, fornecedores.
- **Phishing autorizado:** **GoPhish** (campanhas de teste), **Evilginx2** (proxy reverso para AiTM/captura de sessão e bypass de MFA — **só em engajamento autorizado**).
- Vetores de payload (conceito; o desenvolvimento fica nas referências de MalDev): documentos com macro, LNK, ISO/IMG, HTA, containers. O foco de estudo é **como a defesa detecta** cada um.
- ⚠️ Engenharia social contra pessoas reais **só** com autorização explícita e escopo claríssimo (impacto humano é sério).

**3. Command & Control (C2)**
- Conceitos: **beacon/implant**, **listener**, **redirectors**, **malleable profiles**, jitter/sleep, domain fronting, canais (HTTPS/DNS/SMB).
- Frameworks **open-source** (para aprender em lab): **[Sliver](https://github.com/BishopFox/sliver)** (BishopFox — ótimo para começar), **[Mythic](https://github.com/its-a-feature/Mythic)**, **[Havoc](https://github.com/HavocFramework/Havoc)**, **[Empire/Starkiller](https://github.com/BC-SECURITY/Empire)**, **Covenant**.
- Framework **comercial** padrão de mercado: **Cobalt Strike** (você verá em quase toda vaga de red team; o curso **CRTO** ensina com ele).
- **Infra de operação:** redirectors (Nginx/Apache mod_rewrite), categorização de domínio, certificados — montados com IaC (Terraform/Ansible). Aprender a **construir e descartar** infra é parte do ofício.

**4. Evasão de defesa (conceitual — entenda para emular e para detectar)**
> Aqui o estudo é sobre **mecanismos** e **detecção**, não sobre escrever bypasses prontos. O objetivo profissional é saber por que algo é pego e operar com OPSEC.
- **Telemetria que te pega:** EDR/AV, **AMSI**, **ETW**, Sysmon, logs de Kerberos/LDAP, Windows Event Logs, defensive analytics.
- Conceitos de evasão (estudados em cursos como OSEP/CRTO/MalDev): execução in-memory, *process injection*, redução de IOCs, **LOLBAS/LOLBins** (viver da terra), obfuscação, *unhooking*, BYOVD — **estudados para compreensão e detecção**, não fornecidos como código aqui.
- **A virada de chave do red team:** muitas vezes o caminho mais OPSEC-safe é **não** usar malware — usar credenciais válidas, LOLBins e funcionalidade legítima do AD/cloud (o que você aprendeu em F9/F10). "Living off the land" frequentemente vence "exploit barulhento".

**5. OPSEC & proteção do cliente (o que define profissionalismo)**
- **Deconfliction** (combinar com o cliente o que é exercício vs incidente real), logging de todas as ações, *cleanup* (remover implants/persistência/contas criadas), proteção dos dados acessados (não exfiltrar PII de verdade — usar canários/dados marcados), comunicação segura com o ponto focal.
- **Assume breach vs full-scope**, **black/grey/white box**, e por que assume-breach é o formato dominante (mais eficiente, foca na detecção).

**6. Relatório de red team (o entregável que vale a contratação)**
- Estrutura: sumário executivo (para a liderança), **narrativa de ataque** cronológica, **mapeamento ATT&CK** de cada passo, achados e recomendações, e **o que a defesa detectou/perdeu** (o valor real para o cliente).
- Diferente do report de bug bounty (`08`): foco em **objetivo de negócio atingido** e em **melhoria de detecção**, não em lista de vulnerabilidades.

**7. Purple teaming & detecção (te torna muito mais valioso)**
- Entender o lado azul multiplica seu valor: **Sigma rules**, **Sysmon**, **Elastic/Wazuh**, **Atomic Red Team** (executa TTPs para validar detecção), **Caldera** (emulação automatizada da MITRE).
- Saber explicar "este é o TTP, esta é a telemetria que ele gera, esta é a regra que o detecta" é exatamente o que diferencia um red teamer sênior.

</details>

<details>
<summary>🧪 <strong>Pratica</strong> — monte o lab adversário e LABS guiados (tudo isolado)</summary>

**O home lab de red team (isolado, sem acesso à internet de produção):**
```text
Monte um ambiente fechado no seu hypervisor (Proxmox/VMware):
  • Floresta AD vulnerável         → GOAD (do arquivo 13)
  • Endpoints Windows com defesa   → Windows + Defender/Sysmon ligados
  • Sensor defensivo (Blue)        → Wazuh ou Elastic + Sysmon + Atomic Red Team
  • Servidor C2 (open-source)      → Sliver/Mythic/Havoc numa VM isolada
  • Automação de provisionamento   → Ludus (templates de GOAD + Windows + ranges)
Objetivo: rodar um TTP, ver no C2, e ver a telemetria no Wazuh. Red + Blue no mesmo lab.
```
- Ferramentas para o lab: **[DetectionLab](https://github.com/clong/DetectionLab)**, **[Ludus](https://docs.ludus.cloud)**, **[Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)**, **[MITRE Caldera](https://github.com/mitre/caldera)**.

**LABS / treinamentos guiados:**

| Recurso | O que é | Custo |
|---|---|---|
| **[HTB Pro Labs](https://www.hackthebox.com/hacker/pro-labs)** | **RastaLabs**, **Cybernetics**, **APTLabs** — ambientes red team com defesa e AD enterprise | Pago |
| **[Zero-Point Security — CRTO](https://training.zeropointsecurity.co.uk)** | Curso + lab de **Red Team Ops com Cobalt Strike**; a cert prática mais reconhecida de RT | Pago |
| **[Altered Security](https://www.alteredsecurity.com)** | CRTP/CRTE (AD) e trilha de **red team / evasão** | Pago |
| [TryHackMe — Red Team path](https://tryhackme.com) | Trilha **Red Teaming** (initial access, C2, evasão, OPSEC) — boa porta de entrada | Freemium |
| [OffSec OSEP (PEN-300)](https://www.offsec.com/courses/pen-300/) | Evasão e pentest avançado (AV bypass, pivoting, AD) | Pago |
| [Vulnerable C2 / labs próprios](https://github.com/BishopFox/sliver/wiki) | Pratique Sliver/Mythic no seu lab fechado | Grátis |
| [MalDev Academy](https://maldevacademy.com) / [Sektor7](https://institute.sektor7.net) | Desenvolvimento de malware/evasão **para fins de pesquisa e entendimento de defesa** | Pago |

> 🎯 **Sequência sugerida:** trilha Red Teaming do THM → montar o home lab (GOAD + C2 open-source + Wazuh) e rodar Atomic Red Team observando a detecção → HTB **RastaLabs** → curso **CRTO** (quando puder investir; é o divisor de águas para vagas de red team).

**IA como copiloto em red team (regra do `01`/`10`):**
- Peça para a IA **explicar uma técnica ATT&CK** e a telemetria que ela gera (ótimo para purple teaming).
- Peça para **estruturar a narrativa de um relatório** de red team a partir dos seus fatos (sem dados reais do cliente).
- Peça para mapear seus passos de lab para **ATT&CK IDs**.
- 🔒 **Limite duro:** nada de dados de cliente, infraestrutura real, ou pedir geração de malware/bypass operacional para uso fora de lab. Conceito/lab/ATT&CK público → OK. Dados sensíveis → **modelo local**. **Valide tudo.**

</details>

<details>
<summary>✍️ <strong>Posta</strong> — o conteúdo que sinaliza "red teamer" para recrutador</summary>

- Posts de **purple team**: *"TTP X — como executei no lab e como o Wazuh/Sysmon detectou"*. Mostra os dois lados e é altamente valorizado.
- Writeup de um **Pro Lab retirado** (RastaLabs/Cybernetics, respeitando regras) com foco em OPSEC e narrativa.
- Conceitual: *"assume breach vs full scope"*, *"living off the land: por que credencial válida vence exploit"*, *"montando um home lab red+blue com GOAD e Wazuh"*.
- ⚠️ **Nunca** publique técnicas/infra de um engajamento real de cliente, nem código operacional de evasão. Conteúdo de lab e conceito, sim.

</details>

<details>
<summary>📖 <strong>Writeups da semana</strong></summary>

Research de red team e detecção: [SpecterOps blog](https://specterops.io/blog/), [Red Canary](https://redcanary.com/blog/), [MDSec](https://www.mdsec.co.uk/knowledge-centre/insights/), [Outflank](https://www.outflank.nl/blog/), [TrustedSec](https://trustedsec.com/blog), o **Red Team Village** e o podcast **Critical Thinking**. Leia threat intel (relatórios de APT) e pense em como emularia aquelas TTPs eticamente.

</details>

### 🎯 Missão F11 — "Operação assume-breach ponta a ponta no seu lab"
No seu **home lab fechado (GOAD + endpoints com Sysmon/Wazuh + C2 open-source)**, conduza uma operação **assume-breach**: estabeleça C2, mova-se até o objetivo (ex.: DA ou um "crown jewel" definido por você) **observando a telemetria defensiva**, e escreva um **relatório de red team** com narrativa cronológica e **mapeamento ATT&CK**, incluindo o que foi detectado.

**Critério de aceitação:** relatório de red team de uma operação de lab — narrativa + ATT&CK por passo + seção de detecção (red e blue) — demonstrando OPSEC e raciocínio de objetivo, tudo em ambiente isolado.

### ✅ Checklist F11
- [ ] Pensa e documenta em **MITRE ATT&CK**
- [ ] Entende adversary emulation e formatos de engajamento (assume breach, etc.)
- [ ] C2 open-source (Sliver/Mythic/Havoc) operado em lab; conhece Cobalt Strike de nome/conceito
- [ ] Infra de operação (redirectors, perfis) compreendida
- [ ] Evasão **conceitual**: AMSI/ETW/EDR/telemetria e por que IOCs são pegos
- [ ] "Living off the land" como estratégia OPSEC-first
- [ ] Phishing autorizado (GoPhish/Evilginx) entendido com limites éticos
- [ ] **OPSEC + proteção do cliente** (deconfliction, cleanup, dados marcados)
- [ ] Home lab **red+blue** montado (GOAD + Sysmon/Wazuh + Atomic Red Team)
- [ ] Purple teaming: executa TTP e observa a detecção
- [ ] Relatório de red team (narrativa + ATT&CK) escrito
- [ ] (Meta) **CRTO** / OSEP no horizonte (ver `16`)

---

➡️ **Próximo:** `16-INDICE-INTEGRADO-CERTS-E-CARREIRA-PENTEST.md` — o mapa que junta tudo (web/API + rede/AD/cloud/RT), a escada de certificações e a trilha de conversão para a vaga de pentest/red team.
