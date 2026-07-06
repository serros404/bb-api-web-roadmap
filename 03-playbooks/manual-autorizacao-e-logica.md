# 13 — Caçador Manual: Autorização e Lógica de Negócio (onde o scanner é cego)

[🏠 Início](../README.md) › [03 · Playbooks](README.md) › Caçador Manual: Autorização e Lógica

**Sumário:**
- [1 — Por que esta é a sua trilha premium](#1--por-que-esta-é-a-sua-trilha-premium)
- [2 — Mindset: modelar o sistema, não fuzzar](#2--mindset-modelar-o-sistema-não-fuzzar)
- [3 — Modelagem antes do teste: o mapa de autorização](#3--modelagem-antes-do-teste-o-mapa-de-autorização)
- [4 — BOLA / IDOR a fundo (além de "trocar o ID")](#4--bola--idor-a-fundo-além-de-trocar-o-id)
- [5 — BAC / BFLA a fundo](#5--bac--bfla-a-fundo)
- [6 — Lógica de negócio: taxonomia de vetores](#6--lógica-de-negócio-taxonomia-de-vetores)
- [7 — Multi-tenancy e RBAC/ABAC moderno](#7--multi-tenancy-e-rbacabac-moderno)
- [8 — O workflow manual (automação a serviço)](#8--o-workflow-manual-automação-a-serviço)
- [9 — Encadeamento: de authz/lógica ao impacto máximo](#9--encadeamento-de-authzlógica-ao-impacto-máximo)
- [10 — Treino ↔ caça (CTF/HTB ↔ bounty)](#10--treino--caça-ctfhtb--bounty)
- [🧠 Síntese](#-síntese)

---

> **Esta é a trilha que joga a seu favor.** Se o seu prazer e talento estão nas **falhas de lógica, BAC/BFLA e BOLA/IDOR** — as classes que **scanner nenhum acha** —, este documento é o seu quartel-general. Ele fica **acima** do [playbook de classes (Parte C)](classes-de-bug.md#parte-c--autorização-autenticação-e-lógica-onde-está-o-dinheiro): lá você aprende *cada classe*; aqui você aprende a **operar como caçador de autorização/lógica** — modelar o sistema, montar a matriz de teste, e encadear até o impacto máximo.
>
> Tudo para uso **autorizado**, com **PoC de impacto mínimo** (releia o [aviso ético](../README.md#-aviso-ético-leia-antes-de-tudo--e-releia-em-cada-fase)): provou um IDOR? Mostre **um** registro que não é seu e pare.

---

## 1 — Por que esta é a sua trilha premium

Três fatos que fazem de autorização + lógica o lugar certo para um caçador manual:

- **Scanner é cego aqui.** Não existe "payload" de IDOR ou de logic flaw. O bug é a **ausência de uma checagem** ou uma **regra de negócio quebrada** — algo que só se enxerga entendendo *o que a aplicação deveria impedir*. Nenhuma ferramenta entende a intenção do sistema; **você** entende. É a própria tese do roadmap ([profundidade por humano](../README.md#-a-metodologia-2026-amplitude-por-máquina-profundidade-por-humano)) no seu ponto mais puro.
- **É onde está o dinheiro.** BOLA é, ano após ano, a vulnerabilidade nº 1 de API ([OWASP API Top 10](../02-trilha/02-api-f3.md)). BAC/BFLA e logic flaws fecham **críticos** e **ATOs**. Enquanto 500 hunters jogam payload de XSS no mesmo campo, o endpoint de autorização quebrado está intocado — porque exige contexto, e contexto não se automatiza.
- **Conecta com a lente Orange Tsai.** Muitos bypasses de acesso não são "falta de checagem" — são **confusão de rota/parser**: o middleware acha que `/admin` está protegido, mas `/Admin`, `/admin%2f` ou um header interno forjado ([`x-middleware-subrequest`](orange-tsai-decodificado-js.md#24-path-confusion--normalização--roteamento--expressfastifynext)) chegam ao handler sem passar pela auth. Autorização quebrada + [diferencial de parser](tecnicas-avancadas-e-cadeias.md#h--diferenciais-de-parser-e-confusão-de-componentes-a-veia-orange-tsai) = a cadeia que ninguém vê.

> ⚠️ **O contrapeso honesto.** Manual-first **não** é anti-automação. Você ainda precisa de recon — só que aqui ele tem um papel estreito e servil: **surfacear os objetos, papéis e endpoints** que você vai testar à mão. Sem recon nenhum, você caça uma superfície minúscula. Com recon **a serviço da sua matriz de autorização**, você tem todos os alvos e gasta 100% da sua energia humana na parte que paga. Ver [seção 8](#8--o-workflow-manual-automação-a-serviço).

---

## 2 — Mindset: modelar o sistema, não fuzzar

O caçador de payload pergunta *"o que eu injeto aqui?"*. O caçador de autorização/lógica pergunta outra coisa — e essa pergunta é a habilidade inteira:

> **"O que esta funcionalidade deveria impedir — e quem deveria poder fazê-la, com quais objetos? Onde essa regra pode não estar sendo verificada?"**

Quatro perguntas que você faz para **todo objeto** e **toda ação**:

1. **Quem é o dono?** (a qual usuário/tenant este objeto pertence?)
2. **Como ele é referenciado?** (ID sequencial? UUID? slug? no path, body, header, cookie?)
3. **A posse é checada nesta operação específica?** (ler pode checar; escrever/deletar pode esquecer)
4. **A checagem cobre todos os caminhos?** (todos os verbos, versões de API, formatos, rotas alternativas que chegam ao mesmo objeto?)

O bug quase sempre mora na **assimetria**: a checagem existe num lugar e falta em outro. Ler é protegido, escrever não. `/v2` valida, `/v1` não. A UI esconde o botão, a API não checa. **Sua caça é procurar a assimetria.**

---

## 3 — Modelagem antes do teste: o mapa de autorização

Não comece testando. Comece **desenhando o sistema** — 30 minutos de modelagem economizam horas de tentativa aleatória e revelam bugs que o teste cego não acha.

**Monte três artefatos** (no seu `notes`):

- **Matriz de papéis × funções.** Linhas = papéis (anônimo, user, user premium, moderador, admin, suporte, dono de tenant). Colunas = funções sensíveis (ver X, editar X, deletar X, listar todos, mudar papel, exportar, faturar). Cada célula: *deveria poder?* vs *consegue?*. Cada divergência é um candidato a BAC/BFLA.
- **Grafo de objetos e posse.** Quais objetos existem (user, order, invoice, file, comment, team, project, webhook…), como se referenciam (`order → items → product`), e a quem pertencem. Cada aresta é um candidato a IDOR — inclusive os **aninhados** (`/orders/{a}/items/{b}` — `b` é checado contra o dono de `a`?).
- **Fronteiras de confiança e tenant.** Onde a aplicação **assume** que algo já foi validado: borda/middleware, front vs API, tenant vs tenant, valores que o cliente calcula (preço, total, papel). Toda fronteira é um lugar onde a checagem pode ter ficado do lado errado.

> A modelagem é exatamente o "ler o código e modelar o comportamento real" do [Orange Tsai](orange-tsai-decodificado-js.md#14-lê-o-código-fonte-e-modela-o-comportamento-real-não-o-documentado), aplicado à autorização. Se houver JS de front, Swagger ou GraphQL introspection, você **reconstrói o modelo de autorização** a partir deles (a IA ajuda a resumir — nunca com dado sensível de alvo).

---

## 4 — BOLA / IDOR a fundo (além de "trocar o ID")

O básico ([C1](classes-de-bug.md#c1-idor--bola-broken-object-level-authorization)): duas contas, troca o ID, viu dado alheio → BOLA. O que separa o especialista:

- **IDOR de escrita > leitura.** `PUT`/`PATCH`/`DELETE` no objeto de outro é impacto muito maior que ler. Sempre teste as operações de **mutação**, não só `GET`.
- **Objetos aninhados / relacionais.** `/orders/{meu}/items/{alheio}` — o backend checa o dono de `orders/{meu}` mas confia cegamente em `items/{alheio}`? Relações são onde a checagem "esquece".
- **UUID não é controle.** "É UUID, não dá pra adivinhar" é falso: UUIDs **vazam** em busca, notificações, exports, `Referer`, mensagens de erro, versões antigas da API, respostas de *outros* endpoints. **Colete o ID num lugar, use no IDOR de outro** (second-order/blind).
- **Blind IDOR.** Efeito sem retorno direto: você altera a config de B, a resposta não mostra dado, mas a mudança **aconteceu**. Confirme pelo efeito colateral (login como B em lab, e-mail disparado, estado mudado).
- **Endpoints de massa/agregação.** `list`/`search`/`export`/`report` que não filtram por tenant → IDOR **em lote**. Altíssimo impacto (e altíssimo cuidado de escopo: prove com poucos registros).
- **GraphQL `node(id:)` e global IDs.** IDs globais costumam ser `base64(tipo:id)`. Decodifique, troque o `id` (ou o `tipo`), re-encode. Mesmo BOLA, roupa nova.
- **Canais assíncronos.** Webhooks, e-mails, PDFs, notificações referenciam objetos por ID — e a checagem de autorização costuma faltar nesses caminhos "de trás".
- **IDs "encodados".** Hashids, base64, IDs cifrados mas reversíveis/previsíveis, sequenciais sob encoding — decodifique antes de concluir que é "seguro".
- **Chegar ao objeto por outra porta.** O mesmo objeto servido por um endpoint que **tem** a checagem e outro que **não tem** (versão legada, rota alternativa, método diferente) — ver [BFLA](#5--bac--bfla-a-fundo) e [confusão de rota](tecnicas-avancadas-e-cadeias.md#h--diferenciais-de-parser-e-confusão-de-componentes-a-veia-orange-tsai).

---

## 5 — BAC / BFLA a fundo

Acesso a **funções** que seu papel não deveria alcançar ([C2](classes-de-bug.md#c2-bac--bfla-broken-functionaccess-control)). Os vetores que rendem:

- **O gap UI-vs-API (o mais comum).** A UI esconde o botão de admin, mas o **endpoint** não tem checagem server-side. Descubra as rotas no **JS do front**, no Swagger, na versão anterior da API — e chame-as como user comum. "Não aparece na tela" ≠ "protegido".
- **Verb tampering.** `GET` liberado, mas `PUT`/`PATCH`/`DELETE`/`OPTIONS` não checados no mesmo recurso.
- **Formato / content-type.** JSON bloqueado, `form-urlencoded` passa; ou a **mesma ação via GraphQL** enquanto o REST é protegido (ou vice-versa).
- **Gaps de versão.** `/api/v2/admin/...` valida; `/api/v1/admin/...` (ainda vivo) não. [Improper inventory (API9)](../02-trilha/02-api-f3.md).
- **Rota / case / encoding.** `/admin` negado, mas `/Admin`, `/admin/`, `/admin%2f`, `/./admin` chegam — a **veia Orange Tsai** de [confusão de rota](orange-tsai-decodificado-js.md#24-path-confusion--normalização--roteamento--expressfastifynext) aplicada a controle de acesso.
- **Mass assignment como BFLA.** `role=admin`, `is_admin=true`, `account_id={outro}`, `verified=true` no body de um update de perfil ([BOPLA/API3](../02-trilha/02-api-f3.md)). A função de "virar admin" não existe na UI, mas existe no modelo.
- **Bypass de step-up/re-auth.** Ação sensível pede senha de novo **na UI**, mas o endpoint da API não exige. Pule a UI.
- **"Autorização" por header do cliente.** App confia em `X-Role`, `Referer`, `Origin`, `X-Forwarded-For` para decidir acesso — todos forjáveis.

---

## 6 — Lógica de negócio: taxonomia de vetores

Aqui não há classe fixa — há **raciocínio**. Mas raciocínio tem método: percorra estas **categorias**, e para cada funcionalidade pergunte *"e se eu quebrar esta regra?"*. Esta taxonomia transforma "pensar fora da caixa" em checklist.

| Categoria | A pergunta / o abuso | Exemplo |
|---|---|---|
| **Quantidade / valor / sinal** | e se for negativo, zero, decimal, gigante? | `-1` no carrinho credita saldo; `0.001` fura arredondamento |
| **Sequência / estado** | e se eu pular ou reordenar etapas? | chegar em "pago" sem pagar; usar recurso antes de ativar |
| **Tempo / corrida (TOCTOU)** | e se eu disparar simultâneo? | resgatar cupom único N vezes ([C4](classes-de-bug.md#c4-race-conditions)) |
| **Confiança no cliente** | o servidor confia num valor que o cliente calculou? | preço/total/desconto/papel vindos do front |
| **Quota / limite / reuso** | e se eu exceder ou reusar o "uma vez"? | trial infinito, cupom empilhado, indicação abusada |
| **Moeda / arredondamento** | trocar moeda, casas decimais, taxa? | pagar em moeda fraca, arredondar a favor |
| **Replay / idempotência** | reenviar uma request assinada/única? | reusar token de uso único, replay de pagamento |
| **Escopo de fluxo** | executar a etapa de um fluxo que é de outro? | completar checkout/onboarding de outro tenant |
| **Identidade / verificação** | confiar em e-mail não verificado? merge de conta? | pre-account takeover via SSO |

> O segredo é **entender o fluxo de negócio primeiro** (mapeie no Burp), listar cada **validação implícita**, e quebrá-las uma a uma. É a classe mais humana que existe — automação aqui é ~zero (a IA só ajuda a *brainstormar* vetores a partir da descrição do fluxo, sem dado de alvo).

---

## 7 — Multi-tenancy e RBAC/ABAC moderno

O SaaS moderno é onde autorização mais quebra — e onde os bounties de tenant valem muito:

- **Isolamento de tenant (o bug nº 1 de SaaS).** Todo objeto deveria ser escopado ao **seu** tenant. Teste leitura **e** escrita cross-tenant em **cada tipo de objeto** (user, project, file, invoice, webhook, member…). Uma única falha de isolamento = acesso aos dados de outra empresa.
- **Escalada de papel via atribuição.** Você consegue se **auto-promover**? Convidar a si mesmo como admin? Mudar o próprio `role` num update de perfil (mass assignment)? Aceitar um convite e herdar papel a mais?
- **Herança de permissão.** Lógica de grupo/time/projeto que vaza: "compartilhado com", toggle público/privado, permissão herdada de um projeto pai que não deveria valer no filho.
- **Impersonação / "assumir usuário".** Ferramentas de suporte que "logam como" um cliente costumam ser mal-protegidas — e são ATO instantâneo se acessíveis.
- **ABAC / permissões granulares.** Regras baseadas em atributo (departamento, região, plano) têm casos de borda: um atributo que você controla no perfil muda o que você pode ver.

---

## 8 — O workflow manual (automação a serviço)

Como um caçador manual usa a máquina **só o necessário** para ter todos os alvos, e gasta a energia humana onde paga:

```
1. RECON (servil, mínimo)  → enumere a superfície: endpoints (JS/Swagger/kiterunner),
                              params escondidos (arjun/x8), e MONTE A FROTA DE CONTAS
                              (uma por papel: user A, user B, admin, tenant X, tenant Y).
2. MODELE (seção 3)        → matriz papéis×funções + grafo de objetos + fronteiras.
3. MATRIZ DE TESTE         → para cada célula: "papel R consegue ação A no objeto O do tenant T?"
4. REPEATER (sua casa)     → mude UMA variável por vez: ID, verbo, header de papel, param de tenant.
5. AUTORIZE (Burp)         → compara duas sessões automaticamente (A vs B, user vs admin).
6. CONFIRME + DOCUMENTE    → posse do objeto, impacto, e o que falta para encadear.
```

- **Ferramentas certas para o perfil:** [Autorize](../05-referencia/arsenal-labs-checklists-etica.md#burp--extensões-essenciais-bapp-store) (o motor da caça de autz — testa IDOR/BOLA/BAC com duas sessões), Repeater, [arjun](https://github.com/s0md3v/Arjun)/[x8](https://github.com/Sh1Yo/x8) para params escondidos, [InQL](https://github.com/doyensec/inql) para GraphQL. O resto do recon é só para **surfacear** ([motor de recon](../01-recon/recon-engine-ia-e-automacao.md)).
- **A regra:** a máquina te dá *onde olhar* (endpoints, params, objetos); **você** faz as perguntas de posse e papel que nenhum scanner faz. Se você ainda não abriu o Repeater com duas contas, você ainda não começou a caçar.

---

## 9 — Encadeamento: de authz/lógica ao impacto máximo

Um IDOR de leitura sozinho é "medium". Encadeado, vira "critical". O caçador de autorização/lógica pensa em [cadeias](tecnicas-avancadas-e-cadeias.md#a--cadeias-de-exploração-chaining-pensar-em-sistemas-não-em-bugs):

| Cadeia | Elos | Resultado |
|---|---|---|
| **IDOR read → IDOR write → ATO** | ler ID/e-mail de B → trocar e-mail de B sem confirmação → reset | tomada de conta |
| **BFLA → mass assignment → privesc** | alcançar endpoint de admin → se conceder `role:admin` | admin do sistema |
| **Cross-tenant write → tenant takeover** | escrever objeto de outro tenant → assumir recurso/fluxo | comprometimento de empresa |
| **Logic (pular pagamento) → IDOR (aplicar em outros)** | burlar cobrança → escalar para outras contas | fraude em escala |
| **Route confusion → BAC** | `/admin%2f` passa o middleware → função de admin | bypass de acesso (veia Orange) |

> Inventarie **todo** achado fraco (IDOR de leitura, info leak, param escondido, self-privesc), anote *o que dá / o que falta*, e procure o elo. Uma cadeia bem-narrada que termina em ATO transforma três "lows" num "critical" — e vira um writeup "How I…" ([C8](classes-de-bug.md#c8-account-takeover-ato--encadeamento)).

---

## 10 — Treino ↔ caça (CTF/HTB ↔ bounty)

O loop [CTF/labs → bounty](../02-trilha/03-ctf-cve-carreira-f4-f7.md#-f4--ctf--labs-a-serviço-do-bug-bounty-velocidade-cadeias-e-escalonamento) aplicado ao seu nicho:

- **Treino (velocidade + reflexo):** os labs de **Access control**, **Authentication** e **Business logic vulnerabilities** da [PortSwigger Academy](https://portswigger.net/web-security/access-control) — faça **todos** (são o melhor treino de autz/lógica que existe). No [HTB](https://www.hackthebox.com), máquinas multi-usuário e o mindset "assume breach → o que este papel alcança?".
- **Caça (onde paga, foco JS/SaaS):** apps **multi-tenant** (o isolamento é ouro), **marketplaces** (comprador/vendedor/admin — papéis que se cruzam), **fintech** (movimento de dinheiro = mina de logic flaw), e **qualquer API REST/GraphQL com objetos por ID** ([F3](../02-trilha/02-api-f3.md)). Onde há papéis, tenants e dinheiro, há autz/lógica quebrada.
- **Reconheça no real o que treinou no lab:** o mesmo "pular a etapa" do lab de business logic é o checkout quebrado do alvo; o mesmo Autorize do lab de access control é a sua primeira hora em qualquer programa.

---

## 🧠 Síntese

> **Você não caça payloads — caça a assimetria entre o que o sistema deveria impedir e o que ele de fato verifica.** Scanner é cego a isso porque não entende a intenção; você não é. Modele o sistema (papéis, objetos, tenants, fronteiras), monte a matriz, teste com duas contas no Repeater, e **encadeie** até o impacto máximo. É a classe mais humana, a que mais paga, e a que joga a favor do seu perfil manual. A automação te entrega a frota de alvos; **você** faz as perguntas de posse, papel e lógica que valem o topo da tabela — e as narra num writeup "How I…" que constrói sua reputação.

Base por classe: [playbook de classes — Parte C](classes-de-bug.md#parte-c--autorização-autenticação-e-lógica-onde-está-o-dinheiro). Lente de research: [Orange Tsai (foco JS)](orange-tsai-decodificado-js.md). Onde isso mais aparece: [API/F3](../02-trilha/02-api-f3.md) e o [loop CTF↔bounty da F4](../02-trilha/03-ctf-cve-carreira-f4-f7.md#-f4--ctf--labs-a-serviço-do-bug-bounty-velocidade-cadeias-e-escalonamento). Boa caçada. 🎯

---

⬅️ [Anterior: Orange Tsai Decodificado (foco JS)](orange-tsai-decodificado-js.md) · [⬆️ Topo](#) · 🏁 Fim do núcleo web/API — [voltar ao Início](../README.md)
