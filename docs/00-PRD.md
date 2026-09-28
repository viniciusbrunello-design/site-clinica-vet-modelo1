# PRD — Site 1: Clínica Veterinária (arquétipo clínico)

> Documento para o Claude Code construir o **Site 1** de referência. É um site single page, construído em **Astro** e publicado na **Vercel via GitHub**, com uma única função de backend para a leitura da carteirinha por IA. Depois vira template. Construa fiel a este documento; onde faltar detalhe, siga o `docs/design-system.md` (após aprovado) e a seção "Requisitos Técnicos" em vez de improvisar.

---

## 1. Contexto e objetivo

Este é o primeiro de dois sites de referência para clínicas veterinárias. Depois de pronto e refinado, ele vira um template padronizado que será clonado e personalizado por cliente. Portanto: **construa como um site único e excelente para o cliente descrito abaixo**, mas mantenha os dados do cliente isolados (ver "Requisitos Técnicos"), para facilitar a extração de template mais tarde.

O site tem **um único objetivo de negócio**: transformar visitante em **lead no WhatsApp** da clínica (agendamento ou dúvida). Todo o resto é suporte a isso.

Este site representa o **arquétipo clínico** (medicina veterinária séria: consultas, vacinas, cirurgia, emergência 24h). Ele NÃO tem galeria de antes/depois de banho e tosa — isso é do Site 2 (arquétipo estética). Manter essa fronteira limpa é proposital.

---

## 2. Público

Duas pessoas usam este site, e o design serve às duas:

- **Tutor (visitante):** chegou de uma busca tipo "veterinário 24h perto de mim", ansioso, no celular, com outras abas abertas. Decide em ~5 segundos se a clínica parece confiável. Quer saber rápido: é sério? é perto? atende agora? como falo com eles?
- **Dono da clínica (comprador do meu serviço):** precisa olhar o site e pensar "isso me traz cliente". Os dois heróis (leitor de carteirinha + agendamento) existem para impressionar essa pessoa.

---

## 3. Brief do cliente (fictício, mas tratar como real)

Construa com estes dados concretos — não use "lorem ipsum" nem placeholders vagos.

- **Nome:** Clínica VetSaúde
- **Cidade/bairro:** Vila Mariana, São Paulo — SP (bairro urbano, concorrido)
- **Espécies atendidas:** cães e gatos
- **Posicionamento/tom:** medicina veterinária séria e acolhedora. Mensagem central: "cuidamos do seu pet como cuidaríamos do nosso". Transmitir competência técnica + carinho, sem infantilizar.
- **Emergência:** sim, atende **24 horas**.
- **Serviços:** consultas clínicas, vacinação, exames laboratoriais e de imagem, cirurgia, internação, odontologia veterinária, emergência 24h.
- **Plano de saúde pet:** "Plano VetSaúde" — mensalidade que inclui consultas de rotina, vacinas do calendário e descontos em exames.
- **Equipe (nome + especialidade + CRMV):**
  - Dra. Ana Ribeiro — Clínica geral / Responsável técnica — CRMV-SP 00000
  - Dr. Lucas Martins — Cirurgia — CRMV-SP 00000
  - Dra. Carla Souza — Dermatologia — CRMV-SP 00000
  - Dr. Pedro Nakamura — Medicina felina — CRMV-SP 00000
- **Horário:** Seg–Sex 8h–20h · Sáb 8h–16h · Dom e feriados: emergências 24h
- **Contato:** WhatsApp/telefone (11) 99999-9999 *(fictício — na demonstração, os links de WhatsApp usam o número do Vinicius definido no briefing)* · contato@vetsaude.com.br · Rua Exemplo, 123 — Vila Mariana, São Paulo/SP
- **Prova social:** 4,9★ no Google, +10 anos de bairro, +8.000 tutores atendidos.

---

## 4. Princípios de UX (não negociáveis — vêm de pesquisa de conversão vet 2026)

1. **Acima da dobra responde "quem é essa clínica e o que eu faço agora"** em menos de 5 segundos. Tutor abandona se não achar o que quer nesse tempo.
2. **CTA de agendamento no herói, acima da dobra, alcançável sem rolar no celular.** Enterrar o agendamento é a falha de conversão nº 1 em sites vet. O botão de contato fica também fixo no topo (nav).
3. **Emergência 24h visível de imediato** (tutor em pânico é tráfego de altíssima intenção).
4. **Confiança na hora:** foto real de equipe/estrutura, endereço visível, telefone claro, CRMV à mostra. (No build use placeholders de imagem marcados — ver seção 11.)
5. **Sem pop-up.** Se precisar chamar atenção, use banner/faixa.
6. **Mobile-first.** A maioria chega pelo celular, em situação de estresse.

