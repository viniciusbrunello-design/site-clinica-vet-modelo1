# 04 — Camadas de execução do Site 1 (Clínica VetSaúde)

**Status:** `APROVADO em 2026-09-30`

> Só o Vinicius muda este status para `APROVADO`. Enquanto estiver `RASCUNHO`, nenhuma camada de execução começa.
> Este documento divide o plano aprovado (`docs/03-plano.md`, APROVADO em 2026-09-29) em camadas independentes com **posse exclusiva de arquivos**, **contratos explícitos** entre elas, **ordem de execução em ondas com portões de aprovação** e **critérios de aceite verificáveis**. Fontes: `docs/00-PRD.md` (prioridade máxima), `docs/02-briefing.md`, `docs/03-plano.md`, `CLAUDE.md`.
> Quem distribui o trabalho aos subagentes é o orquestrador (sessão principal). Este arquivo é a especificação; ele não executa nada.

---

## 1. Visão geral das camadas

| ID | Camada | Agente responsável | Onda | Depende de | Objetivo (uma frase) |
|----|--------|--------------------|------|------------|----------------------|
| C1 | Fundação e configuração | `camada-fundacao` | 1 | — | Montar o esqueleto Astro (config, dependências, layout base, index vazio, utilitários compartilhados) que todas as camadas usam. |
| C2 | Design system e tokens | `camada-design` | 2 | C1 | Definir a régua visual (`docs/design-system.md`), os tokens CSS globais, os componentes-base e os ativos de marca, e parar para aprovação do Vinicius. |
| C3 | Dados e conteúdo | `camada-dados-conteudo` | 2 | C1 | Produzir `cliente.json`, `regras-vacinas.json`, `alimentos.json` e toda a copy real, sem inventar dados. |
| C4 | Seções estáticas | `camada-secoes` | 3 | C2 (aprovado), C3 | Construir os componentes `.astro` das seções que o visitante lê (nav, herói-texto, serviços, plano, equipe, localização, depoimentos, footer, botão flutuante). |
| C5 | Ferramentas interativas | `camada-interativos` | 4 | C3, C4 | Implementar o front das ferramentas com JS: widget de carteirinha (com mock), agendamento em 30s e Pacote Captação. |
| C6 | API da carteirinha | `camada-api-carteirinha` | 5 | C1, C3, C5 | Implementar a única função serverless `/api/carteirinha` (IA lê, regra decide), proteções de custo/abuso, testes Vitest e `docs/deploy.md`. |
| C7 | SEO técnico | `camada-seo` | 4 | C3, C4 | Implementar title/meta/OG/canonical/JSON-LD/noindex-demo, `robots.txt` e `sitemap.xml` gerados de `cliente.json`. |
| C8 | Integração final e verificação | `camada-fundacao` | 6 | C1–C7 | Compor `index.astro` e o `<head>`, rodar build/testes, checklist de aceite e verificações transversais (grep de segredos, `localStorage`). |

> Todas as camadas foram atribuídas a um agente que existe em `.claude/agents/`. O agente genérico `executor-camada` não foi necessário: cada camada tem um agente especializado.

---

## 2. Ondas de execução e portões de aprovação

```
Onda 1  ── C1 Fundação
              │
Onda 2  ── C2 Design  ‖  C3 Dados        (rodam em paralelo)
              │
        ▲ PORTÃO 1 — Vinicius aprova docs/design-system.md
        │  Nenhuma camada visual (C4, C5, C7) começa antes disso.
              │
Onda 3  ── C4 Seções estáticas
              │
Onda 4  ── C5 Interativos  ‖  C7 SEO      (rodam em paralelo)
              │
Onda 5  ── C6 API da carteirinha
              │
Onda 6  ── C8 Integração final
              │
        ▲ PORTÃO 2 — Vinicius aprova a entrega final (build/testes/checklist)
```

**Regras das ondas:**
- **Onda 1 (C1):** roda sozinha. Sem ela nada compila.
- **Onda 2 (C2 ‖ C3):** design e dados rodam em paralelo — o passo de dados **não depende visualmente** do design (plano, seção 11, passo 3). **C2 termina em portão obrigatório:** o orquestrador para e só libera a Onda 3 depois que o Vinicius aprovar `docs/design-system.md` vendo a `design-preview.astro` (PRD seção 6). C3 pode terminar antes ou depois do portão; não depende dele.
- **Onda 3 (C4):** exige C2 **aprovado** e C3 concluído.
- **Onda 4 (C5 ‖ C7):** interativos e SEO rodam em paralelo — não se tocam em arquivos e ambos dependem apenas de C3 + C4.
- **Onda 5 (C6):** depende de C3 (regras-vacinas/limites), de C5 (contrato de resposta consumido pelo widget) e de C1 (dependências instaladas).
- **Onda 6 (C8):** integração final por `camada-fundacao`; depende de todas. Termina no **Portão 2** (aprovação final do Vinicius).
- Ao fim de cada onda com paralelismo, o **orquestrador** roda o build completo (`npm run build`) e `npm run test`. Dentro de uma onda paralela, cada camada roda apenas `npx astro check` (regra dos agentes).

---

## 3. Mapa de posse de arquivos

Cada arquivo/pasta tem **exatamente uma** camada dona (cria/edita). As demais só leem. Sem duplicatas.

| Arquivo / pasta | Camada dona | Quem apenas lê |
|-----------------|-------------|----------------|
| `astro.config.mjs` | C1 | C6 |
| `package.json` | C1 | — |
| `vitest.config.ts` | C1 | C6 |
| `tsconfig.json` | C1 | — |
| `.gitignore` | C1 | — |
| `.env.example` | C1 | C6 |
| `src/lib/links.ts` | C1 | C4, C5 |
| `src/lib/escape.ts` | C1 | C5, C6 |
| `src/layouts/Base.astro` | C1 | — |
| `src/pages/index.astro` | C1 | — |
| `scripts/gerar-og.mjs` | C1 | — |
| `public/favicon.svg` | C2 | C7 |
| `public/favicon.png` | C2 | C7 |
| `public/og.svg` | C2 | C1, C7 |
| `public/og.png` (gerado por `npm run og`, commitado) | C1 (gera via `scripts/gerar-og.mjs`) | C7 |
| `docs/design-system.md` | C2 | C4, C5, C7, supervisor |
| `src/styles/global.css` | C2 | (importado por `Base.astro` em C1/C8) |
| `src/components/base/` (Botao, Card, Faixa, CampoFormulario, Selo, PlaceholderFoto, Icone, Container) | C2 | C4, C5 |
| `src/pages/design-preview.astro` | C2 | — |
| `src/data/cliente.json` | C3 | C1, C4, C5, C6, C7 |
| `src/data/regras-vacinas.json` | C3 | C6 |
| `src/data/alimentos.json` | C3 | C5 |
| `src/components/Nav.astro` | C4 | — |
| `src/components/Hero.astro` | C4 | — |
| `src/components/Servicos.astro` | C4 | — |
| `src/components/PlanoVetSaude.astro` | C4 | — |
| `src/components/Equipe.astro` | C4 | — |
| `src/components/Localizacao.astro` | C4 | — |
| `src/components/Depoimentos.astro` | C4 | — |
| `src/components/Footer.astro` | C4 | — |
| `src/components/BotaoWhatsappFlutuante.astro` | C4 | — |
| `src/components/CarteirinhaWidget.astro` | C5 | — |
| `src/components/Agendamento.astro` | C5 | — |
| `src/components/pacote-captacao/CalculadoraIdade.astro` | C5 | — |
| `src/components/pacote-captacao/ChecklistEmergencia.astro` | C5 | — |
| `src/components/pacote-captacao/PodeComer.astro` | C5 | — |
| `src/pages/api/carteirinha.ts` | C6 | — |
| `src/lib/carteirinha/` (schema, classificar, ratelimit, validar-entrada, ia, tipos) | C6 | — |
| `src/lib/carteirinha/**/*.test.ts` (testes Vitest) | C6 | — |
| `docs/deploy.md` | C6 | — |
| `src/components/Seo.astro` | C7 | (renderizado por `Base.astro` em C8) |
| `src/pages/robots.txt.ts` | C7 | — |
| `src/pages/sitemap.xml.ts` | C7 | — |

