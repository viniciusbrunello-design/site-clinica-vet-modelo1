# PRD — Site 1: Clínica Veterinária (arquétipo clínico)

> Documento para o Claude Code construir o **Site 1** de referência. É um site single page, construído em **Astro** e publicado na **Vercel via GitHub**, que depois vira template. Construa fiel a este documento; onde faltar detalhe, siga o `docs/design-system.md` (após aprovado) e a seção "Requisitos Técnicos" em vez de improvisar.

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
- **Contato:** WhatsApp/telefone (11) 99999-9999 · contato@vetsaude.com.br · Rua Exemplo, 123 — Vila Mariana, São Paulo/SP
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

**O que é:** o tutor envia a foto da carteirinha; uma IA de visão lê e devolve uma **triagem** das vacinas, sempre encaminhando para o WhatsApp da clínica. É o gancho que diferencia o site. Fica no herói, à direita da headline.

**Faixa-âncora (banner) acima do widget** — usar estatística REAL, nunca número inventado:

> "Cinomose e parvovirose matam até 90% dos cães não vacinados. O seu pet está protegido? Envie a foto da carteirinha e descubra em segundos se há vacinas em atraso." *(fonte: letalidade de cinomose 50–90% em cães não vacinados; parvovirose de alta mortalidade em filhotes)*

**Fluxo e estados:**

1. Área de upload (arrastar ou tocar; no celular abrir câmera: `accept="image/*" capture="environment"`).
2. Prévia da imagem carregada.
3. **Checkbox de consentimento obrigatório** (LGPD) — botão "Analisar" só habilita com imagem + consentimento marcados.
4. Estado de carregando (spinner + "Lendo a carteirinha…").
5. Resultado: lista de vacinas lidas com status ("em dia" / "verificar"), + disclaimer, + botão grande de WhatsApp com as vacinas detectadas já escritas na mensagem.
6. Estado de erro: se a leitura falhar, não trava — mostra "não consegui ler, tente foto mais nítida ou fale com a gente" + botão WhatsApp.

**Regras de segurança do conteúdo (CRÍTICO):**

- **Nunca** afirmar "está tudo em dia / seu pet está protegido". Sempre enquadrar como triagem e remeter à clínica. Falso "em dia" é perigoso (raiva/parvo).
- Sempre exibir disclaimer: "Isto é uma triagem automática, não um laudo. A leitura de imagem pode falhar e o que cada pet precisa depende de idade, espécie e estilo de vida. Quem confirma é a equipe."
- O objetivo é **puxar a conversa pro WhatsApp**, não dar veredito clínico.

**Integração de IA (importante):**

- Implementar com **`demoMode: true`** por padrão (em `src/data/cliente.json`): retorna um resultado de exemplo (mock) após um pequeno delay, para o site ser demonstrável e vendável sem backend. Mostrar um selo "modo demonstração" quando ligado.
- Para produção: `demoMode: false` chama um endpoint configurável (`endpointCarteirinha` em `cliente.json`) — um webhook do N8N / backend do dono.
- **NUNCA colocar a chave de API no front-end** (ficaria exposta no navegador). A chamada real passa pelo backend do dono, que guarda a chave. Deixar isso comentado no código.
- Formato de retorno esperado do endpoint (igual ao mock):

```json
{
  "especie": "cão",
  "vacinas": [
    { "nome": "V10 (múltipla)", "ultima": "03/2024", "status": "ok" },
    { "nome": "Antirrábica",    "ultima": "01/2024", "status": "pendente" }
  ],
  "observacao": "Leitura automática; datas podem variar conforme a caligrafia."
}
```

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
Contato, navegação, e nota LGPD: imagens da checagem de vacinas usadas só para triagem, não armazenadas; triagem não substitui avaliação de veterinário.

---

## 10. Camada "Pacote Captação" (incluída neste site como vitrine do arquétipo)

Três micro-ferramentas interativas, cada uma terminando em CTA de WhatsApp. Marcar no código como bloco de pacote (componentes isolados em `src/components/pacote-captacao/`, para virar módulo destacável no template).

1. **Calculadora de idade do pet em anos humanos** — por espécie e porte (NÃO usar o mito do ×7; cães variam por porte, gato tem curva própria). Resultado puxa: "quer um check-up adequado à fase dele?".
2. **Checklist "sinais de emergência — se vir qualquer um, ligue AGORA"** — lista de sinais graves (ex.: sangramento que não para, convulsão, dificuldade para respirar, abdômen inchado e duro, ingestão de veneno). **Sempre encaminha** ao telefone/WhatsApp 24h; **nunca** diz "pode esperar" nem minimiza. Risco baixo, valor alto.
3. **"Meu pet pode comer isso?"** — usar **lista fixa curada** de alimentos comuns (tóxicos: chocolate, uva/passa, cebola, alho, xilitol, macadâmia, álcool; seguros com ressalva: cenoura, maçã sem semente, banana). **Não** usar geração livre de IA aqui — um "seguro" errado machuca. Se o item não estiver na lista, orientar a perguntar à clínica.