---

## 5. Arquitetura de informação (ordem das seções)

Single page com navegação por âncoras. Ordem:

1. **Nav fixa** (logo, links de âncora, botão WhatsApp, faixa de emergência)
2. **Herói** — headline + emergência 24h + CTAs + **Herói 1: leitor de carteirinha**
3. **Herói 2: Agendamento em 30s no WhatsApp**
4. **Serviços**
5. **Plano VetSaúde** (assinatura)
6. **Equipe (com CRMV)**
7. **[Pacote Captação]** — calculadora de idade + checklist de emergência + "meu pet pode comer isso" (ver seção 10; marcar como camada de pacote)
8. **Localização + horários + emergência**
9. **Depoimentos**
10. **Footer + LGPD + botão flutuante de WhatsApp**

---

## 6. Design

O design todo ficará a critério da IA que irá construir o projeto.

**Regra de processo:** antes de construir qualquer seção, a camada de design deve registrar suas decisões em `docs/design-system.md` (paleta, tipografia, espaçamentos, componentes-base, estilo de ícones e placeholders de foto, e uma lista do que evitar — "tells" de site genérico feito por IA). Esse arquivo precisa ser **aprovado pelo Vinicius** e passa a ser a referência visual obrigatória para todas as outras camadas e para o supervisor.

---

## 7. Herói 1 — Leitor de carteirinha de vacinação (feature de destaque)

**O que é:** o tutor envia a foto da carteirinha; uma **IA de visão real** lê nomes e datas das vacinas, um conjunto de **regras fixas** classifica cada vacina e o site devolve uma **triagem**, sempre encaminhando para o WhatsApp da clínica. É o gancho que diferencia o site. Fica no herói, à direita da headline.

**Faixa-âncora (banner) acima do widget** — usar estatística REAL, nunca número inventado:

> "Cinomose e parvovirose matam até 90% dos cães não vacinados. O seu pet está protegido? Envie a foto da carteirinha e descubra em segundos se há vacinas em atraso." *(fonte: letalidade de cinomose 50–90% em cães não vacinados; parvovirose de alta mortalidade em filhotes)*

**Fluxo e estados (front-end):**

1. Área de upload (arrastar ou tocar; no celular abrir câmera: `accept="image/*" capture="environment"`).
2. Prévia da imagem carregada. A imagem é **reduzida no navegador** antes do envio (lado maior ~1600px, JPEG, alvo ≤ 1,5 MB) — a Vercel recusa requisições acima de ~4,5 MB.
3. **Checkbox de consentimento obrigatório** (LGPD) — botão "Analisar" só habilita com imagem + consentimento marcados. O texto do consentimento informa que a foto é enviada a um serviço de IA apenas para leitura e não é armazenada.
4. Estado de carregando (spinner + "Lendo a carteirinha…").
5. **Resultado:** lista de vacinas lidas, cada uma com status **por vacina** ("em dia" / "verificar"), + disclaimer, + botão grande de WhatsApp com as vacinas a verificar já escritas na mensagem (ou a mensagem "sem pendências" do briefing, se nenhuma estiver a verificar).
6. **Ilegível:** se a IA não conseguir identificar vacinas e datas (foto borrada, escura, cortada, letra ilegível, ou a imagem não é uma carteirinha), mostra: "Não consegui identificar as vacinas e datas. Tente uma foto mais nítida, com boa luz e a página inteira, ou fale com a gente." + botão WhatsApp.
7. **Limite de uso atingido:** mostra "Muitas análises em pouco tempo. Tente novamente mais tarde ou fale com a gente pelo WhatsApp." + botão WhatsApp.
8. **Erro técnico** (timeout, serviço fora): não trava — mesma saída amigável + botão WhatsApp.

**Regras de segurança do conteúdo (CRÍTICO):**

