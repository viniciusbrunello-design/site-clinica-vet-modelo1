# 03 — Plano técnico do Site 1 (Clínica VetSaúde, arquétipo clínico)

**Status:** `APROVADO em 2026-09-29`

> Este plano define **o quê** construir e **em que ordem**. A divisão em camadas/agentes e a posse de arquivos ficam para a etapa 4 (`docs/04-camadas.md`). Fontes: `docs/00-PRD.md` (prioridade máxima), `docs/02-briefing.md`, `docs/01-objetivo.md`, `CLAUDE.md`.

---

## 1. Resumo

Site single-page em **Astro**, publicado na **Vercel via GitHub**, cujo objetivo único é transformar visitante em **lead no WhatsApp** (agendamento ou dúvida). Todas as páginas são estáticas; a **única** exceção de backend é a função serverless `src/pages/api/carteirinha.ts`, que lê a foto da carteirinha com IA de visão real (Anthropic) e devolve uma triagem por vacina. Os dados do cliente ficam isolados em `src/data/cliente.json` para o site virar template clonável. O visual é definido primeiro em `docs/design-system.md`, aprovado pelo Vinicius antes de qualquer seção. O site é mobile-first, acessível, com SEO local caprichado (é o diferencial de venda).

---

## 2. Visão geral

- **Um objetivo de negócio:** lead no WhatsApp. Dois "heróis" servem a isso e impressionam o comprador (dono da clínica): o leitor de carteirinha e o agendamento em 30s.
- **Dois públicos:** o tutor ansioso no celular (decide em ~5s) e o dono da clínica (quer ver "isso me traz cliente").
- **Arquétipo clínico:** medicina séria (consultas, vacinas, cirurgia, emergência 24h). Sem banho e tosa / antes e depois — isso é o Site 2. Fronteira mantida limpa de propósito.
- **Template desde já:** dados do cliente só em `cliente.json`; marca em variáveis CSS; dimensões variáveis (PRD seção 12) existem como flags.

---

## 3. Arquitetura técnica e estrutura de pastas

**Stack:** Astro (páginas estáticas pré-renderizadas) + adapter `@astrojs/vercel` para a única rota dinâmica. Deploy Vercel via GitHub. JS no navegador só nas ferramentas interativas, como scripts isolados por componente. Marca em variáveis CSS no `:root`. Fontes via Google Fonts ou locais; nenhuma outra dependência externa em runtime no navegador.

**Armazenamento:** nenhum banco de dados. Única exceção: Upstash Redis **exclusivamente** para contadores de limite de uso da carteirinha (chaves com hash e expiração automática). Nada de dado pessoal, imagem ou resultado é gravado. Proibido `localStorage`/`sessionStorage` — estado só em memória.

**Testes:** framework **Vitest**, executado por `npm run test` (é o comando que o supervisor roda). Cobre a lógica da função da carteirinha sem chamar a IA real (ver seção 7).

### Árvore de diretórios proposta

```
/
├── astro.config.mjs          # adapter @astrojs/vercel; site (URL) lido de config
├── package.json              # scripts (dev, build, test); dependências abaixo
├── vitest.config.ts          # configuração do Vitest
├── tsconfig.json
├── .gitignore                # inclui .env
├── .env.example              # nomes das variáveis SEM valores (referência)
├── public/
│   ├── favicon.svg           # derivado do logo (pata) — usa cor da marca
│   ├── favicon.png           # fallback
│   └── og.svg                # fonte da imagem Open Graph (tokens da marca) → convertida em og.png no build
├── src/
│   ├── data/
│   │   ├── cliente.json          # TODOS os dados do cliente + flags (seção 4)
│   │   ├── regras-vacinas.json   # intervalos de reforço por vacina/espécie — VALIDAR-VET; usado só pelo código
│   │   └── alimentos.json        # lista fixa curada "pode comer isso?" — VALIDAR-VET
│   ├── layouts/
│   │   └── Base.astro            # <head> (SEO, OG, JSON-LD, canonical, noindex demo)
│   ├── styles/
│   │   └── global.css            # variáveis CSS da marca no :root, reset, utilitários (PROPRIEDADE do design system, passo 2)
│   ├── components/
│   │   ├── Nav.astro                 # nav fixa + faixa emergência + menu hambúrguer (JS)
│   │   ├── Hero.astro                # headline + emergência + CTAs (contém o widget da carteirinha)
│   │   ├── CarteirinhaWidget.astro   # Herói 1: upload/estados/resultado (JS)
│   │   ├── Agendamento.astro         # Herói 2: mini-form → wa.me (JS)
│   │   ├── Servicos.astro            # 6 cards
│   │   ├── PlanoVetSaude.astro       # dimensão variável (flag temPlano)
│   │   ├── Equipe.astro              # grid flexível 1..4+ (CRMV)
│   │   ├── Localizacao.astro         # endereço, "Como chegar", horários, emergência
│   │   ├── Depoimentos.astro         # 3 depoimentos demo
│   │   ├── Footer.astro              # contato, nav, nota LGPD
│   │   ├── BotaoWhatsappFlutuante.astro
│   │   └── pacote-captacao/          # bloco destacável (módulo do template)
│   │       ├── CalculadoraIdade.astro   # JS
│   │       ├── ChecklistEmergencia.astro # JS (sempre "Ligar agora" tel:)
│   │       └── PodeComer.astro           # JS, usa alimentos.json (nunca IA)
│   ├── lib/                          # utilidades compartilhadas (montar links wa.me, escapar texto, esquema zod da IA, etc.)
│   └── pages/
│       ├── index.astro               # single page: monta todas as seções na ordem do PRD
│       ├── robots.txt.ts             # rota pré-renderizada: gera robots.txt a partir do cliente.json (ver SEO)
│       ├── sitemap.xml.ts            # rota pré-renderizada: só gerada quando demoSite = false (ver SEO)
│       └── api/
│           └── carteirinha.ts        # ÚNICA função serverless (prerender = false)
└── docs/
    ├── design-system.md              # produzido/aprovado ANTES das seções
    └── deploy.md                     # passo a passo de variáveis, Upstash, Anthropic, Vercel
```