**Arquivos com integração de dois donos, resolvidos por posse única:**
- `src/layouts/Base.astro` e `src/pages/index.astro` pertencem **só a C1**. C1 os cria como esqueleto na Onda 1 e os finaliza na Onda 6 (integração): importa `global.css`, renderiza `<Seo />` (de C7) no `<head>` e compõe os componentes de C4/C5. Nenhuma outra camada edita esses dois arquivos.
- `src/components/Seo.astro` pertence **só a C7**; C1 apenas o renderiza dentro de `Base.astro` na Onda 6.
- `src/styles/global.css` pertence **só a C2**; `Base.astro` (C1) o importa.
- `public/og.png` é gerado **localmente** pelo script `scripts/gerar-og.mjs` (C1) via `npm run og`, a partir de `public/og.svg` (C2); é **commitado** no repositório (não vai no `.gitignore`) e regenerado com `npm run og` quando a marca muda. O **build não depende** do script.
- `.env.example` pertence **só a C1**; os nomes das variáveis são um contrato definido por C6 (ver §5.6). Se C6 precisar de uma variável nova, é pedido de mudança de contrato para C1.

---

## 4. Ondas × dependências (resumo visual)

| Onda | Camadas | Pré-requisito para liberar a onda |
|------|---------|-----------------------------------|
| 1 | C1 | — |
| 2 | C2 ‖ C3 | C1 concluída |
| — | **PORTÃO 1** | Vinicius aprova `design-system.md` |
| 3 | C4 | C2 aprovado + C3 concluída |
| 4 | C5 ‖ C7 | C4 concluída |
| 5 | C6 | C5 concluída (+ C1, C3) |
| 6 | C8 | C2–C7 concluídas |
| — | **PORTÃO 2** | Vinicius aprova a entrega final |

---

## 5. Especificação de cada camada

### C1 — Fundação e configuração
**Agente:** `camada-fundacao` · **Onda:** 1 · **Depende de:** —

**Objetivo:** montar o esqueleto Astro que todas as camadas usam e os utilitários compartilhados, sem construir seção nenhuma.

**Pode criar/editar (posse exclusiva):**
`astro.config.mjs`, `package.json`, `vitest.config.ts`, `tsconfig.json`, `.gitignore`, `.env.example`, `src/lib/links.ts`, `src/lib/escape.ts`, `src/layouts/Base.astro`, `src/pages/index.astro`, `scripts/gerar-og.mjs`. Também é dono de `public/og.png` (artefato **commitado**, gerado pelo script `scripts/gerar-og.mjs` via `npm run og`, **fora do build**).

**Pode apenas ler:** `public/og.svg` (C2), `src/data/cliente.json` (C3), especificação de C6 e C7.

**Depende de / consome:** nada na Onda 1. Na Onda 6 (C8) consome tudo.

**Entrega / contratos que expõe:**
- `astro.config.mjs` com adapter `@astrojs/vercel`, `site` lido da URL; **todas as páginas pré-renderizadas**; única rota dinâmica permitida `/api/carteirinha`. **O build não gera `og.png`.**
- **`scripts/gerar-og.mjs`** + comando `npm run og` no `package.json`: converte `public/og.svg` → `public/og.png` **1200×630** com `sharp`. Roda **localmente sob demanda**, nunca no build; o `og.png` resultante é commitado no repositório.
- `package.json` com scripts `dev`, `build`, `check` (`astro check`), `test` (`vitest run`), `og` (`node scripts/gerar-og.mjs`); dependências instaladas conforme contrato de C6: `astro`, `@astrojs/vercel`, `@astrojs/check`, `typescript`, `@anthropic-ai/sdk`, `@upstash/ratelimit`, `@upstash/redis`, `zod`, `sharp`, `vitest`.
- `.gitignore` inclui `.env` e `.env.*` (exceto `.env.example`). **`public/og.png` não é ignorado** (é commitado).
- `.env.example` com os nomes **sem valores**: `ANTHROPIC_API_KEY`, `MODELO_IA`, `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`.
- **`src/lib/links.ts`** (usado por C4, C5): 
  - `montarLinkWhatsapp(numero: string, mensagem: string): string` → `https://wa.me/<numero>?text=<encodeURIComponent(mensagem)>`.
  - `montarLinkTelefone(telefone: string): string` → `tel:` com dígitos.
  - `preencherMensagem(template: string, valores: Record<string, string>): string` → substitui `{chave}` pelos valores (para `mensagensWhatsapp`).
- **`src/lib/escape.ts`** (usado por C5, C6): `escaparHtml(texto: string): string`.
- `src/layouts/Base.astro`: esqueleto com `<html lang="pt-BR">`, `charset`, `viewport`, `<slot />`. Ponto marcado por comentário `<!-- SEO: <Seo /> entra aqui na integração (C8) -->` e `<!-- CSS: import global.css na integração (C8) -->`.
- `src/pages/index.astro`: esqueleto vazio usando `Base.astro`, com comentários da ordem das seções (PRD 5). Composição real é feita em C8.

**Critérios de aceite:**
- `C1.1` `npm run build` passa sem erros com o esqueleto. **Verificar:** saída do comando.
- `C1.2` `astro.config.mjs` usa adapter `@astrojs/vercel` e nenhuma página, além do esqueleto de `/api`, é dinâmica. **Verificar:** inspeção do config + `dist/` só com HTML estático (rastreabilidade item 16).
- `C1.3` `.gitignore` ignora `.env`, `.env.*` (menos `.env.example`) e **não** ignora `public/og.png`. **Verificar:** `git check-ignore .env` retorna `.env`; `git check-ignore public/og.png` **não** retorna nada (arquivo versionado).
- `C1.4` `.env.example` contém exatamente os 4 nomes de variáveis, **sem valores**. **Verificar:** inspeção + `grep` não encontra `=` com conteúdo após.
- `C1.5` `src/lib/links.ts` exporta `montarLinkWhatsapp`, `montarLinkTelefone`, `preencherMensagem` com as assinaturas acima; `montarLinkWhatsapp` só gera `wa.me` (nunca API oficial). **Verificar:** inspeção de código (rastreabilidade item 8, 11).
- `C1.6` `src/lib/escape.ts` exporta `escaparHtml`. **Verificar:** inspeção.
- `C1.7` `package.json` tem os scripts `dev`/`build`/`check`/`test`/`og` e as dependências listadas. **Verificar:** inspeção.
- `C1.8` `npm run og` gera `public/og.png` **1200×630** a partir de `public/og.svg`; o **build não depende** do script (passa mesmo sem rodar `npm run og`). **Verificar:** rodar `npm run og` e conferir as dimensões do PNG; rodar `npm run build` sem executar o script e confirmar que passa.

**Fora do escopo:** qualquer componente de seção, CSS de marca (é C2), conteúdo/dados (C3), código da função da carteirinha (C6), tags de SEO (C7). C1 **não** define valores de tokens nem cores.

---

### C2 — Design system e tokens
**Agente:** `camada-design` · **Onda:** 2 (paralela a C3) · **Depende de:** C1 · **PORTÃO 1 ao terminar**

**Objetivo:** definir a régua visual obrigatória do site e entregá-la de forma que o Vinicius aprove **vendo**, não lendo códigos de cor.

**Pode criar/editar (posse exclusiva):**
`docs/design-system.md`, `src/styles/global.css`, `src/components/base/` (todos os componentes-base), `public/favicon.svg`, `public/favicon.png`, `public/og.svg`, `src/pages/design-preview.astro`.