- **Nunca** afirmar "está tudo em dia / seu pet está protegido" como conclusão geral. Status é sempre por vacina e sempre enquadrado como triagem. Falso "em dia" é perigoso (raiva/parvo).
- Sempre exibir disclaimer: "Isto é uma triagem automática, não um laudo. A leitura de imagem pode falhar e o que cada pet precisa depende de idade, espécie e estilo de vida. Quem confirma é a equipe."
- O objetivo é **puxar a conversa pro WhatsApp**, não dar veredito clínico.

**Arquitetura da análise (backend mínimo na Vercel):**

- **Uma única função serverless**: `src/pages/api/carteirinha.ts` (rota Astro com `prerender = false`, adapter `@astrojs/vercel`). O resto do site continua 100% estático.
- **Divisão de responsabilidades — "a IA lê, a regra decide":**
  1. A IA de visão (Anthropic, modelo Claude da linha Haiku ou equivalente de baixo custo — ID do modelo via variável de ambiente `MODELO_IA`) recebe a imagem e devolve **somente** JSON estruturado com: se está legível, espécie aparente, e a lista de vacinas com nome lido e data da última dose. Temperatura 0, `max_tokens` baixo.
  2. O **código da função** (determinístico) mapeia cada nome lido para um tipo de vacina e aplica `src/data/regras-vacinas.json` (intervalo de reforço por vacina e espécie, marcado `VALIDAR-VET`) comparando com a **data atual**. Vacina não reconhecida, sem data ou com data ilegível → "verificar".
  3. A IA **nunca** decide se está vencida e nunca gera texto exibido livremente ao usuário.
- **Validação da saída da IA:** o JSON é validado contra um esquema; qualquer desvio → tratado como ilegível. Todo texto vindo da IA (nomes de vacina) é **escapado** antes de exibir (a imagem pode conter texto malicioso).
- **Chave de API:** somente em variável de ambiente (`ANTHROPIC_API_KEY`) — no `.env` local (fora do Git) e no painel da Vercel. **Nunca** no código, no front-end, em logs ou em mensagens de erro.
- **Privacidade:** a imagem **não é armazenada** em lugar nenhum (nem em log). Processada em memória e descartada.

**Proteção contra abuso e custo (OBRIGATÓRIO — em camadas):**

1. **Validação de entrada:** só `POST`; só `image/jpeg`, `image/png`, `image/webp`; tamanho máximo 4 MB; recusa requisições sem o consentimento marcado; verifica `Origin` do próprio site.
2. **Limite de requisições persistente** com Upstash Redis (`@upstash/ratelimit`, integrado pelo Marketplace da Vercel; variáveis `UPSTASH_REDIS_REST_URL` e `UPSTASH_REDIS_REST_TOKEN`):
   - **por visitante (IP):** máx. 5 análises por hora e 10 por dia;
   - **global do site:** máx. 200 análises por dia (teto de custo).
   - Valores configuráveis em `cliente.json` (`carteirinha.limites`). O IP é usado apenas como chave temporária (com hash) e expira sozinho.
3. **Falha fechada:** se a chave da IA **ou** o limitador não estiverem configurados, a função **não chama a IA** e responde "não configurado" → o site cai no modo demonstração. Nunca rodar IA real sem limitador.
4. **Custo por chamada limitado:** imagem reduzida, `max_tokens` baixo, timeout de ~20s.
5. **Rede de segurança final (manual, fora do código):** limite de gasto mensal configurado no painel da Anthropic. Opcional: regra de rate limit no Firewall da Vercel para `/api/carteirinha`.

**Modo demonstração (`demoMode`):**

- `demoMode: true` em `cliente.json` → o front **não chama a API** e usa um mock local (útil para desenvolvimento e para clones sem chave configurada). Selo visível "Modo demonstração".
- `demoMode: false` → o front chama `/api/carteirinha`. Se a API responder "não configurado", o front cai automaticamente no mock com o selo.
- O mock usa **datas relativas à data atual** (ex.: V10 aplicada há 4 meses = em dia; antirrábica há 14 meses = verificar; gripe canina = verificar), nunca datas fixas.

**Formato de resposta da API (e do mock):**

```json
{
  "status": "ok",
  "legivel": true,
  "especie": "cão",
  "vacinas": [
    { "nome": "V10 (múltipla)", "ultima": "05/2026", "status": "ok" },
    { "nome": "Antirrábica",    "ultima": "07/2025", "status": "pendente" }
  ],
  "observacao": "Leitura automática; datas podem variar conforme a caligrafia."
}
```