**Dependências principais (package.json):** `astro`, `@astrojs/vercel`, `@upstash/ratelimit`, `@upstash/redis`, SDK Anthropic; **dev/build:** `vitest` (testes), `zod` (validação do esquema do JSON da IA), `sharp` (converte `og.svg` → `og.png` 1200×630 no build; já é dependência comum do Astro).

**Frase simples:** o site inteiro é um monte de páginas prontas (rápidas e baratas); só a leitura da carteirinha precisa de um "mini-servidor", e nada mais.

---

## 4. Esquema do `src/data/cliente.json`

Todos os dados do cliente ficam aqui — **nada hardcoded nos componentes**. Regra: **uma informação = um único campo** (sem flags duplicadas). Tipos e exemplos abaixo (valores fictícios do brief, PRD seção 3; WhatsApp da demo = número do Vinicius, briefing).

| Campo | Tipo | Exemplo | Origem |
|-------|------|---------|--------|
| `nome` | string | `"Clínica VetSaúde"` | PRD 3 |
| `site` | string (URL) | `"https://site-clinica-vet-modelo1.vercel.app"` | briefing (URL provisória da Vercel; o Vinicius confirma/troca no deploy — é só mudar aqui) |
| `demoMode` | boolean | `false` | PRD 7 — ver comportamento abaixo |
| `demoSite` | boolean | `true` | briefing — liga `noindex` + aviso de site demonstrativo |
| `whatsapp` | string (só dígitos, formato wa.me) | `"5519982257235"` | briefing (número do Vinicius na demo) |
| `telefone` | string | `"(11) 99999-9999"` | PRD 3 (fictício, exibido) |
| `email` | string | `"contato@vetsaude.com.br"` | PRD 3 |
| `endereco` | objeto `{ logradouro, bairro, cidade, uf }` | `{ "logradouro": "Rua Exemplo, 123", "bairro": "Vila Mariana", "cidade": "São Paulo", "uf": "SP" }` | PRD 3 |
| `horarios` | array de `{ dias, horario }` | `[{ "dias": "Seg–Sex", "horario": "8h–20h" }, { "dias": "Sáb", "horario": "8h–16h" }, { "dias": "Dom e feriados", "horario": "Emergências 24h" }]` | PRD 3 |
| `provaSocial` | objeto `{ estrelas, anos, tutores }` | `{ "estrelas": "4,9★ no Google", "anos": "+10 anos de bairro", "tutores": "+8.000 tutores atendidos" }` | PRD 3 |
| `servicos` | array de `{ icone, titulo, frase }` | `[{ "icone": "estetoscopio", "titulo": "Consultas e exames", "frase": "..." }]` (6 itens) | PRD 9 |
| `servicosAgendamento` | array de string | `["Consulta", "Vacinação", "Exame", "Cirurgia", "Odontologia", "Emergência"]` | PRD 8 (alimenta o select) |
| `equipe` | array de `{ nome, especialidade, crmv }` | `[{ "nome": "Dra. Ana Ribeiro", "especialidade": "Clínica geral / Responsável técnica", "crmv": "CRMV-SP 00000" }, ...]` (4 itens) | PRD 3 |
| `plano` | objeto `{ nome, inclui[], descricao }` | `{ "nome": "Plano VetSaúde", "inclui": ["consultas de rotina", "vacinas do calendário", "descontos em exames"] }` | PRD 3/9 |
| `depoimentos` | array de `{ autor, texto, estrelas, tema }` | 3 itens (um emergência noturna, um lembrete de vacina) | PRD 9 |
| `mensagensWhatsapp` | objeto | ver abaixo | briefing (aprovadas) |
| `carteirinha` | objeto | ver abaixo | PRD 7 |
| `lgpd` | objeto `{ controladora, contato }` | `{ "controladora": "Clínica VetSaúde", "contato": "contato@vetsaude.com.br" }` | briefing |
| `flags` | objeto (dimensões variáveis) | ver abaixo | PRD 12 |