**Pode apenas ler:** `docs/00-PRD.md` (2,4,6), `docs/02-briefing.md`, este documento.

**Depende de / consome:** de C1, o esqueleto Astro (para a `design-preview.astro` compilar).

**Entrega / contratos que expõe:**
- **`docs/design-system.md`** completo (paleta com uso de cada cor; tipografia; espaçamentos; grid/breakpoints; raios/sombras; estilo de ícones de traço — **nunca** emojis; estilo dos placeholders de foto; componentes-base com estados hover/foco/desabilitado/erro; lista explícita de "tells" a evitar; diretrizes de acessibilidade AA, foco visível, `prefers-reduced-motion`; mobile-first 360px).
- **Contrato de tokens em `src/styles/global.css`** — variáveis no `:root` com **estes nomes obrigatórios** (design define os valores; pode acrescentar outras, mas estas precisam existir, pois C4/C5 as consomem):
  - Cores: `--cor-primaria`, `--cor-primaria-escura`, `--cor-primaria-clara`, `--cor-acento` (CTA principal), `--cor-emergencia`, `--cor-texto`, `--cor-texto-suave`, `--cor-fundo`, `--cor-fundo-alt`, `--cor-borda`, `--cor-sucesso` (status "em dia"), `--cor-atencao` (status "verificar").
  - Tipografia: `--fonte-titulo`, `--fonte-texto`.
  - Espaçamento: escala `--espaco-1` … `--espaco-8`.
  - Forma: `--raio-sm`, `--raio-md`, `--raio-lg`, `--sombra-sm`, `--sombra-md`.
  - Layout: `--largura-container`.
- **Contrato de componentes-base em `src/components/base/`** (nomes de arquivo e props exatos que C4/C5 importam):
  - `Botao.astro` — props: `variante: "primario" | "cta" | "emergencia" | "secundario"`, `href?`, `type?`, `tamanho?: "md" | "lg"`; conteúdo via slot.
  - `Card.astro` — props: `destaque?: boolean`; conteúdo via slots.
  - `Faixa.astro` — props: `variante: "info" | "emergencia" | "destaque"`; conteúdo via slot.
  - `CampoFormulario.astro` — props: `label`, `id`, `tipo`, `nome`; suporta select/date/text.
  - `Selo.astro` — props: `texto` (ex.: "Modo demonstração").
  - `PlaceholderFoto.astro` — props: `rotulo`, `proporcao?`; renderiza bloco com gradiente/ícone e emite comentário `<!-- FOTO: {rotulo} -->`.
  - `Icone.astro` — props: `nome`, `tamanho?`; expõe um conjunto fixo de ícones de traço. **Lista mínima obrigatória de nomes** (fixada agora porque C3 roda em paralelo e depende dela; C2 pode acrescentar outros, mas todos estes precisam existir): `estetoscopio`, `seringa`, `exame`, `bisturi`, `internacao`, `dente`, `emergencia`, `pata`, `telefone`, `whatsapp`, `relogio`, `mapa`, `calendario`, `check`, `alerta`, `upload`, `camera`, `escudo`, `estrela`, `menu`, `fechar`. Os mesmos nomes são listados no `design-system.md`. C3 usa **somente** nomes desta lista.
  - `Container.astro` — wrapper de largura máxima.
- **Ativos de marca:** `favicon.svg`, `favicon.png` (derivados do logo), `og.svg` (com tokens da marca + nome + "Emergência 24h", com **todo o texto convertido em caminhos/paths**, sem depender de fontes instaladas), que C1 converte em `og.png` via `npm run og`.
- **`design-preview.astro`** com `<meta name="robots" content="noindex,nofollow">`, mostrando paleta, tipografia e todos os componentes-base em seus estados.

**Critérios de aceite:**
- `C2.1` `docs/design-system.md` contém todas as seções listadas acima, incluindo a lista de "tells" a evitar. **Verificar:** inspeção do documento (rastreabilidade itens 22, 23).
- `C2.2` `global.css` define **todos** os tokens do contrato com os nomes exatos. **Verificar:** `grep` de cada nome de variável no arquivo.
- `C2.3` cada componente-base existe com os nomes de arquivo e props do contrato; `Icone.astro` implementa **todos** os nomes da lista mínima obrigatória. **Verificar:** inspeção de `src/components/base/` + conferir a presença de cada nome da lista (`estetoscopio`, `seringa`, `exame`, `bisturi`, `internacao`, `dente`, `emergencia`, `pata`, `telefone`, `whatsapp`, `relogio`, `mapa`, `calendario`, `check`, `alerta`, `upload`, `camera`, `escudo`, `estrela`, `menu`, `fechar`) em `Icone.astro`.
- `C2.4` nenhum emoji é usado como ícone; ícones são de traço via `Icone.astro`. **Verificar:** inspeção + `grep` por emojis nos componentes-base.
- `C2.5` `design-preview.astro` compila, tem `noindex` e renderiza paleta/tipografia/componentes. **Verificar:** `npx astro check` + abrir a página.
- `C2.6` `favicon.svg`, `favicon.png` e `og.svg` existem e usam os tokens da marca; o `og.svg` tem **todo o texto em caminhos/paths** (não depende de fontes instaladas). **Verificar:** inspeção dos arquivos + `grep` confirma **nenhum** elemento `<text>` em `public/og.svg`.
- `C2.7` diretrizes de acessibilidade (AA, foco visível, `prefers-reduced-motion`) e mobile-first estão escritas e refletidas nos estados dos componentes-base. **Verificar:** inspeção (rastreabilidade item 22).

**Portão:** ao terminar, **C2 para**. O orquestrador apresenta a `design-preview` ao Vinicius. Onda 3 só é liberada com aprovação.

**Fora do escopo:** seções do site, ferramentas interativas, conteúdo/dados, SEO. Não edita `Base.astro` nem `index.astro`.

---

### C3 — Dados e conteúdo
**Agente:** `camada-dados-conteudo` · **Onda:** 2 (paralela a C2) · **Depende de:** C1

**Objetivo:** produzir todos os dados do cliente e toda a copy real, isolados em arquivos JSON, sem inventar nada e sem depender do visual.

**Pode criar/editar (posse exclusiva):**
`src/data/cliente.json`, `src/data/regras-vacinas.json`, `src/data/alimentos.json`.

**Pode apenas ler:** `docs/00-PRD.md` (3, 7–10, 12), `docs/02-briefing.md`, esquema em `docs/03-plano.md` (§4).

**Depende de / consome:** nada além das fontes documentais.

**Entrega / contratos que expõe (o esquema que C1/C4/C5/C6/C7 consomem):**
- **`cliente.json`** exatamente com o esquema do plano §4: `nome`, `site`, `demoMode` (`false`), `demoSite` (`true`), `avisoDemo` (texto do aviso **visível** de site demonstrativo, exibido por C4 quando `demoSite:true`), `whatsapp` (dígitos, número do Vinicius da demo), `telefone`, `email`, `endereco {logradouro,bairro,cidade,uf}`, `horarios[{dias,horario}]`, `provaSocial {estrelas,anos,tutores}`, `servicos[{icone,titulo,frase}]` (6), `servicosAgendamento[]`, `equipe[{nome,especialidade,crmv}]` (4), `plano {nome,inclui[],descricao}`, `depoimentos[{autor,texto,estrelas,tema}]` (3), `mensagensWhatsapp {agendamento,plano,carteirinhaPendencias,carteirinhaSemPendencias,emergencia,generico,ferramentas}` (textos exatos do plano §4, com placeholders `{servico}`, `{pet}`, `{data}`, `{periodo}`, `{vacinas}`), `carteirinha {limites {porIpHora:5,porIpDia:10,globalDia:200}}`, `lgpd {controladora,contato}`, `flags {heroiPrincipal:"carteirinha", temCarteirinha:true, emergencia24h:true, especies:["cão","gato"], temPlano:true, pacote:"captacao", temBanhoTosa:false}`.
  - **Sem** flags duplicadas: **não** existem `carteirinha.ativa`, `carteirinha.iaReal` nem `especies` de topo (fonte única = `flags.especies`).
  - `servicos[].icone` deve usar **apenas** nomes da lista mínima obrigatória de `Icone.astro` fixada no contrato de C2 (§5-C2): `estetoscopio`, `seringa`, `exame`, `bisturi`, `internacao`, `dente`, `emergencia`, `pata`, `telefone`, `whatsapp`, `relogio`, `mapa`, `calendario`, `check`, `alerta`, `upload`, `camera`, `escudo`, `estrela`, `menu`, `fechar`. Caso precise de um nome fora da lista, é pedido de mudança de contrato a C2.