Outras respostas possíveis: `{ "status": "ilegivel", "legivel": false, "motivo": "..." }`, `{ "status": "limite" }`, `{ "status": "nao_configurado" }`, `{ "status": "erro" }`. Nenhuma resposta de erro expõe detalhes internos.

---

## 8. Herói 2 — Agendamento em 30s no WhatsApp

**O que é:** mini-formulário que monta uma mensagem pronta e abre o WhatsApp da clínica. NÃO é sistema de agenda (sem backend, sem login). Seção logo abaixo do herói.

**Campos:** nome do pet (texto) · serviço (select, vindo de `servicosAgendamento` em `cliente.json`) · dia preferido (date) · período (Manhã / Tarde / Qualquer horário).

**Ação:** botão "Enviar pedido no WhatsApp" gera link `https://wa.me/<numero>?text=<msg>` com a mensagem preenchida, ex.:

> "Olá! Quero agendar *Consulta* para o *Thor*. Dia preferido: 14/06/2026 (Manhã). Vim pelo site."

**Regra:** apenas `wa.me` (link). **Não** usar a API oficial do WhatsApp (evitar qualquer dependência/risco de bloqueio). O número vem de `cliente.json`.

---

## 9. Demais seções (com conteúdo real)

### Nav
Logo (paw + "Clínica VetSaúde"), links: Serviços · Vacinas · Agendar · Plano · Equipe · Onde estamos. Botão WhatsApp à direita. Faixa/etiqueta "Emergência 24h" visível. Menu hambúrguer no mobile.

### Herói (texto)
- Etiqueta: "Emergência 24h — ligue agora" (com indicador visual).
- H1 (item de SEO mais importante): "Cuidado de verdade para quem faz parte da família."
- Subtítulo: "Clínica veterinária em Vila Mariana. Consultas, vacinas, exames, cirurgia e emergência 24h — com uma equipe que trata o seu pet como trataria o dela."
- CTAs: "Agendar em 30 segundos" (rola pro agendamento) + "Checar vacinas do pet" (rola pra carteirinha).
- Faixa de confiança: +10 anos · +8.000 tutores · 4,9★ no Google.

### Serviços (6 cards)
Consultas e exames · Vacinação · Cirurgia · Internação · Odontologia · Emergência 24h. Cada card: ícone, título, 1 frase. (Escrever frases reais, curtas, orientadas a benefício.)

### Plano VetSaúde (assinatura)
Bloco destacando o plano mensal: consultas de rotina + vacinas do calendário + descontos em exames. CTA "Quero saber do plano" → WhatsApp com mensagem específica. (Este bloco é uma **dimensão variável** — ver seção 12.)

### Equipe (grid de 4)
Foto (placeholder), nome, especialidade, CRMV. Reforça credibilidade e é exigência do conselho.

### Localização + horários
Endereço, botão "Como chegar" (link para Google Maps por endereço, sem API key), horários, faixa de emergência 24h com telefone.

### Depoimentos (3)
Escrever 3 depoimentos realistas de tutores (5★), incluindo um sobre emergência noturna e um sobre lembrete de vacina. Ideal futuro: puxar avaliações reais do Google.

> ⚠️ Estes depoimentos são de demonstração do template. Em cliente real, substituir obrigatoriamente por avaliações reais — nunca publicar depoimento inventado em site de cliente.

### Footer + LGPD
Contato, navegação, e nota LGPD: a foto da carteirinha é enviada a um serviço de inteligência artificial (Anthropic) apenas para leitura automática, não é armazenada, e o endereço de IP é usado de forma temporária só para limitar o uso da ferramenta; a triagem não substitui avaliação de veterinário. Controladora dos dados: a clínica (nome e contato vindos do `cliente.json`).

---

## 10. Camada "Pacote Captação" (incluída neste site como vitrine do arquétipo)

Três micro-ferramentas interativas, cada uma terminando em CTA de WhatsApp. Marcar no código como bloco de pacote (componentes isolados em `src/components/pacote-captacao/`, para virar módulo destacável no template).