> **Espécies:** existe **um único** campo, `flags.especies` (removido o `especies` de topo, que duplicava a mesma informação).

**Comportamento do `demoMode`** (uma flag só, nome do PRD; **não** existe `carteirinha.iaReal`):
- `demoMode: false` → o front **chama** `/api/carteirinha`. Se a API responder `"nao_configurado"`, o front cai no **mock** com o selo "Modo demonstração".
- `demoMode: true` → o front **nunca** chama a API e usa direto o mock.
- **Valor padrão neste site:** `demoMode: false` — a falha fechada da função (seção 7) já garante o mock enquanto a chave e o Upstash não estiverem configurados.
- **O mock é usado SOMENTE** nas respostas `"demo"` (demoMode true) e `"nao_configurado"`. Nas respostas `"limite"`, `"erro"` e `"ilegivel"`, **NUNCA** mostrar o mock — seria um resultado falso apresentado como real. Cada um desses estados mostra a própria mensagem + botão de WhatsApp.

**`mensagensWhatsapp`** (as `{}` são preenchidas em runtime; textos exatos do briefing):
```json
{
  "agendamento": "Olá! Quero agendar *{servico}* para o *{pet}*. Dia preferido: {data} ({periodo}). Vim pelo site.",
  "plano": "Olá! Vim pelo site e quero saber mais sobre o Plano VetSaúde (mensalidade, o que inclui e como assinar).",
  "carteirinhaPendencias": "Olá! Fiz a triagem da carteirinha no site e apareceram vacinas para verificar: {vacinas}. Gostaria de confirmar a situação do meu pet com a equipe.",
  "carteirinhaSemPendencias": "Olá! Fiz a triagem da carteirinha no site e gostaria que a equipe confirmasse se as vacinas do meu pet estão em dia.",
  "emergencia": "Olá! É uma *emergência* com meu pet e preciso de atendimento agora.",
  "generico": "Olá! Vim pelo site da Clínica VetSaúde e gostaria de tirar uma dúvida.",
  "ferramentas": "Olá! Usei as ferramentas do site e gostaria de agendar um check-up / tirar uma dúvida sobre meu pet."
}
```

**`carteirinha`** (PRD 7) — sem `ativa` e sem `iaReal` (ver notas abaixo):
```json
{
  "limites": {
    "porIpHora": 5,
    "porIpDia": 10,
    "globalDia": 200
  }
}
```
> Removidos `carteirinha.ativa` (a **única** chave que liga/desliga a carteirinha é `flags.temCarteirinha`) e `carteirinha.iaReal` (duplicava o `demoMode`).

**`flags`** — dimensões variáveis (PRD 12), mesmo com valor fixo aqui, existem para o template dobrar depois. Cada uma será marcada com comentário no código onde afeta o layout:
```json
{
  "heroiPrincipal": "carteirinha",   // carteirinha | slider (Site 2)
  "temCarteirinha": true,            // única chave que liga/desliga a carteirinha
  "emergencia24h": true,             // false remove faixa e etiqueta
  "especies": ["cão", "gato"],       // cão+gato | só cão | só gato (afeta textos) — fonte única de espécies
  "temPlano": true,                  // false oculta bloco Plano
  "pacote": "captacao",              // captacao | marketing (Site 2)
  "temBanhoTosa": false              // aqui não é destaque
}
```
(`nº de veterinários / layout da equipe` é derivado do tamanho de `equipe` — grid flexível 1..4+.)

---

## 5. Mapa de seções → componentes

Ordem exata do PRD seção 5. "JS" = tem script interativo no navegador; "estático" = só HTML/CSS gerado no build.

| # | Seção (PRD 5/9) | Componente | Interatividade |
|---|-----------------|------------|----------------|
| 1 | Nav fixa (logo, âncoras, WhatsApp, faixa emergência) | `Nav.astro` | **JS** (menu hambúrguer mobile) |
| 2 | Herói (headline + emergência 24h + CTAs) | `Hero.astro` | estático (CTAs rolam por âncora) |
| 2b | Herói 1 — leitor de carteirinha (à direita da headline) | `CarteirinhaWidget.astro` | **JS** (upload, redução de imagem, estados, resultado) |
| 3 | Herói 2 — agendamento em 30s | `Agendamento.astro` | **JS** (monta mensagem → `wa.me`) |
| 4 | Serviços (6 cards) | `Servicos.astro` | estático |
| 5 | Plano VetSaúde (assinatura) | `PlanoVetSaude.astro` | estático (CTA WhatsApp); **dimensão variável** `temPlano` |
| 6 | Equipe (grid 4, com CRMV) | `Equipe.astro` | estático (grid flexível) |
| 7 | Pacote Captação | `pacote-captacao/CalculadoraIdade.astro` | **JS** |
| 7 | " | `pacote-captacao/ChecklistEmergencia.astro` | **JS** (ação "Ligar agora" `tel:`) |
| 7 | " | `pacote-captacao/PodeComer.astro` | **JS** (lista fixa `alimentos.json`, nunca IA) |
| 8 | Localização + horários + emergência | `Localizacao.astro` | estático (link Google Maps por endereço, sem API key) |
| 9 | Depoimentos (3) | `Depoimentos.astro` | estático |
| 10 | Footer + LGPD | `Footer.astro` | estático |
| 10 | Botão flutuante de WhatsApp | `BotaoWhatsappFlutuante.astro` | estático (link `wa.me`) |
| — | `<head>` SEO/OG/JSON-LD/canonical/noindex | `layouts/Base.astro` | estático |
| — | `robots.txt` / `sitemap.xml` gerados do `cliente.json` | `pages/robots.txt.ts` / `pages/sitemap.xml.ts` | estático (build) |