- **`regras-vacinas.json`** (`VALIDAR-VET`): para cada vacina de cães e gatos — nome canônico, apelidos/variações que aparecem em carteirinhas, espécie, intervalo de reforço. Usado **só pelo código** de C6, nunca pela IA.
- **`alimentos.json`** (`VALIDAR-VET`): lista fixa curada — tóxicos (chocolate, uva/passa, cebola, alho, xilitol, macadâmia, álcool) e seguros-com-ressalva (cenoura, maçã sem semente, banana), conforme PRD 10.
- Toda a **copy** que as seções renderizam (H1, subtítulo, CTAs, frases de serviços, depoimentos etc.) — textos exatos do PRD onde ele os define.

**Critérios de aceite:**
- `C3.1` `cliente.json` é JSON válido e segue o esquema do plano §4, sem campos duplicados proibidos; **todo `servicos[].icone` usa somente nomes da lista mínima obrigatória de `Icone.astro` (§5-C2)**. **Verificar:** parse do JSON + `grep` garante ausência de `carteirinha.ativa`, `carteirinha.iaReal`, `"especies"` fora de `flags`; conferir cada `servicos[].icone` contra a lista de ícones (rastreabilidade item 12).
- `C3.2` `mensagensWhatsapp` contém os 7 textos com os placeholders exatos. **Verificar:** inspeção contra o plano §4 (rastreabilidade item 8).
- `C3.3` `carteirinha.limites` = `{porIpHora:5, porIpDia:10, globalDia:200}`. **Verificar:** inspeção (rastreabilidade item 20).
- `C3.4` `flags` tem exatamente as 7 dimensões variáveis do plano com os valores fixos deste cliente. **Verificar:** inspeção (rastreabilidade item 12).
- `C3.5` estatística da carteirinha e demais números vêm do PRD; nenhum dado inventado; `whatsapp` = número do Vinicius da demo. **Verificar:** conferência com PRD 3/7 e briefing (rastreabilidade item 9).
- `C3.6` `regras-vacinas.json` e `alimentos.json` estão marcados `VALIDAR-VET` e a lista de alimentos bate com o PRD 10. **Verificar:** `grep VALIDAR-VET` + inspeção (rastreabilidade itens 10, 11).
- `C3.7` nenhum texto diz "está tudo em dia", "pode esperar" ou minimiza emergência. **Verificar:** `grep` por essas expressões retorna vazio.
- `C3.8` depoimentos marcados como demonstrativos. **Verificar:** inspeção.
- `C3.9` `cliente.json` contém o campo `avisoDemo` com o texto do aviso de site demonstrativo (usado por C4). **Verificar:** inspeção do JSON (rastreabilidade item 14).

**Fora do escopo:** qualquer arquivo `.astro`, CSS, tokens (C2), lógica. C3 **não** escreve componentes; só dados e texto.

---

### C4 — Seções estáticas
**Agente:** `camada-secoes` · **Onda:** 3 · **Depende de:** C2 (aprovado), C3

**Objetivo:** construir os componentes das seções que o visitante lê, 100% pelos tokens/componentes de C2 e lendo dados de C3.

**Pode criar/editar (posse exclusiva):**
`src/components/Nav.astro`, `Hero.astro`, `Servicos.astro`, `PlanoVetSaude.astro`, `Equipe.astro`, `Localizacao.astro`, `Depoimentos.astro`, `Footer.astro`, `BotaoWhatsappFlutuante.astro`.

**Pode apenas ler:** `docs/design-system.md`, `src/styles/global.css` e `src/components/base/` (C2); `src/data/cliente.json` (C3); `src/lib/links.ts` (C1).

**Depende de / consome:**
- Tokens e componentes-base de C2 (nomes e props do §5-C2).
- `cliente.json` de C3 (nomes de campo do §5-C3, incluindo `demoSite` e `avisoDemo` para o aviso visível de site demonstrativo).
- `montarLinkWhatsapp`, `montarLinkTelefone`, `preencherMensagem` de C1.

**Entrega / contratos que expõe (nomes exatos que C1/C8 compõem em `index.astro`):**
- Componentes exportados por default: `Nav`, `Hero`, `Servicos`, `PlanoVetSaude`, `Equipe`, `Localizacao`, `Depoimentos`, `Footer`, `BotaoWhatsappFlutuante`.
- **Contrato de âncoras** (ids que a `Nav` referencia e o Hero/CTA rolam): `#servicos`, `#vacinas` (widget carteirinha, na seção herói), `#agendar`, `#plano`, `#equipe`, `#onde-estamos`. C4 aplica esses `id` às seções que possui; onde a âncora aponta para conteúdo de C5 (carteirinha/agendamento), o `id` fica na seção herói / ponto de composição de C4 e o componente de C5 é injetado por C8 (via slot no `Hero` ou composição no `index.astro`).
- `Hero.astro` contém o **único `<h1>`** do site (headline do PRD 9) e expõe um **slot nomeado do Astro** `<slot name="interativo" />` no ponto à direita da headline, onde C8 injeta o `CarteirinhaWidget` (C5) a partir do `index.astro`. C4 **não** insere componentes de C5 dentro de seus próprios arquivos.
- O `Agendamento` (C5) e as demais ferramentas do Pacote Captação são compostas por C8 **diretamente no `index.astro`** (na ordem do PRD 5), **não** dentro dos arquivos de C4 e **sem** depender de comentários dentro dos arquivos de C4.
- **Aviso de site demonstrativo:** quando `cliente.json.demoSite:true`, C4 renderiza um aviso discreto e **visível** (no `Footer.astro` ou numa faixa fina no topo, ex.: via `Nav.astro`) com o texto de `cliente.json.avisoDemo`; quando `false`, nada é renderizado.

**Critérios de aceite:**
- `C4.1` cada componente usa **apenas** tokens/componentes de C2 — nenhuma cor, fonte ou espaçamento solto (hardcoded). **Verificar:** `grep` por hex de cor / `px` de fonte fora de `global.css` retorna vazio (rastreabilidade item 23).
- `C4.2` nenhum dado do cliente hardcoded; tudo vem de `cliente.json`. **Verificar:** `grep` por strings do brief (ex.: "Vila Mariana", "CRMV") nos componentes retorna vazio (rastreabilidade item 11).
- `C4.3` existe **um único `<h1>`** (no `Hero`) e a hierarquia de headings é correta. **Verificar:** `grep -c "<h1"` no HTML gerado = 1 (rastreabilidade itens 1, 13).
- `C4.4` todo texto está no HTML gerado no build (nada injetado por JS). **Verificar:** inspeção do `dist/` (rastreabilidade item 13).
- `C4.5` links de WhatsApp usam só `wa.me` (via `montarLinkWhatsapp`) com mensagem de `cliente.json`; a faixa/CTA de emergência usa `tel:` como ação principal. **Verificar:** inspeção + `grep` não acha `api.whatsapp.com` (rastreabilidade itens 8, 11).
- `C4.6` flags respeitadas: `temPlano:false` ocultaria o bloco Plano; `emergencia24h:false` ocultaria faixa e etiqueta; pontos marcados com comentário onde o template dobra. **Verificar:** inspeção do código condicional (rastreabilidade item 12).
- `C4.7` pontos de foto usam `PlaceholderFoto` e emitem comentário `<!-- FOTO: ... -->`. **Verificar:** `grep "FOTO:"` (rastreabilidade item 11).
- `C4.8` "Como chegar" (Localizacao) é link do Google Maps por endereço, **sem API key**. **Verificar:** inspeção do href (rastreabilidade item 9).
- `C4.9` a 360px, acima da dobra aparecem quem é a clínica, emergência 24h e um CTA sem rolar; menu hambúrguer acessível; foco visível. **Verificar:** teste manual + inspeção (rastreabilidade itens 1, 22).
- `C4.10` `npx astro check` passa. **Verificar:** saída do comando.
- `C4.11` `Hero.astro` expõe `<slot name="interativo" />` à direita da headline. **Verificar:** inspeção do componente.
- `C4.12` com `demoSite:true`, o aviso de **site demonstrativo** (texto de `cliente.json.avisoDemo`, no Footer ou faixa fina no topo) aparece **visível** na página; com `demoSite:false`, **não** aparece. **Verificar:** inspeção do HTML gerado alternando a flag.