1. **Calculadora de idade do pet em anos humanos** — por espécie e porte (NÃO usar o mito do ×7; cães variam por porte, gato tem curva própria). Resultado puxa: "quer um check-up adequado à fase dele?".
2. **Checklist "sinais de emergência — se vir qualquer um, ligue AGORA"** — lista de sinais graves (ex.: sangramento que não para, convulsão, dificuldade para respirar, abdômen inchado e duro, ingestão de veneno). **Sempre encaminha** ao telefone/WhatsApp 24h; **nunca** diz "pode esperar" nem minimiza. Risco baixo, valor alto.
3. **"Meu pet pode comer isso?"** — usar **lista fixa curada** de alimentos comuns (tóxicos: chocolate, uva/passa, cebola, alho, xilitol, macadâmia, álcool; seguros com ressalva: cenoura, maçã sem semente, banana). **Não** usar geração livre de IA aqui — um "seguro" errado machuca. Se o item não estiver na lista, orientar a perguntar à clínica.

---

## 11. Requisitos técnicos

- **Stack:** Astro + deploy na Vercel via GitHub, com adapter `@astrojs/vercel`. **Todas as páginas são estáticas** (pré-renderizadas no build). **Única exceção de backend:** a função `src/pages/api/carteirinha.ts` (seção 7). Nenhum outro endpoint, banco de dados ou função serverless.
- **Armazenamento:** nenhum banco de dados. **Única exceção:** Upstash Redis usado **exclusivamente** para contadores de limite de uso (chaves com hash e expiração automática). Nada de dado pessoal, imagem ou resultado armazenado.
- **Variáveis de ambiente** (nunca no código; documentadas em `.env.example` sem valores): `ANTHROPIC_API_KEY`, `MODELO_IA`, `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`. `.env` no `.gitignore`.
- **Um componente por seção** em `src/components/` (ex.: `Hero.astro`, `Servicos.astro`). Ferramentas do Pacote Captação em `src/components/pacote-captacao/`.
- **Dados do cliente isolados** em `src/data/cliente.json`: nome, WhatsApp, telefone, e-mail, endereço, horários, serviços, serviços do agendamento, equipe, depoimentos, prova social, mensagens prontas de WhatsApp, `demoMode`, `demoSite`, `carteirinha.limites`, `site` (URL) e as flags das dimensões variáveis (seção 12). **Nenhum dado do cliente hardcoded nos componentes.**
- **Regras de vacinas** em `src/data/regras-vacinas.json` (marcado `VALIDAR-VET`), usadas pelo código da função — nunca pela IA.
- **Marca em variáveis CSS** no `:root` (trocar cores = trocar tema), seguindo o `docs/design-system.md`.
- **Texto renderizado no HTML no build** (o Google precisa ler). Nada de injetar conteúdo textual via JS no navegador (exceto os resultados dinâmicos das ferramentas interativas).
- **JS no navegador apenas nas ferramentas interativas** (carteirinha, agendamento, pacote captação, menu mobile), como scripts isolados por componente.
- **Proibido** `localStorage`/`sessionStorage`. Estado em memória.
- **WhatsApp:** só via link `wa.me`. **Telefone:** link `tel:` (ação principal nos CTAs de emergência).
- **Fontes:** Google Fonts (ou fontes locais) — nenhuma outra dependência externa em tempo de execução no navegador.
- **Pontos de foto** marcados com `<!-- FOTO: ... -->` (placeholders elegantes: bloco com gradiente/ícone, sem depender de imagem externa que possa quebrar).
- **Responsivo** (mobile-first, testar 360px) e **acessível** (foco visível, `prefers-reduced-motion`, labels, contraste AA, mensagens de estado com `aria-live`).

---

## 12. Dimensões variáveis a marcar conscientemente

Mesmo neste cliente elas têm valor fixo, mas devem existir como **flags em `cliente.json`** e ser marcadas no código (comentário) onde o template vai precisar dobrar depois. Isto evita rachaduras nos próximos clientes:

- **Herói principal:** carteirinha (aqui) vs. slider antes/depois (Site 2).
- **Tem carteirinha?** sim/não (e, se sim, IA real ou só demonstração).
- **Emergência 24h?** sim/não (some a faixa e a etiqueta se não).
- **Espécies:** cão+gato / só cão / só gato (afeta textos).
- **Nº de veterinários / layout da equipe:** 1 até 4+ (grid flexível).
- **Tem plano de assinatura?** sim/não (mostra/oculta o bloco Plano).
- **Pacote:** Captação (aqui) vs. Marketing/Instagram (Site 2).
- **Tem banho e tosa?** sim/não (aqui não é destaque; no Site 2 é o herói).