**Frase simples:** cada bloco do site é um arquivo; só os que o visitante "usa" (menu, carteirinha, agendamento, ferramentas) têm um pouco de JavaScript.

---

## 6. Ferramentas interativas (regras de comportamento)

### 6.1 Herói 1 — Leitor de carteirinha
- **Faixa-âncora** acima do widget com a estatística **exata** do PRD 7 (cinomose/parvovirose — nunca inventar número).
- **Fluxo front:** upload (`accept="image/*" capture="environment"`) → prévia → **redução no navegador** (lado maior ~1600px, JPEG, alvo ≤1,5 MB) → **checkbox de consentimento LGPD obrigatório** (botão "Analisar" só habilita com imagem + consentimento) → estado "Lendo a carteirinha…" → resultado.
- **Regra de ouro (CRÍTICO):** a **IA só lê** nomes e datas; **o código classifica** "em dia/verificar" com `regras-vacinas.json` comparando com a data atual. Status **sempre por vacina**. **Nunca** afirmar "está tudo em dia / seu pet está protegido" como conclusão geral. Sempre exibir o disclaimer do PRD 7 e um botão grande de WhatsApp (mensagem com as vacinas a verificar já escritas, ou a mensagem "sem pendências").
- **Estados de saída:** resultado normal · ilegível (foto ruim ou não é carteirinha) · limite atingido · erro técnico — todos com saída amigável + botão WhatsApp, nunca travam.
- **Modo demonstração (mock):** aparece **somente** quando `demoMode: true` **ou** a API responde `"nao_configurado"`. Usa **datas relativas à data atual** (V10 há 4 meses = em dia; antirrábica há 14 meses = verificar; gripe canina = verificar) + selo visível "Modo demonstração". O mock **NUNCA** aparece em `"limite"`, `"erro"` ou `"ilegivel"` (seria um resultado falso apresentado como real) — cada um desses estados mostra a própria mensagem + WhatsApp.
- **Acessibilidade:** mudanças de estado com `aria-live`; foco visível; respeitar `prefers-reduced-motion`.

### 6.2 Herói 2 — Agendamento em 30s
- Mini-form: nome do pet (texto) · serviço (select de `servicosAgendamento`) · dia preferido (date) · período (Manhã/Tarde/Qualquer).
- **Dia preferido:** não aceita datas passadas — o campo `date` tem `min` = a data de hoje. A data entra na mensagem no formato **dd/mm/aaaa**.
- Botão monta `https://wa.me/<numero>?text=<msg>` com a mensagem aprovada e abre o WhatsApp. **Só `wa.me`**, nunca a API oficial. Sem backend, sem agenda, sem login.

### 6.3 Pacote Captação (`src/components/pacote-captacao/`, módulo destacável)
1. **Calculadora de idade** por espécie e porte (não usar o mito do ×7). Resultado puxa check-up → WhatsApp. Conteúdo `VALIDAR-VET`.
2. **Checklist de sinais de emergência** — sempre encaminha; ação principal **"Ligar agora" (`tel:`)**, WhatsApp secundário. **Nunca** diz "pode esperar" nem minimiza. `VALIDAR-VET`.
3. **"Meu pet pode comer isso?"** — **lista fixa curada** em `alimentos.json` (tóxicos e seguros-com-ressalva do PRD 10). **Nunca** geração livre de IA. Item fora da lista → orienta perguntar à clínica. `VALIDAR-VET`.

---

## 7. API da carteirinha e proteções de custo

**Arquivo único:** `src/pages/api/carteirinha.ts` (`prerender = false`). Nenhum outro endpoint/função/banco existe.