**Fora do escopo:** implementar as ferramentas interativas (C5), a função da API (C6), as tags de SEO (C7). C4 **não** edita `cliente.json` (pedido de mudança de contrato se faltar campo), nem `Base.astro`/`index.astro`.

---

### C5 — Ferramentas interativas
**Agente:** `camada-interativos` · **Onda:** 4 (paralela a C7) · **Depende de:** C3, C4

**Objetivo:** implementar o front das ferramentas com JS leve e isolado, incluindo o widget de carteirinha em modo demonstração (mock), consumindo o contrato de resposta da API.

**Pode criar/editar (posse exclusiva):**
`src/components/CarteirinhaWidget.astro`, `src/components/Agendamento.astro`, `src/components/pacote-captacao/CalculadoraIdade.astro`, `ChecklistEmergencia.astro`, `PodeComer.astro`.

**Pode apenas ler:** `docs/design-system.md`, `global.css`, `src/components/base/` (C2); `cliente.json` e `alimentos.json` (C3); `src/lib/links.ts` e `src/lib/escape.ts` (C1); o contrato de resposta da API (§5-C6).

**Depende de / consome:**
- Tokens/componentes de C2.
- `cliente.json` (C3): `whatsapp`, `demoMode`, `servicosAgendamento`, `mensagensWhatsapp`, `flags`, `carteirinha`; `alimentos.json` (C3) para o "pode comer".
- `montarLinkWhatsapp`, `montarLinkTelefone`, `preencherMensagem` (C1) e `escaparHtml` (C1).
- **Contrato de resposta da API** (§5-C6) — o front consome exatamente os `status`: `ok`, `ilegivel`, `limite`, `nao_configurado`, `erro`.

**Entrega / contratos que expõe:**
- Componentes exportados: `CarteirinhaWidget`, `Agendamento`, `CalculadoraIdade`, `ChecklistEmergencia`, `PodeComer` — inseridos por C8: o `CarteirinhaWidget` no `<slot name="interativo" />` do `Hero` (C4) e os demais compostos diretamente no `index.astro`.
- Função interna `analisarCarteirinha(file)` com o comportamento do PRD 7 / plano §6.1.

**Critérios de aceite (reprovação automática se violar as regras de segurança):**
- `C5.1` **Carteirinha nunca** exibe "está tudo em dia" / "seu pet está protegido" como conclusão geral; status é **por vacina** + disclaimer do PRD 7 + botão grande de WhatsApp. **Verificar:** inspeção + teste manual + `grep` por "tudo em dia"/"está protegido" retorna vazio (rastreabilidade itens 2, 3).
- `C5.2` checkbox de consentimento LGPD obrigatório; botão "Analisar" só habilita com **imagem + consentimento**. **Verificar:** teste manual + inspeção (rastreabilidade item 2).
- `C5.3` imagem reduzida no navegador antes do envio (lado maior ~1600px, JPEG, alvo ≤1,5 MB). **Verificar:** inspeção do código de redução (rastreabilidade itens 2, 11-upload).
- `C5.4` estados completos: upload, prévia, "Lendo a carteirinha…", resultado, **ilegível**, **limite**, **erro** — todos com saída para WhatsApp, nenhum trava. **Verificar:** teste manual de cada estado (rastreabilidade item 2).
- `C5.5` **mock aparece só** em `demoMode:true` **ou** resposta `nao_configurado`, com selo "Modo demonstração" e **datas relativas** à data atual; **nunca** em `limite`/`erro`/`ilegivel`. **Verificar:** teste manual + inspeção da lógica (rastreabilidade item 4).
- `C5.6` texto vindo da API é **escapado** (`escaparHtml`) antes de inserir; nada de `innerHTML` com dados da API. **Verificar:** inspeção (rastreabilidade item 7).
- `C5.7` nenhuma chave de API no front; chamada é `POST /api/carteirinha` só quando `demoMode:false`. **Verificar:** `grep` por `ANTHROPIC`/chaves no bundle retorna vazio (rastreabilidade item 21).
- `C5.8` **Agendamento** gera link `wa.me` (via `montarLinkWhatsapp`) com a mensagem no formato do PRD; campo data com `min` = hoje e formato **dd/mm/aaaa** na mensagem; sem backend/API oficial. **Verificar:** teste manual (link abre WhatsApp preenchido) (rastreabilidade item 8).
- `C5.9` **Checklist de emergência:** ação principal "Ligar agora" (`tel:`), WhatsApp secundário; nunca diz "pode esperar". **Verificar:** inspeção + teste manual (rastreabilidade item 10).
- `C5.10` **"Pode comer isso?"** usa só `alimentos.json`; item fora da lista → orienta perguntar à clínica; **nenhuma** geração livre de IA. **Verificar:** inspeção (sem chamada de IA) + teste manual (rastreabilidade item 10).
- `C5.11` **Calculadora de idade** usa tabela por espécie/porte de C3, **nunca ×7**. **Verificar:** inspeção + teste manual (rastreabilidade item 10).
- `C5.12` **Proibido** `localStorage`/`sessionStorage`; estado em memória. **Verificar:** `grep` retorna vazio (rastreabilidade item 11).
- `C5.13` acessibilidade: `aria-live` nas mudanças de estado, labels, teclado, foco visível, `prefers-reduced-motion`; upload com `accept="image/*" capture="environment"`. **Verificar:** inspeção + teste (rastreabilidade item 22).
- `C5.14` `npx astro check` passa. **Verificar:** saída.

**Fora do escopo:** a função `/api/carteirinha` (C6), as seções estáticas (C4), SEO (C7). C5 **não** edita `cliente.json` nem os arquivos de C4.

---

### C6 — API da carteirinha
**Agente:** `camada-api-carteirinha` · **Onda:** 5 · **Depende de:** C1, C3, C5

**Objetivo:** implementar a única função serverless do site — "a IA lê, a regra decide" — com proteções de custo/abuso, testes que simulam a IA e o guia de deploy.

**Pode criar/editar (posse exclusiva):**
`src/pages/api/carteirinha.ts`, `src/lib/carteirinha/` (ex.: `validar-entrada.ts`, `ia.ts`, `schema.ts`, `classificar.ts`, `ratelimit.ts`, `tipos.ts`), os testes `src/lib/carteirinha/**/*.test.ts`, `docs/deploy.md`.

**Pode apenas ler:** `cliente.json` e `regras-vacinas.json` (C3); `src/lib/escape.ts` (C1); `astro.config.mjs`, `package.json`, `vitest.config.ts`, `.env.example` (C1).

