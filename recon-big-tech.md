# 🛰️ Recon em Big Tech — como chegar nas funções pouco usadas

[🏠 Início](README.md) · extensão do [recon minimalista](ROADMAP-MINIMALISTA.md#recon-o-mínimo--domínios-subs-js-apis-keys) para alvos grandes

> Recon em empresa pequena é trivial (2 subs, 1 app). Em **big tech, o recon _é_ o jogo** — a superfície é gigante, escondida atrás de CDN/WAF, e a parte fácil já foi garimpada por centenas de caçadores. O ouro está no **esquecido / legado / interno / raramente usado** que a multidão não achou. Como a [Assetnote](https://www.assetnote.io/resources/blog/the-art-of-recon-strategies-for-modern-asset-discovery) resume: *"achar o ponto fraco costuma ser mais difícil que explorá-lo."*
>
> Aterrado em fontes reais: [Sam Curry — We Hacked Apple for 3 Months](https://samcurry.net/hacking-apple) (55 bugs) e o framework de recon da Assetnote (**breadth · depth · context · amplification · focus**). Tudo para uso **autorizado**, dentro de escopo.

---

## 1 — Por que big tech é outro jogo

- **Escala absurda.** A Apple é dona do bloco **`17.0.0.0/8` inteiro ("Applenet", ~25 mil web servers)** e ~7 mil domínios ([Curry](https://samcurry.net/hacking-apple)). Você não "olha o site"; você **mapeia uma cidade**.
- **CDN/WAF na frente.** Scan de IP na internet toda tem **blind spot**: bate no CDN, não na app. Por isso **enumeração de subdomínio vence scan de IP** — o subdomínio entrega o `Host`/SNI certo pra alcançar a aplicação real (Assetnote).
- **A superfície óbvia está lotada.** O que um scanner acha, 300 pessoas já acharam. Seu diferencial é a **superfície esquecida**: legado, staging, versões antigas de API, painéis internos, o que só o app mobile fala.

## 2 — Amplitude: mapear TUDO que a org tem

**ASN → faixas de IP.** Descubra os ASNs da org, extraia as faixas, suba tudo:
```bash
# ASN e ranges
amass intel -org "Nome da Empresa"           # ASNs ligados ao nome
asnmap -a AS0000 -silent                      # ranges de um ASN  (ou bgp.he.net / bgpview)
# probe + screenshot de tudo que responde
asnmap -a AS0000 -silent | httpx -silent -sc -title -td | tee asn_live.txt
```
> Foi assim que a Apple caiu: painel de admin esquecido num IP **não-listado** dentro do /8. `gowitness`/`aquatone` pra screenshot em massa e triar o "interessante" (admin, staging, erro, login antigo).

**Aquisições = novo território.** Empresa comprou outra? Herdou os domínios, ASNs e servidores **mal-integrados** dela (a "amplification" da Assetnote). Cheque Crunchbase/Wikipedia/press releases → jogue os novos domínios no funil.

**Subdomínios em massa** (o principal): passivo + ativo + permutação, com **muitas** fontes:
```bash
subfinder -d apple.com -all -silent | anew subs.txt
# + amass enum, github-subdomains, permutação (alterx/gotator) + brute com puredns
httpx -l subs.txt -silent -sc -title -td -location | tee live.txt
```

**Cloud.** Buckets e blobs escapam do DNS: S3, Azure Blob, GCS, DigitalOcean Spaces — enumere por nomes/prefixos da marca.

**Indexe tudo.** Curry montou um dashboard com **status + headers + body + screenshot** de cada host → priorizou pelos "interessantes". Faça o mesmo: sua vantagem é ver 5.000 hosts e saber **quais 20 olhar**.

## 3 — Profundidade: entrar em cada asset

- **JavaScript é o mapa da API.** Extraia e leia o JS → rotas, endpoints, GraphQL, params (Apple: acharam `/services/public/account` lendo script). Use [jsluice](https://github.com/BishopFox/jsluice)/LinkFinder; a Assetnote chama isso de "depth" (JS → GraphQL queries, permissões de API).
- **App mobile → APIs que o web não expõe.** *"As APIs que o app mobile fala tocam partes da aplicação que você não veria no fluxo normal."* Extraia endpoints do APK/IPA ([apkleaks](https://github.com/dwisiswant0/apkleaks)), e intercepte com Burp (Curry driblou parte do SSL pinning ativando o proxy só depois de navegar). Reverse de **thick clients** revela a API por baixo.
- **Erros que vazam.** Param malformado → exceção. Na Apple, um `marketCode` inválido devolveu uma **REST exception com hostname interno + token de auth**. Sempre quebre o input e leia o stack trace.

## 4 — 🎯 As "funções pouco usadas" (o que você procura)

O coração da sua pergunta. Onde mora a superfície que ninguém garimpou:

- **Legado via arquivo.** [Wayback/archive](https://github.com/lc/gau) (`gau`, `waymore`) → endpoints velhos, páginas de debug, backups, arquivos apagados, **versões antigas de API** (`/v1` ainda vivo enquanto `/v3` é o atual — e o `/v1` não tem as checagens novas).
- **Actuator / debug / docs expostos.** Spring Boot **`/actuator`** (Apple achou `/viewer/actuator`!), `swagger.json`/`openapi.json`/`api-docs`, **WADL** (`application.wadl` — na Apple, revelou a **API inteira** de um app Java), `.git/`, `.env`, GraphQL **introspection**.
- **Staging / dev / UAT / preprod.** Dorks + subdomínios: `staging.` `dev.` `uat.` `preprod.` `qa.` `internal.` — ambientes com menos hardening e dados reais.
- **Painéis internos / admin** em IPs não-listados (via o scan de ASN da seção 2).
- **Endpoints só alcançáveis por client antigo/mobile** — não linkados em lugar nenhum no web.
- **API não usada pelo front.** A superfície de API que a UI não chama é o [tema da própria PortSwigger (API testing)](https://portswigger.net/web-security/api-testing) — endpoints escondidos + server-side parameter pollution.

## 5 — Context: wordlist inteligente, não brute burro

A Assetnote insiste: *"se vai rodar wordlist, seja esperto sobre qual."* Em vez de `common.txt` genérico, use listas **cientes da tecnologia** do alvo (achou Spring? procura `actuator`, `env`, `heapdump`; achou Java? `wadl`, `.jsp`; achou Node? rotas de API). Wordlists da própria Assetnote e do SecLists por tech. Isso multiplica o hit rate.

## 6 — GitHub / OSINT (onde o interno vaza)

- **Repos da org e de funcionários:** URLs internas (Apple achou uma **URL do maven interno** num repo público), configs, endpoints.
- **Segredos:** [githound](https://github.com/tillson/git-hound), [trufflehog](https://github.com/trufflesecurity/trufflehog), GitHub dorks (`"empresa.com" api_key`, `org:empresa password`).
- **Funcionários / e-mails / tech stack:** LinkedIn, job posts (revelam a stack interna → o que procurar).

## 7 — O fluxo (do amplo ao fraco)

```
ASN + aquisições + subs em massa + cloud    → mapeia a CIDADE
        ↓ (httpx + screenshot + index)
tria os "interessantes" (admin/staging/erro/legado)
        ↓ (JS + mobile + wordlist esperta + wayback)
acha a FUNÇÃO POUCO USADA (v1 antigo, actuator, wadl, painel interno)
        ↓
Burp na veia → authz/lógica/IDOR na superfície que ninguém olhou
```

> A máquina te dá a **amplitude** (a cidade inteira); você aplica a **focus** (Assetnote) — o instinto de ver, entre 5.000 hosts, os 20 que cheiram a bug. É o mesmo princípio do [recon minimalista](ROADMAP-MINIMALISTA.md), só que em escala de big tech.

## Fontes

- [Sam Curry et al. — *We Hacked Apple for 3 Months: Here's What We Found*](https://samcurry.net/hacking-apple)
- [Assetnote — *The Art of Recon: Strategies for Modern Asset Discovery*](https://www.assetnote.io/resources/blog/the-art-of-recon-strategies-for-modern-asset-discovery)
- [awesome-google-vrp-writeups](https://github.com/xdavidhu/awesome-google-vrp-writeups) · [BigBountyRecon](https://github.com/Viralmaniar/BigBountyRecon) (58 técnicas de dork/OSINT)

---

⬅️ [Programas com Hall da Fama](programas-hall-da-fama.md) · [ROADMAP MINIMALISTA](ROADMAP-MINIMALISTA.md) · [🏠 Início](README.md)