**Fluxo da função:**
1. **Validação de entrada:** só `POST`; só `image/jpeg | image/png | image/webp`; tamanho máx. 4 MB; exige consentimento marcado; **verifica o `Origin`** (ver regra abaixo).
2. **Falha fechada:** se faltar `ANTHROPIC_API_KEY` **ou** o limitador (Upstash) não estiver configurado, a função **não chama a IA** e responde `{ "status": "nao_configurado" }` → o front cai no mock com selo. **Nunca** roda IA real sem limitador.
3. **Limite de uso (Upstash Redis, `@upstash/ratelimit`):** por IP (5/hora, 10/dia) e global (200/dia). Valores lidos de `cliente.json` (`carteirinha.limites`). IP usado só como chave temporária **com hash** e expiração automática. Estourou → `{ "status": "limite" }`.
4. **Chamada à IA (Anthropic, modelo via `MODELO_IA`, linha Haiku ou equivalente barato):** temperatura 0, `max_tokens` baixo, timeout ~20s. A IA devolve **somente** JSON: legível?, espécie aparente, lista de vacinas com nome lido e data da última dose. A IA **nunca** decide vencimento nem gera texto exibido livremente.
5. **Validação da saída:** JSON validado contra esquema (**zod**); qualquer desvio → tratado como ilegível. Todo texto vindo da IA (nomes) é **escapado** antes de exibir.
6. **Classificação determinística:** o código mapeia cada nome para um tipo e aplica `regras-vacinas.json` (intervalo por vacina/espécie) contra a data atual. Não reconhecida / sem data / data ilegível → "verificar".
7. **Resposta:** formato do PRD 7 (`ok` / `ilegivel` / `limite` / `nao_configurado` / `erro`). Nenhuma resposta expõe detalhes internos.

**Verificação de `Origin` (mesma origem):** a função compara o `Origin` da requisição com o **host da própria requisição** (mesma origem). Em **desenvolvimento**, aceita `localhost`. **Não** compara com o campo `site` do `cliente.json` — ele é um placeholder e muda por cliente; usá-lo bloquearia o site publicado e as URLs de preview da Vercel.

**Privacidade e segredos:** imagem **nunca armazenada** (nem em log) — processada em memória e descartada. Chaves só em variáveis de ambiente (`.env` local fora do Git + painel Vercel), nunca no código, front, logs, mensagens de erro, `docs/` ou chat. Rede de segurança manual (fora do código): limite de gasto mensal na Anthropic; opcional rate limit no Firewall da Vercel.

**Como testar sem chamar a IA real:**
- **Framework:** Vitest, via `npm run test` (comando que o supervisor executa).
- **Modo demonstração** (`demoMode: true`) cobre todo o front sem tocar a API.
- **Testes automatizados** que simulam a resposta da IA (mock do SDK / fixtures de JSON) para exercitar: parsing, validação de esquema (zod), escape de texto, classificação por `regras-vacinas.json`, e os cinco estados de resposta — **sem** chamar a API de verdade.
- Testes de proteção: entrada inválida (método, mime, tamanho, sem consentimento, Origin de outra origem) → rejeição; falta de chave/limitador → `nao_configurado`; estouro de limite → `limite` (simular contador do Upstash).
- IA real só é testada pelo Vinicius depois de preencher o `.env` (ver `docs/deploy.md`).

---

## 8. Design system (`docs/design-system.md`)

Produzido **primeiro** na execução e **aprovado pelo Vinicius antes de qualquer seção** (PRD 6). Deve conter, no mínimo:
- **Paleta** vet clássica, séria, limpa e acolhedora (azul/teal confiável + acento quente para CTA/emergência), como variáveis CSS no `:root`. Sem infantilizar.
- **Tipografia** (famílias, escalas, pesos), **espaçamentos** (escala), **raios/sombras**.
- **Componentes-base:** botões (primário/CTA, emergência, secundário), cards, faixas/banners, campos de formulário, selos (ex.: "Modo demonstração"), estados de foco.
- **Estilo de ícones** (traço consistente, **não** emojis) e **placeholders de foto** (bloco com gradiente/ícone, sem depender de imagem externa; marcados `<!-- FOTO: ... -->`).
- **Logo** (pata + "Clínica VetSaúde") em SVG inline usando variáveis CSS; favicon derivado; **`og.svg`** (imagem de compartilhamento com os tokens da marca + nome + "Emergência 24h"), que o build converte em `og.png`.
- **Lista explícita do que evitar** (tells de site genérico de IA): gradiente roxo padrão, emojis como ícones, herói centralizado genérico, cards todos idênticos, stock-art infantil.
- Diretrizes de **acessibilidade** (contraste AA, foco visível, `prefers-reduced-motion`) e **mobile-first** (testar 360px).

> As **variáveis CSS globais** (`src/styles/global.css`) são propriedade do design system (passo 2 da ordem de construção), não do passo de dados.

**Frase simples:** antes de montar qualquer tela, a gente escreve as "regras de aparência"; o Vinicius aprova, e todo o resto obedece.

---

## 9. SEO técnico