**Depende de / consome:**
- `regras-vacinas.json` (C3) para a classificação determinística; `carteirinha.limites` (C3) para os limites.
- `escaparHtml` (C1); dependências instaladas por C1 (`@anthropic-ai/sdk`, `@upstash/ratelimit`, `@upstash/redis`, `zod`, `vitest`); adapter `@astrojs/vercel` (C1) para `prerender = false`.
- Variáveis de ambiente pelos nomes do `.env.example` (C1): `ANTHROPIC_API_KEY`, `MODELO_IA`, `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`.

**Entrega / contratos que expõe (consumido por C5):**
- Rota `POST /api/carteirinha` com `export const prerender = false`.
- **Formato de resposta** (idêntico ao que o mock de C5 imita), PRD 7:
  - `{ status:"ok", legivel:true, especie, vacinas:[{nome, ultima, status:"ok"|"pendente"}], observacao }`
  - `{ status:"ilegivel", legivel:false, motivo }`
  - `{ status:"limite" }` (HTTP 429)
  - `{ status:"nao_configurado" }`
  - `{ status:"erro" }`
  - Nenhuma resposta expõe detalhes internos.

**Critérios de aceite (reprovação automática se violar):**
- `C6.1` "IA lê, regra decide": o prompt pede **só** JSON `{legivel, especie, vacinas:[{nome_lido, data_ultima_dose}]}`, temperatura 0, `max_tokens` baixo, timeout ~20s; a classificação `ok/pendente` é feita pelo código com `regras-vacinas.json` + data atual; não reconhecida/sem data/data ilegível → `pendente`. **Verificar:** inspeção + testes Vitest de classificação (rastreabilidade item 3).
- `C6.2` saída da IA validada com `zod`; qualquer desvio → `ilegivel`; o prompt instrui a IA a ignorar instruções escritas na imagem; todo texto da IA é escapado. **Verificar:** testes Vitest (JSON inválido → `ilegivel`) + inspeção (rastreabilidade itens 2, 7).
- `C6.3` **falha fechada:** sem `ANTHROPIC_API_KEY` **ou** sem Upstash → `{status:"nao_configurado"}` **sem chamar a IA**. **Verificar:** teste Vitest (rastreabilidade item 19).
- `C6.4` **limite de uso** com `@upstash/ratelimit`: por IP (5/h, 10/dia) e teto global (200/dia), lidos de `carteirinha.limites`; IP como chave **com hash** e expiração; excedeu → `{status:"limite"}` HTTP 429. **Verificar:** testes Vitest simulando contador (6ª análise → `limite`) (rastreabilidade item 20).
- `C6.5` **validação de entrada:** só `POST`; `image/jpeg|png|webp`; máx. 4 MB; consentimento obrigatório; **`Origin` = host da própria requisição** (localhost aceito em dev), **não** comparado ao campo `site`. **Verificar:** testes Vitest (método/mime/tamanho/sem consentimento/Origin de outra origem rejeitados) (rastreabilidade itens 5, 6).
- `C6.6` **chave só em env** (nunca em código/log/resposta/erro); **imagem nunca armazenada nem logada**; processada em memória. **Verificar:** inspeção + `grep` por segredos + revisão de logs (rastreabilidade itens 7, 21).
- `C6.7` respostas **exatamente** no formato do PRD 7; erros genéricos. **Verificar:** testes Vitest dos 5 estados (rastreabilidade item 2).
- `C6.8` **nunca** conclusão geral "tudo em dia"; só status por vacina. **Verificar:** inspeção (rastreabilidade item 3).
- `C6.9` testes Vitest cobrem: legível (em dia + a verificar), ilegível, JSON inválido, vacina desconhecida, data ausente, limite por IP, teto global, falta de chave, falta de limitador, arquivo grande, tipo inválido, sem consentimento, Origin de outra origem. **`npm run test` passa.** **Verificar:** saída do comando (rastreabilidade itens 2, 18, 19, 20).
- `C6.10` `docs/deploy.md` explica em linguagem simples: criar chave Anthropic + **limite de gasto mensal**, conectar Upstash pelo Marketplace da Vercel, cadastrar variáveis (local e Vercel), opcional Firewall da Vercel, como testar após deploy. **Verificar:** inspeção do documento (rastreabilidade item 24).
- `C6.11` nenhum outro endpoint/função/banco criado além desta rota; Upstash usado **só** para contadores. **Verificar:** inspeção da árvore (rastreabilidade itens 6, 11).

**Fora do escopo:** front do widget (C5), qualquer seção/estilo. C6 **não** edita `cliente.json`, `astro.config.mjs`, `package.json` (pedido de mudança de contrato a C1/C3 se precisar).

---

### C7 — SEO técnico
**Agente:** `camada-seo` · **Onda:** 4 (paralela a C5) · **Depende de:** C3, C4

**Objetivo:** implementar todo o SEO técnico — o diferencial de venda — a partir de `cliente.json`, sem tocar em `Base.astro` (expõe um componente que C8 renderiza).

**Pode criar/editar (posse exclusiva):**
`src/components/Seo.astro`, `src/pages/robots.txt.ts`, `src/pages/sitemap.xml.ts`.

**Pode apenas ler:** `cliente.json` (C3); `public/favicon.svg`, `favicon.png`, `og.svg`/`og.png` (C2/C1); a copy/headings de C4 (para coerência do JSON-LD).

**Depende de / consome:**
- `cliente.json` (C3): `nome`, `site`, `endereco`, `telefone`, `horarios`, `provaSocial`, `demoSite`.
- Ativos `favicon.*` e `og.png` (URL absoluta baseada em `site`).

**Entrega / contratos que expõe:**
- **`src/components/Seo.astro`** — componente para o `<head>`, sem props obrigatórias (lê `cliente.json`), que renderiza: `<title>` e `<meta name="description">` reais e locais; Open Graph (título, descrição, imagem `/og.png` absoluta, `locale=pt_BR`); `<link rel="canonical">` com `site`; **JSON-LD `VeterinaryCare`** (nome, endereço, telefone, horário incluindo 24h, URL); `favicon` (SVG + PNG fallback); quando `demoSite:true`, `<meta name="robots" content="noindex,nofollow">`. **O aviso visível de site demonstrativo é responsabilidade de C4 (no corpo da página), não de `Seo.astro`** — este componente cuida apenas das tags do `<head>`. C8 insere `<Seo />` no `<head>` de `Base.astro`.
- **`robots.txt.ts`** — rota pré-renderizada gerada de `cliente.json`; **nunca bloqueia rastreamento**; referencia o sitemap **só** quando `demoSite:false`.
- **`sitemap.xml.ts`** — rota pré-renderizada; gera sitemap **só** quando `demoSite:false` (quando `true`, não gera).

**Critérios de aceite:**
- `C7.1` `<title>`, `<meta description>`, OG (com imagem, `locale=pt_BR`), canonical presentes e vindos de `cliente.json`. **Verificar:** inspeção do HTML gerado (rastreabilidade item 13).
- `C7.2` JSON-LD `VeterinaryCare` sintaticamente válido e coerente com os dados visíveis (nome, endereço, telefone, horário 24h, URL). **Verificar:** validador de JSON-LD + comparação com a página (rastreabilidade item 13).
- `C7.3` com `demoSite:true`: `<meta robots noindex,nofollow>` presente, **sitemap não gerado**, robots.txt **não** referencia sitemap e **não** bloqueia rastreamento. (O aviso **visível** de site demonstrativo fica em C4 — ver C4.12.) **Verificar:** inspeção do `<head>`, do `robots.txt` gerado e ausência de `sitemap.xml` (rastreabilidade item 14).
- `C7.4` com `demoSite:false`: sem `noindex`, sitemap gerado e referenciado no robots.txt. **Verificar:** alternar a flag e inspecionar (rastreabilidade item 14).
- `C7.5` OG usa `/og.png` com URL **absoluta** baseada em `site`. **Verificar:** inspeção da meta tag (rastreabilidade item 15).
- `C7.6` favicon SVG + PNG referenciados. **Verificar:** inspeção do `<head>`.
- `C7.7` `npx astro check` passa. **Verificar:** saída.