---

## 11. Requisitos técnicos

- **Stack:** Astro (saída 100% estática) + deploy na Vercel via GitHub. Sem backend, sem banco de dados.
- **Um componente por seção** em `src/components/` (ex.: `Hero.astro`, `Servicos.astro`). Ferramentas do Pacote Captação em `src/components/pacote-captacao/`.
- **Dados do cliente isolados** em `src/data/cliente.json`: nome, WhatsApp, telefone, e-mail, endereço, horários, serviços, serviços do agendamento, equipe, depoimentos, prova social, mensagens prontas de WhatsApp, `demoMode`, `endpointCarteirinha` e as flags das dimensões variáveis (seção 12). **Nenhum dado do cliente hardcoded nos componentes.**
- **Marca em variáveis CSS** no `:root` (trocar cores = trocar tema), seguindo o `docs/design-system.md`.
- **Texto renderizado no HTML no build** (o Google precisa ler). Nada de injetar conteúdo textual via JS no navegador.
- **JS apenas nas ferramentas interativas** (carteirinha, agendamento, pacote captação, menu mobile), como scripts isolados por componente.
- **Proibido** `localStorage`/`sessionStorage`. Estado em memória.
- **WhatsApp:** só via link `wa.me`.
- **Fontes:** Google Fonts (ou fontes locais) — nenhuma outra dependência externa em tempo de execução.
- **Pontos de foto** marcados com `<!-- FOTO: ... -->` (placeholders elegantes: bloco com gradiente/ícone, sem depender de imagem externa que possa quebrar).
- **Carteirinha:** função `analisarCarteirinha(file)` com `demoMode` (mock) e caminho de produção via `fetch` ao `endpointCarteirinha`; comentar no código a proibição de chave de API no front.
- **Responsivo** (mobile-first, testar 360px) e **acessível** (foco visível, `prefers-reduced-motion`, labels, contraste AA).

---

## 12. Dimensões variáveis a marcar conscientemente

Mesmo neste cliente elas têm valor fixo, mas devem existir como **flags em `cliente.json`** e ser marcadas no código (comentário) onde o template vai precisar dobrar depois. Isto evita rachaduras nos próximos clientes:

- **Herói principal:** carteirinha (aqui) vs. slider antes/depois (Site 2).
- **Tem carteirinha?** sim/não.
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
- Login, área do tutor, prontuário, qualquer backend, banco de dados ou funções serverless.
- Pagamento real / cobrança.
- Múltiplas páginas ou blog (pode ser add-on de SEO numa fase futura; agora é single page).
- Integração real de IA na carteirinha (fica em `demoMode`; o endpoint é plugado depois).

---

## 15. Critérios de aceitação (definition of done)

- [ ] `npm run build` gera o site estático sem erros e o deploy na Vercel funciona.
- [ ] Acima da dobra, no celular, aparecem: quem é a clínica, emergência 24h e um CTA de ação (agendar/carteirinha) sem precisar rolar.
- [ ] Agendamento gera link `wa.me` com mensagem preenchida e abre o WhatsApp.
- [ ] Carteirinha funciona em `demoMode`: upload → consentimento → analisar → resultado com disclaimer e botão de WhatsApp; nunca afirma "está tudo em dia".
- [ ] Checklist de emergência e "meu pet pode comer isso" funcionam e sempre encaminham ao contato; "pode comer" usa lista fixa, não IA livre.
- [ ] Dados do cliente isolados em `src/data/cliente.json` + variáveis CSS; nenhum dado do cliente hardcoded nos componentes; pontos de foto marcados com `FOTO`.
- [ ] SEO: title, meta, OG, canonical, JSON-LD VeterinaryCare, um h1, HTML semântico, texto presente no HTML gerado.
- [ ] Responsivo a 360px, foco de teclado visível, `prefers-reduced-motion` respeitado.
- [ ] Visual segue o `docs/design-system.md` aprovado.
- [ ] Dimensões variáveis da seção 12 existem como flags em `cliente.json` e estão marcadas com comentários no código.

---

### Nota sobre uma decisão consciente (single page vs. múltiplas páginas)

Especifiquei **single page** de propósito: casa com o modelo de clonar rápido, hospedar barato e vender no volume, e para a intenção principal ("veterinário em <bairro>") uma página bem estruturada ranqueia bem. O trade-off honesto: SEO profundo (uma página por serviço + blog) rende mais no longo prazo. Isso fica como add-on de fase futura, não some da estratégia. Com Astro, adicionar páginas depois é simples.