Tudo no `layouts/Base.astro` e nas rotas geradas, alimentado por `cliente.json`. É o diferencial de venda (PRD 13):
- `<title>` e `<meta name="description">` reais e locais (ex.: "Clínica veterinária em Vila Mariana | Consultas, Vacinas e Emergência 24h").
- **Open Graph** (título, descrição, imagem, `locale=pt_BR`) + `<link rel="canonical">` usando o campo `site`.
- **JSON-LD `VeterinaryCare`** com nome, endereço, telefone, horário, URL (de `cliente.json`).
- Um único `<h1>` (a headline do herói); hierarquia de headings correta; HTML semântico; `alt` descritivo.
- Conteúdo textual presente no HTML **gerado no build** (nada de texto injetado por JS, exceto resultados dinâmicos das ferramentas).
- **`robots.txt` (rota gerada do `cliente.json`, não arquivo fixo em `public/`):** **nunca bloqueia o rastreamento**. Se bloqueasse, o Google não leria o `noindex` e poderia indexar o endereço mesmo assim. Quando `demoSite: true`, o robots.txt **não** referencia o sitemap; quando `demoSite: false`, referencia normalmente.
- **`noindex`/demo:** quando `demoSite: true`, o bloqueio do Google é feito **somente** pela meta tag `<meta name="robots" content="noindex,nofollow">` (+ aviso discreto de "site demonstrativo"); removidos quando virar cliente real.
- **Sitemap (rota gerada do `cliente.json`):** quando `demoSite: false`, gerar `sitemap.xml` normalmente; quando `demoSite: true`, **não gerar** o sitemap (nem referenciá-lo no robots.txt).
- **Favicon** (SVG + PNG fallback).
- **Imagem OG:** o design entrega `public/og.svg` com os tokens da marca; um **script no build converte para `og.png` 1200×630** usando **sharp** (dependência comum do Astro). Nada de serviço externo. A URL da OG é absoluta (baseada no campo `site`).

---

## 10. Design e conteúdo

- **Design system primeiro** (seção 8), aprovado, vira referência obrigatória para todas as seções e para o supervisor.
- **Copy:** textos reais e curtos, orientados a benefício, em pt-BR; tom sério e acolhedor, sem infantilizar. Headline, subtítulo, CTAs e faixa de confiança do herói conforme PRD 9.
- **Dados do cliente só em `cliente.json`** — nada hardcoded. Marca em variáveis CSS.
- **Nunca inventar dados** (números, contatos, estatísticas). A estatística da carteirinha usa **exatamente** a fonte do PRD 7. Depoimentos são demo (marcados; trocar por reais em cliente real). Conteúdo de saúde marcado `<!-- VALIDAR-VET: ... -->`. Pontos de foto marcados `<!-- FOTO: ... -->`.

---

## 11. Ordem de construção

Sequência com dependências (a divisão fina em camadas é a etapa 4):

1. **Setup do Astro** — projeto, adapter `@astrojs/vercel`, `package.json` (scripts `dev`/`build`/`test`), `astro.config.mjs`, `vitest.config.ts`, `.gitignore` (com `.env`), `.env.example` (nomes sem valores), layout base vazio. *Dep.: nenhuma.*
2. **Design system** (`docs/design-system.md`) **+ `src/styles/global.css` (variáveis CSS globais)** **+ `public/og.svg`** → **PARAR para aprovação do Vinicius** (o design-system.md). *Dep.: 1.*
3. **Dados e conteúdo** — `cliente.json`, `regras-vacinas.json`, `alimentos.json` (com marcações `VALIDAR-VET`). **Não inclui variáveis CSS globais** (elas são do passo 2). Este passo **não depende visualmente do design** e pode rodar **em paralelo ao passo 2**. *Dep.: 1.*
4. **Seções estáticas** — Nav, Hero (texto), Serviços, Plano, Equipe, Localização, Depoimentos, Footer, botão flutuante. *Dep.: 2, 3.*
5. **Ferramentas interativas** — Agendamento, Pacote Captação (calculadora, checklist, pode comer) e o front do widget de carteirinha (com modo demonstração/mock). *Dep.: 3, 4.*
6. **API da carteirinha** — `src/pages/api/carteirinha.ts` + integração com o widget (falha fechada, limitador Upstash, verificação de Origin por mesma origem, validação com zod, classificação por regras) + testes Vitest que simulam a IA. *Dep.: 3, 5.*
7. **SEO técnico** — title/meta/OG/canonical/JSON-LD/noindex-demo/favicon; `robots.txt.ts` e `sitemap.xml.ts` gerados do `cliente.json`; conversão `og.svg` → `og.png` (sharp) no build. *Dep.: 3, 4.*
8. **Integração final e verificação** — `npm run build` sem erros, `npm run test` verde, checklist de aceite (seção 13), `docs/deploy.md`, revisão de acessibilidade/responsivo. *Dep.: todas.*

**Frase simples:** primeiro as regras visuais (que já trazem as cores) — e, em paralelo, os dados —, depois as telas prontas, depois as ferramentas, depois o mini-servidor da carteirinha, depois o SEO, e por fim juntar tudo e testar.

---

## 12. Matriz de rastreabilidade

Cada requisito do PRD (seções 4, 7–13) e cada critério de aceite (seção 15) → onde é atendido → como verificar.