**Fora do escopo:** gerar `og.png` (é C1), criar os ativos de marca (C2), editar `Base.astro`/`index.astro` (C1), conteúdo/dados (C3), o aviso **visível** de site demonstrativo (é C4). C7 **não** cria o `<h1>` (é de C4).

---

### C8 — Integração final e verificação
**Agente:** `camada-fundacao` · **Onda:** 6 · **Depende de:** C1–C7

**Objetivo:** compor tudo em `index.astro`/`Base.astro`, rodar build e testes, executar o checklist de aceite e as verificações transversais.

**Pode criar/editar (posse exclusiva):** `src/layouts/Base.astro` e `src/pages/index.astro` (mesmos arquivos de C1 — finalização).

**Pode apenas ler:** todos os componentes de C2, C4, C5, C7; `cliente.json` (C3); relatórios das camadas.

**Depende de / consome:** os componentes exportados por C4 (`Nav`, `Hero`, `Servicos`, `PlanoVetSaude`, `Equipe`, `Localizacao`, `Depoimentos`, `Footer`, `BotaoWhatsappFlutuante`), C5 (`CarteirinhaWidget`, `Agendamento`, `CalculadoraIdade`, `ChecklistEmergencia`, `PodeComer`), C7 (`Seo`), C2 (`global.css`) e a rota de C6.

**Entrega / contratos que expõe:**
- `Base.astro` finalizado: importa `global.css`, renderiza `<Seo />` no `<head>`.
- `index.astro` compondo todas as seções **na ordem do PRD 5**, com as âncoras corretas. O `CarteirinhaWidget` (C5) é injetado no slot do herói via `<Hero><CarteirinhaWidget slot="interativo" /></Hero>`; o `Agendamento` e as ferramentas do Pacote Captação (C5) são compostos **diretamente no `index.astro`**, na ordem do PRD 5.

**Critérios de aceite:**
- `C8.1` `npm run build` passa **sem erros**; todas as páginas estáticas; única rota dinâmica `/api/carteirinha`. **Verificar:** saída + inspeção do `dist/` (rastreabilidade item 16).
- `C8.2` `npm run test` (Vitest) passa **sem falhas**. **Verificar:** saída.
- `C8.3` ordem das seções e âncoras de navegação conferem com o PRD 5; a nav rola para os ids corretos. **Verificar:** inspeção + teste manual (rastreabilidade item 1).
- `C8.4` **verificação transversal — segredos:** `grep` por `ANTHROPIC_API_KEY`, tokens Upstash e chaves em todo o repo/`dist` retorna só referências a `process.env`/`.env.example`. **Verificar:** `grep` (rastreabilidade item 21).
- `C8.5` **verificação transversal — armazenamento local:** `grep` por `localStorage`/`sessionStorage` em todo o `src/` retorna vazio. **Verificar:** `grep` (rastreabilidade item 11).
- `C8.6` **verificação transversal — WhatsApp/telefone:** nenhum uso de `api.whatsapp.com`; emergências usam `tel:`. **Verificar:** `grep` (rastreabilidade itens 8, 10, 11).
- `C8.7` checklist do PRD §15 revisado item a item (com apoio dos relatórios das camadas). **Verificar:** tabela de checklist no relatório de C8.
- `C8.8` responsivo a 360px, foco de teclado visível, `prefers-reduced-motion` respeitado no site montado. **Verificar:** teste manual (rastreabilidade item 22).

**Fora do escopo:** reescrever componentes de outras camadas (se algum estiver errado, é ciclo de correção do supervisor com a camada dona). C8 **não** edita `cliente.json`, componentes de seção/interativos, `Seo.astro` etc.

---

## 6. Checagem de cobertura (matriz de rastreabilidade do plano → camada)

Cada linha da matriz do `03-plano.md` (§12) e da ordem de construção (§11) tem **exatamente uma** camada responsável (a que implementa/produz). "Verif." indica onde é conferido.

| # | Item da matriz do plano (§12) | Camada responsável | Verif. |
|---|-------------------------------|--------------------|--------|
| 1 | UX §4 (5s, CTA no herói, emergência visível, confiança, sem pop-up, mobile-first) | **C4** (base visual de C2) | C4.9, C8.3 |
| 2 | Carteirinha — fluxo, estados, segurança (front) | **C5** | C5.1–C5.6 |
| 2b | Carteirinha — função/estados (backend) | **C6** | C6.7, C6.9 |
| 3 | "IA lê, regra decide"; nunca "tudo em dia" | **C6** | C6.1, C6.8 |
| 4 | Mock só em `demo`/`nao_configurado`, nunca em `limite`/`erro`/`ilegivel` | **C5** | C5.5 |
| 5 | Verificação de `Origin` por mesma origem (não pelo `site`) | **C6** | C6.5 |
| 6 | API única + proteções de custo/abuso | **C6** | C6.4, C6.5, C6.11 |
| 7 | Chave só em env; imagem nunca armazenada; texto escapado | **C6** (escape util de C1) | C6.6, C6.2 |
| 8 | Agendamento → `wa.me`; data não-passada; dd/mm/aaaa | **C5** | C5.8 |
| 9 | Demais seções com conteúdo real | **C4** (copy de C3) | C4.2, C3.5 |
| 10 | Pacote Captação (checklist "Ligar agora"; pode-comer lista fixa; idade sem ×7) | **C5** (dados de C3) | C5.9–C5.11 |
| 11a | Estático + 1 função; sem DB (só Upstash contadores) | **C6** (config de C1) | C6.11, C1.2 |
| 11b | Sem `localStorage`/`sessionStorage` | **C8** (grep transversal) | C8.5 |
| 11c | WhatsApp só `wa.me`; emergência `tel:` | **C4** (nav/local) e **C5** (ferramentas); verif. transversal | C4.5, C5.9, C8.6 |
| 11d | Dados isolados em `cliente.json`; nada hardcoded | **C3** (dados) + **C4/C5** (consumo) | C3.1, C4.2 |
| 11e | Marcações `FOTO` / `VALIDAR-VET` | `FOTO` → **C4**; `VALIDAR-VET` → **C3** | C4.7, C3.6 |
| 12 | Dimensões variáveis como flags, sem duplicação | **C3** (flags) + marcação no código de C4/C5 | C3.4, C4.6 |
| 13 | SEO — title/meta/OG/canonical/JSON-LD | **C7** | C7.1, C7.2 |
| 13b | Um `<h1>`, HTML semântico, texto no build | **C4** | C4.3, C4.4 |
| 14 | SEO demo — `noindex` (C7); robots não bloqueia; sitemap só se `demoSite:false` (C7); **aviso visível de site demonstrativo** (C4) | **C7** (tags do `<head>`) + **C4** (aviso visível) | C7.3, C7.4, C4.12 |
| 15 | OG PNG 1200×630 gerado por `npm run og` (sharp, **fora do build**) e commitado, sem serviço externo | **C1** (script `scripts/gerar-og.mjs`, consome `og.svg` de C2) | C1.8, C7.5 |
| 16 | DoD build sem erros / única rota dinâmica | **C8** (esqueleto por C1) | C8.1, C1.1 |
| 17 | DoD demoMode fluxo completo com selo | **C5** | C5.5 |
| 18 | DoD IA real — legível/ilegível, nunca "tudo em dia" | **C6** | C6.1, C6.9 |
| 19 | DoD falha fechada sem chave/limitador | **C6** | C6.3 |
| 20 | DoD limite (6ª → "limite") + teto global | **C6** (limites de C3) | C6.4, C3.3 |
| 21 | DoD sem segredos no código/logs/repo | **C6** (origem) + **C8** (grep transversal) | C6.6, C8.4 |
| 22 | DoD responsivo 360px, foco, reduced-motion | **C2** (diretrizes) + **C4/C5** (implementação) | C2.7, C4.9, C5.13, C8.8 |
| 23 | DoD segue design-system aprovado | **C2** (produz) + supervisor (verifica todas as visuais) | C2.1, C4.1 |
| 24 | DoD `docs/deploy.md` passo a passo | **C6** | C6.10 |
| — | Ordem de construção §11 passo 1 (setup) | **C1** | C1.1–C1.8 |
| — | §11 passo 2 (design system + global.css + og.svg) | **C2** | C2.1–C2.7 |
| — | §11 passo 3 (dados/conteúdo) | **C3** | C3.1–C3.9 |
| — | §11 passo 4 (seções estáticas) | **C4** | C4.1–C4.12 |
| — | §11 passo 5 (interativos) | **C5** | C5.1–C5.14 |
| — | §11 passo 6 (API carteirinha) | **C6** | C6.1–C6.11 |
| — | §11 passo 7 (SEO técnico) | **C7** | C7.1–C7.7 |
| — | §11 passo 8 (integração final) | **C8** | C8.1–C8.8 |