---

## 13. SEO (é o diferencial de venda — capriche)

- `<title>` e `<meta name="description">` reais e locais (ex.: "Clínica veterinária em Vila Mariana | Consultas, Vacinas e Emergência 24h").
- Open Graph (título, descrição, imagem, locale pt_BR) e `<link rel="canonical">`.
- **JSON-LD `VeterinaryCare`** com nome, endereço, telefone, horário, URL.
- Um único `<h1>`; hierarquia de headings correta; HTML semântico; `alt` descritivo.
- Conteúdo textual no HTML gerado no build (ver seção 11), não em JS.

---

## 14. Fora de escopo (não construir agora)

- Galeria/slider de antes e depois de banho e tosa (isso é o **Site 2**).
- Login, área do tutor, prontuário, banco de dados (exceto os contadores de limite de uso da seção 11) e qualquer backend além da função `/api/carteirinha`.
- Armazenar imagens, resultados de triagem ou qualquer dado do tutor.
- Pagamento real / cobrança.
- Múltiplas páginas ou blog (pode ser add-on de SEO numa fase futura; agora é single page).
- Analytics / pixels de rastreamento (ponto comentado para o futuro).

---

## 15. Critérios de aceitação (definition of done)

- [ ] `npm run build` gera o site sem erros; todas as páginas são estáticas e a única rota dinâmica é `/api/carteirinha`.
- [ ] Acima da dobra, no celular, aparecem: quem é a clínica, emergência 24h e um CTA de ação (agendar/carteirinha) sem precisar rolar.
- [ ] Agendamento gera link `wa.me` com mensagem preenchida e abre o WhatsApp.
- [ ] Carteirinha em `demoMode`: upload → consentimento → analisar → resultado (mock com datas relativas) com disclaimer, selo "Modo demonstração" e botão de WhatsApp.
- [ ] Carteirinha com IA real (chave + limitador configurados): uma foto legível retorna vacinas lidas com status por vacina calculado pelas regras; uma foto ilegível ou que não é carteirinha retorna o estado "ilegível"; nunca afirma "está tudo em dia" como conclusão geral.
- [ ] Sem chave **ou** sem limitador configurado, a função não chama a IA e o site cai no modo demonstração.
- [ ] Limite de uso funciona: a 6ª análise na mesma hora pelo mesmo visitante recebe o estado "limite atingido"; o teto global diário existe e é configurável.
- [ ] Nenhuma chave de API no código, no front-end, no repositório, em logs ou em mensagens de erro; `.env` fora do Git; `.env.example` sem valores.
- [ ] Nenhuma imagem ou resultado é armazenado; texto vindo da IA é escapado antes de exibir.
- [ ] Checklist de emergência e "meu pet pode comer isso" funcionam e sempre encaminham ao contato (emergência: "Ligar agora" via `tel:` como ação principal); "pode comer" usa lista fixa, não IA.
- [ ] Dados do cliente isolados em `src/data/cliente.json` + variáveis CSS; nenhum dado do cliente hardcoded nos componentes; pontos de foto marcados com `FOTO`; conteúdo de saúde marcado com `VALIDAR-VET`.
- [ ] SEO: title, meta, OG, canonical, JSON-LD VeterinaryCare, um h1, HTML semântico, texto presente no HTML gerado; `noindex` + aviso de site demonstrativo quando `demoSite: true`.
- [ ] Responsivo a 360px, foco de teclado visível, `prefers-reduced-motion` respeitado.
- [ ] Visual segue o `docs/design-system.md` aprovado.
- [ ] Dimensões variáveis da seção 12 existem como flags em `cliente.json` e estão marcadas com comentários no código.
- [ ] `docs/deploy.md` explica, passo a passo e em linguagem simples, como configurar as variáveis de ambiente, o Upstash, o limite de gasto na Anthropic e o deploy na Vercel.

---

### Nota sobre uma decisão consciente (single page vs. múltiplas páginas)

Especifiquei **single page** de propósito: casa com o modelo de clonar rápido, hospedar barato e vender no volume, e para a intenção principal ("veterinário em <bairro>") uma página bem estruturada ranqueia bem. O trade-off honesto: SEO profundo (uma página por serviço + blog) rende mais no longo prazo. Isso fica como add-on de fase futura, não some da estratégia. Com Astro, adicionar páginas depois é simples.