| Requisito / critério | Onde no plano | Como verificar |
|----------------------|---------------|----------------|
| UX seção 4 (5s, CTA no herói, emergência visível, confiança, sem pop-up, mobile-first) | Seções 5, 10; Hero/Nav | Teste manual a 360px; inspeção de código |
| Carteirinha — fluxo, estados, regras de segurança (7) | Seções 6.1, 7 | Teste manual (demo + IA real pelo Vinicius); testes Vitest que simulam a IA |
| Carteirinha — "IA lê, regra decide"; nunca "tudo em dia" (7) | Seção 7 (classificação determinística) | Inspeção de código + testes Vitest de classificação |
| Carteirinha — mock só em `demo`/`nao_configurado`, nunca em `limite`/`erro`/`ilegivel` | Seções 4, 6.1 | Teste manual dos estados; testes Vitest de estados de resposta |
| Verificação de Origin por mesma origem (não pelo campo `site`) | Seção 7 | Inspeção de código; teste Vitest (Origin de outra origem → rejeita; localhost em dev → aceita) |
| API única + proteções de custo/abuso (7, 11) | Seção 7 | Inspeção de código; testes de validação de entrada, limite e falha fechada |
| Chave só em env; imagem nunca armazenada; texto escapado (7, 11) | Seções 3, 7 | Inspeção de código; grep por segredos; revisão de logs |
| Agendamento em 30s → `wa.me`; data não-passada e formato dd/mm/aaaa (8) | Seção 6.2 | Teste manual (link abre WhatsApp com mensagem; `min` = hoje; data formatada) |
| Demais seções com conteúdo real (9) | Seções 5, 10 | Inspeção visual + conferência com PRD 3/9 |
| Pacote Captação (10) — checklist "Ligar agora"; pode-comer lista fixa | Seção 6.3 | Teste manual; inspeção (sem IA na lista) |
| Requisitos técnicos (11) — estático + 1 função; sem DB (só Upstash contadores); sem localStorage; wa.me/tel:; dados isolados; FOTO/VALIDAR-VET | Seções 3, 4, 5, 10 | `npm run build`; inspeção de código; grep |
| Dimensões variáveis como flags, sem duplicação (12) | Seção 4 (`flags`) | Inspeção de `cliente.json` + comentários no código |
| SEO (13) — title/meta/OG/canonical/JSON-LD/h1/semântica | Seção 9 | Inspeção do HTML gerado; validador de JSON-LD |
| SEO demo — `noindex` bloqueia (robots.txt NÃO bloqueia); sitemap só se `demoSite:false` | Seção 9 | Inspeção do `<head>`, do `robots.txt` gerado e da ausência/presença do `sitemap.xml` conforme `demoSite` |
| Imagem OG PNG 1200×630 gerada no build (sharp), sem serviço externo | Seções 3, 9 | Inspeção do build (`og.png` gerado); prévia no WhatsApp |
| DoD (15): build sem erros / única rota dinâmica | Seção 11 (passo 8) | `npm run build` |
| DoD (15): demoMode fluxo completo com selo | Seções 6.1, 7 | Teste manual |
| DoD (15): IA real — legível/ilegível, nunca "tudo em dia" | Seção 7 | Teste do Vinicius com `.env`; testes Vitest simulando a IA |
| DoD (15): falha fechada sem chave/limitador | Seção 7 | Teste Vitest (`nao_configurado`) |
| DoD (15): limite (6ª análise → "limite") + teto global | Seção 7 | Teste Vitest simulando contador Upstash |
| DoD (15): sem segredos no código/logs/repo | Seções 3, 7 | grep + revisão |
| DoD (15): responsivo 360px, foco, reduced-motion | Seções 6, 8, 10 | Teste manual + inspeção |
| DoD (15): segue design-system aprovado | Seções 8, 10 | Revisão do supervisor |
| DoD (15): `docs/deploy.md` passo a passo | Seção 11 (passo 8) | Inspeção do documento |

---

## 13. Critérios de aceite globais

- `npm run build` passa **sem erros**; todas as páginas estáticas; única rota dinâmica `/api/carteirinha`.
- `npm run test` (Vitest) passa **sem falhas**.
- Todas as regras do CLAUDE.md respeitadas: backend único, sem `localStorage`/`sessionStorage`, WhatsApp só `wa.me`, emergência `tel:` como ação principal, dados só em `cliente.json`, segredos só em env, imagem nunca armazenada, nunca "tudo em dia".
- Checklist completo do PRD seção 15 atendido (ver matriz).
- `design-system.md` aprovado antes das seções; visual segue-o.
- `docs/deploy.md` escrito em linguagem simples.

---

## 14. Riscos