**Resultado:** todos os itens da matriz e todos os passos da ordem de construção têm dono. Nenhum item ficou sem camada. Itens transversais (11b, 11c, 21) têm um dono primário para verificação (C8 via grep), enquanto as camadas produtoras cumprem a regra na origem. O item 14 tem duas frentes com donos distintos e sem sobreposição de arquivos: C7 cuida das tags do `<head>` (`Seo.astro`/`robots`/`sitemap`) e C4 cuida do aviso visível no corpo da página.

---

## 7. Regra de mudança de contrato

Se uma camada precisar de algo que **não está no seu contrato** (ex.: um campo novo em `cliente.json`, um token CSS que não existe, uma dependência não instalada, uma variável de ambiente nova), ela **para imediatamente** e **reporta ao orquestrador** como "pedido de mudança de contrato", indicando: o que falta, por quê e o arquivo dono. Quem faz a alteração é **a camada dona do arquivo** (ver §3), nunca a solicitante. Exemplos:
- Falta um campo em `cliente.json` → C3 altera; a solicitante espera.
- Falta um nome de ícone em `Icone.astro` → C2 altera.
- Precisa de uma variável de ambiente nova → C1 altera `.env.example` (e C6 usa).
- Precisa de uma dependência nova → C1 altera `package.json`.
Nenhuma camada edita arquivo fora da sua posse para "resolver rápido". Isso preserva a posse exclusiva e evita conflitos de integração.

---

## 8. Riscos de integração e mitigação

| Risco | Mitigação |
|-------|-----------|
| `Base.astro`/`index.astro` são "gargalos" de integração (esqueleto em C1, finalização em C8) | Posse única de C1; `Hero.astro` (C4) expõe `<slot name="interativo" />` e os interativos de C5 são compostos por C8 no `index.astro`; C8 só compõe, não reescreve componentes. |
| C4 é dono do `Hero.astro` mas o `CarteirinhaWidget` é de C5 (roda depois) — ninguém conseguiria inseri-lo se C4 apenas deixasse um comentário | `Hero.astro` expõe `<slot name="interativo" />` à direita da headline; C8 injeta `<CarteirinhaWidget slot="interativo" />` pelo `index.astro`; o `Agendamento` e as ferramentas do Pacote Captação também são compostos por C8 direto no `index.astro`, sem editar arquivos de C4. |
| Contrato de resposta da API divergir entre o mock (C5) e a função (C6) | O formato do PRD 7 está fixado em §5-C6 e é a fonte única; C5 imita exatamente os mesmos `status`; testes de C6 cobrem cada estado. |
| Nomes de tokens/componentes de C2 não baterem com o que C4/C5 esperam | Contrato de nomes obrigatórios em §5-C2; C4/C5 só usam esses nomes; divergência vira pedido de mudança de contrato. |
| Ícones referenciados em `cliente.json` (`servicos[].icone`) sem correspondente em `Icone.astro` — C2 e C3 rodam em paralelo | A **lista mínima obrigatória de nomes de ícones** está fixada agora no contrato de C2 (§5-C2); C3 usa **somente** nomes dessa lista; se faltar, pedido de mudança a C2. |
| C2 e C3 rodam em paralelo e ambas mexem em conteúdo de `servicos` (ícone vs. texto) | Posse separada: C3 dona de `cliente.json`; C2 dona de `Icone.astro`; o único acoplamento é o nome do ícone — resolvido pela lista fixa na Onda 2. |
| Fontes ausentes no servidor de build da Vercel quebrariam o texto do `og.png` | `og.svg` tem o texto **convertido em caminhos/paths** (C2) e o `og.png` é gerado **localmente** por `npm run og` e **commitado** (C1); o build da Vercel não gera a imagem nem depende de fontes. |
| `og.png` não existir quando C7 referencia | `og.png` é gerado por `npm run og` (C1) e **commitado** no repositório; C7 só referencia a URL. Regenerar com `npm run og` quando a marca mudar. |
| Aviso de "site demonstrativo" ficar invisível se colocado só no `<head>` | O aviso **visível** é responsabilidade de C4 (Footer/faixa no topo), controlado por `demoSite` e com texto de `avisoDemo` (C3); `Seo.astro` (C7) cuida apenas das tags do `<head>` (noindex etc.). |
| Portão de design atrasar as ondas visuais | C3 (dados) roda em paralelo e não depende do portão; adianta trabalho enquanto se aguarda a aprovação. |
| Vazamento de segredo passar despercebido | C6 garante na origem (C6.6) + C8 faz grep transversal final (C8.4). |
| `localStorage` acidental em C4 (nav) ou C5 | Proibição explícita (C5.12) + grep transversal em C8 (C8.5). |

---

## 9. Histórico de revisões

| Data | Autor | Mudança |
|------|-------|---------|
| 2026-09-29 | Subagente arquiteto de camadas | Criação de `04-camadas.md` (RASCUNHO) a partir do plano aprovado: 8 camadas, 6 ondas, 2 portões (aprovação do design-system e aprovação final), posse exclusiva de arquivos, contratos entre camadas, critérios de aceite verificáveis e matriz de cobertura completa. |
| 2026-09-30 | Subagente arquiteto de camadas | Aplicadas 4 correções do Vinicius (status segue RASCUNHO): **(1) Slot do Hero** — `Hero.astro` (C4) expõe `<slot name="interativo" />` e C8 injeta o `CarteirinhaWidget` e demais interativos de C5 pelo `index.astro`, resolvendo o conflito de posse (atualizados §5-C4 contrato/entrega, novo C4.11, §5-C5 entrega, §5-C8 entrega, §8 riscos). **(2) Nomes de ícones** — fixada no contrato de C2 (§5-C2) a lista mínima obrigatória de 21 ícones de `Icone.astro`, já que C2 e C3 rodam em paralelo; atualizados C2.3, C3 contrato e C3.1, §8 riscos. **(3) og.png** — deixa de ser gerado no build: `og.svg` (C2) com texto em caminhos/paths, novo `scripts/gerar-og.mjs` + `npm run og` (C1), `og.png` commitado (removido do `.gitignore`); atualizados §3 mapa, §5-C1, C1.3, C1.8, §5-C2 e C2.6, §6 item 15, §8 riscos. **(4) Aviso visível de site demonstrativo** — passa a ser responsabilidade de C4 (Footer/faixa) com texto de `cliente.json.avisoDemo` (novo campo em C3); `Seo.astro` (C7) fica só com as tags do `<head>`; atualizados §5-C3 contrato + novo C3.9, §5-C4 entrega/consumo + novo C4.12, §5-C7 e C7.3, §6 item 14. |