| Risco | Tipo | Mitigação |
|-------|------|-----------|
| Custo/abuso da IA da carteirinha (chave exposta ou flood) | Técnico/negócio | Falha fechada sem limitador; limite por IP + teto global (Upstash); imagem reduzida, `max_tokens` baixo, timeout; segredos só em env; limite de gasto na Anthropic; opcional Firewall Vercel |
| Leitura errada da IA gerar falso "em dia" (perigo clínico) | Negócio/segurança | "IA lê, regra decide"; status sempre por vacina; disclaimer obrigatório; nunca conclusão "tudo em dia"; sempre encaminha ao WhatsApp |
| **Mock exibido como resultado real** | Negócio/segurança | Mock aparece **somente** em `demo`/`nao_configurado`; em `limite`, `erro` e `ilegivel` mostra a própria mensagem + WhatsApp, nunca o mock; teste automatizado por estado |
| **Origin bloqueando o próprio site em produção** | Técnico | Comparar `Origin` com o **host da própria requisição** (mesma origem), não com o campo `site` (placeholder que muda por cliente); aceitar `localhost` em dev; cobre site publicado e previews da Vercel |
| **robots.txt impedindo a leitura do `noindex`** | Negócio/SEO | robots.txt **nunca** bloqueia o rastreamento; o bloqueio do Google em demo é feito só pelo `noindex`; robots.txt/sitemap gerados do `cliente.json`, sitemap só quando `demoSite:false` |
| Texto malicioso embutido na imagem da carteirinha | Técnico/segurança | JSON validado contra esquema (zod); todo texto da IA escapado antes de exibir; desvio → tratado como ilegível |
| Vazamento de segredo em log/erro/commit | Técnico | Env fora do Git; `.env.example` sem valores; nenhuma resposta expõe detalhes; revisão + grep antes de entregar |
| Imagem OG em SVG não renderiza prévia no WhatsApp | Negócio | `og.svg` (design) convertido em `og.png` 1200×630 no build via sharp, em URL absoluta; sem serviço externo |
| "Tells" de site genérico feito por IA reduzem credibilidade | Negócio | Lista do que evitar no design-system; ícones de traço (não emojis); cards variados; sem stock-art infantil |
| Conteúdo de saúde impreciso (idade, emergência, alimentos) | Negócio/legal | Tudo marcado `VALIDAR-VET`; lista de alimentos fixa e curada; em cliente real validar com veterinário |
| SEO fraco por ser single page | Negócio | HTML semântico + JSON-LD local + conteúdo no build; add-on de páginas/blog em fase futura (Astro facilita) |
| Depoimentos demo publicados como reais em cliente | Negócio/legal | Marcados como demo; regra explícita de substituir por avaliações reais antes de virar cliente |
| Limite da Vercel (~4,5 MB por requisição) estoura no upload | Técnico | Redução no navegador (~1600px, ≤1,5 MB) + limite de 4 MB na função |

---

## 15. Perguntas em aberto

1. ~~**URL do projeto na Vercel**~~ **(RESOLVIDA)** — usar provisoriamente `https://site-clinica-vet-modelo1.vercel.app` no campo `site`. O Vinicius confirma ou troca na hora do deploy (é só mudar no `cliente.json`).
2. ~~**Método de geração da imagem Open Graph**~~ **(RESOLVIDA)** — o design produz `public/og.svg` com os tokens da marca; um script no build converte para `og.png` 1200×630 usando **sharp** (dependência comum do Astro). Sem serviço externo.
3. **Provisionamento da IA real:** criação da chave Anthropic com limite de gasto, conexão do Upstash e cadastro das variáveis de ambiente dependem do Vinicius (fora do código). Até lá, tudo roda e é testado em modo demonstração + testes que simulam a IA. *(pendência do Vinicius — não bloqueia o plano)*
4. **Aprovação do `design-system.md`** antes de qualquer seção — depende do Vinicius. *(pendência do Vinicius — não bloqueia o plano)*

Nenhuma contradição encontrada entre PRD e briefing (o briefing complementa o PRD). Se alguma surgir na execução, prevalece o PRD e a contradição é registrada aqui.

---

## 16. Histórico de revisões

| Data | Autor | Mudança |
|------|-------|---------|
| 2026-09-29 | Subagente planejador | Criação do plano (RASCUNHO) a partir do PRD e do briefing aprovado. |
| 2026-09-29 | Subagente planejador | Revisão 1 (correções do Vinicius), Status mantido RASCUNHO: **(1)** eliminação de flags duplicadas no `cliente.json` — removidos `carteirinha.iaReal`, `carteirinha.ativa` e `especies` de topo; `demoMode` passou a `false` (padrão) com comportamento e regra "mock só em demo/nao_configurado" detalhados nas seções 4, 6.1 e 7. **(2)** Verificação de `Origin` por mesma origem (host da requisição, localhost em dev), não pelo campo `site` (seção 7). **(3)** `robots.txt` e `sitemap` gerados do `cliente.json` como rotas pré-renderizadas; `noindex` é o único bloqueio em demo; robots.txt não bloqueia rastreamento; sitemap só quando `demoSite:false`; removido `robots.txt` de `public/` (seções 3, 5, 9). **(4)** Vitest definido como framework + script `test`; adicionadas dependências `vitest`, `zod` e `sharp` (seções 3, 7). **(5)** Variáveis CSS globais movidas para o passo 2 (design system); passo 3 (dados) não depende do design e roda em paralelo (seções 8, 11). **(6)** Perguntas 1 e 2 resolvidas (URL provisória; OG via `og.svg`→`og.png` com sharp); 3 e 4 mantidas como pendências do Vinicius. **(7)** Agendamento: data mínima = hoje e formato dd/mm/aaaa (seção 6.2). **(8)** Matriz (12) e riscos (14) atualizados com os três novos riscos (mock como resultado real; Origin bloqueando o site em produção; robots.txt impedindo a leitura do noindex). |
