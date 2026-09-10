# SDD: Honestidade da vitrine `/produtos`

- **Módulo:** frontend (Angular)
- **Superfície / rotas:** `/produtos` (lista) e `/produtos/:id` (detalhe — mesmo serviço de leitura)
- **Inventário Impeccable:** [`docs/sprints/frontend/inventory-impeccable-2026-09-10.md`](../../sprints/frontend/inventory-impeccable-2026-09-10.md) — **ainda não está em `main`**; vive no [PR #2](https://github.com/A-DAVI/MyBuddy/pull/2) (`cursor/inventory-impeccable-frontend-0505`). Se o merge do inventário ocorrer antes desta implementação, o caminho acima passa a ser canônico.
- **Decisão de grilling:** **seguir** (já grillado — não reabrir). Fatia: matar sucesso silencioso na leitura do catálogo.
- **Shape brief:** seção [Shape / UX](#shape--ux) neste arquivo.
- **Runbook:** [`docs/sprints/frontend/runbook-honestidade-vitrine-produtos.md`](../../sprints/frontend/runbook-honestidade-vitrine-produtos.md)
- **Status da spec:** decidida. Código de produto **não** entra neste PR.

---

## Contexto

O inventário Impeccable (2026-09-10) marcou a vitrine marketplace como **P0**: `ProdutoService` tenta HTTP e, no `catchError`, devolve mock/`localStorage` como se a API tivesse respondido. A tela `/produtos` não tem estados de loading / vazio / erro da **carga inicial** — só um “nenhum produto encontrado” quando o filtro local zera a lista. O detalhe `/produtos/:id` usa o mesmo fallback.

O grilling escolheu **esta** superfície (não perfil, não checkout) com apetite estreito: honestidade da leitura. O backend `ProdutoController` **já existe** e os GETs de catálogo são `permitAll`; não há SDD Java neste ciclo a menos que o contrato se prove quebrado na implementação.

Loop do processo: inventário → grilling (seguir) → **este SDD** → runbook → só então PR de produto.

---

## Shape / UX

### Job / audiência

**Quem:** visitante da vitrine pública (adotante, tutor, qualquer pessoa — rotas **sem** `authGuard`).

**Job:** ver o catálogo real da plataforma (produtos e, se a API responder, categorias) para decidir o que comprar. Não é o painel do petshop nem o checkout.

### Outcome / prova

A pessoa consegue distinguir:

1. “ainda estou carregando o catálogo”
2. “a API respondeu e não há produtos (ou o filtro não bateu)”
3. “a API falhou — isto **não** é um catálogo verdadeiro”
4. “estes cards vieram de `GET produtos`”

**Prova (não opinião de design):**

- Com API fora do ar, `/produtos` **não** mostra os 16 itens seed de `mybuddy_produtos_local`.
- Specs Vitest de `ProdutoService` **falham** se `buscarComFiltros` / `buscarPorId` / `buscarCategorias` completarem com mock após erro HTTP.
- Estados loading / empty / error são visíveis na lista sem mudar o layout atual (hero, busca, carrossel, grades).

### Direção

Manter a vitrine como está visualmente. Trocar a **fonte de verdade** da leitura: HTTP da API, falha visível. Sem redesenho, sem novo design system, sem extrair o god component `marketplace.ts` além de um recorte mínimo de carga da lista se isso deixar o estado explícito.

### Escopo / fronteiras

| Entra | Não entra |
|-------|-----------|
| `ProdutoService` nos caminhos de **leitura** | CRUD do painel petshop (`criar` / `atualizar` / `deletar`) |
| UI de `/produtos`: loading / empty / error no layout vigente | Checkout, persistência de carrinho, Mercado Pago |
| Mesma honestidade de serviço em `/produtos/:id` | Rewrite de `marketplace.ts` (banner, favoritos, drawer, toast) |
| Ajuste trivial de empty/error no detalhe | Split completo do god component |
| Vitest no serviço (e smoke Playwright opcional) | SDD/código Java, Flyway, Prettier em massa |

### Estados / faixas (`/produtos`)

| Estado | Quando | O que a pessoa vê (sem redesenhar) |
|--------|--------|-------------------------------------|
| **Loading** | `buscarComFiltros` em voo (carga inicial e recargas da lista) | Indicador no **miolo do catálogo** (grades de ofertas / mais vendidos / resultados). Hero, busca e chrome permanecem. Não renderizar cards de mock. |
| **Success** | HTTP 200 com `content` (ou lista) **não vazio** | Layout atual: carrossel, banners, ofertas, mais vendidos, catálogo. Dados mapeados do DTO da API. |
| **Empty** | HTTP 200 com lista vazia **ou** filtro/busca sem matches | Distinguir se for barato: (a) catálogo da API vazio vs (b) filtro local sem resultado (já existe `.no-results` em `marketplace.html`). Não preencher com seed. |
| **Error** | HTTP 4xx/5xx, rede, timeout — o observable **erra** | Mensagem de falha no miolo da lista + ação de **tentar de novo** (dispara `carregarProdutos` de novo). **Proibido** pintar o seed de `localStorage` como sucesso. |

**Detalhe `/produtos/:id` (mínimo):**

- Já existe `.carregando` enquanto `produto` é `null` (`detalhes-produto.html`). Manter.
- Depois da honestidade do serviço, 404/rede **não** podem mostrar produto seed.
- Empty/error honestos **só se for trivial** (reusar o bloco `.carregando` com texto de falha / “não encontrado”, ou permanecer na página sem navegar para a lista fingindo que o item existia). **Não** redesenhar a PDP.

### Interação / layout

- **Não** mudar hierarquia visual: hero, CTA “Explorar Ofertas”, painel de busca, chips de filtro, carrossel de categorias, banners, grades de cards, FAB/drawer, footer.
- Loading/empty/error entram **no lugar dos grids de produto**, não como página nova nem modal de redesign.
- Filtros, ordenação, favoritos `mybuddy_favoritos` e carrinho em memória **continuam** como hoje; só deixam de ser alimentados por catálogo fake.
- Carrossel de categorias hoje é **hardcoded** em `marketplace.ts` (não chama `buscarCategorias`). Esta fatia **não** exige religar o carrossel na API. O serviço, se chamado, tem de ser honesto.

### Restrições / decisões em aberto

**Restrições (fechadas no grilling):**

- Sem `catchError` → `of(localStorage/mock)` nos reads.
- Sem `authGuard` novo nesta fatia (`/produtos` permanece público, alinhado a `GET /api/produtos/**` `permitAll`).
- Sem SDD de backend salvo contrato comprovadamente quebrado.
- Sem Prettier no tree; formatar só arquivos que o PR de **implementação** tocar.

**Em aberto (a implementação escolhe o mais barato; não voltam ao grilling):**

1. Seed no construtor (`inicializarProdutosLocais`): writes do painel ainda usam `localStorage` (fora desta fatia). Preferir **não** semear o catálogo só porque a vitrine injetou o serviço; não é obrigatório apagar os helpers de write.
2. Empty da API vs empty de filtro: dois textos se couber no HTML atual; um empty genérico é aceitável se o diff for menor.
3. Extração minúscula de `carregarProdutos` vs flags no próprio `marketplace.ts`.
4. Playwright smoke: obrigatório só se o Vitest não der para afirmar a UI; senão opcional.

---

## Problema

Quem abre `/produtos` (e `/produtos/:id`) **não consegue saber** se está vendo o Postgres da plataforma ou um catálogo plantado no browser.

Evidência (main atual):

- `ProdutoService` grava/lê a chave `mybuddy_produtos_local` e, nos reads, engole o erro HTTP (`produto.service.ts`).
- O construtor **semeia ≥ 16 produtos** se o storage estiver “incompleto” (`inicializarProdutosLocais`).
- `marketplace.ts` (~819 linhas) é god component: no `error` de `carregarProdutos` só faz `console.error` — e o `error` quase nunca dispara, porque o serviço já converteu falha em `next(mock)`.
- A spec **celebra** o fallback (`produto.service.spec.ts`: “should fall back to localStorage on API error”).

Isso viola [`docs/CONVENTIONS.md`](../../CONVENTIONS.md) (catálogo = API primeiro; `catchError` que finge sucesso é anti-padrão) e o anti-padrão de [`AGENTS.md`](../../../AGENTS.md).

---

## Objetivos

1. Reads de `ProdutoService` (`buscarComFiltros`, `buscarPorId`, `buscarCategorias`, no mínimo) **propagam erro**. Sem `of(mock)` / `of(localStorage)`.
2. `/produtos` mostra **loading / empty / error** no layout vigente.
3. `/produtos/:id` deixa de mostrar produto seed quando a API falha; empty/error na PDP só se for trivial.
4. Testes deixam de tratar fallback silencioso como comportamento desejado.

---

## Não-objetivos

- Checkout, `CartService`, persistência de carrinho.
- CRUD do painel petshop (`criar` / `atualizar` / `deletar` / `avaliarProduto` e respectivos fallbacks).
- Mercado Pago, Keycloak, `mockUserRole`.
- Rewrite Java (`ProdutoController` / `ProdutoService` backend / Flyway).
- Dump Prettier / ESLint no frontend inteiro.
- Split completo de `marketplace.ts` (banner autoplay, toast, FAB, favoritos, mapeamento de DTO no mesmo arquivo). Extração **opcional** só da carga da lista.
- Religar o carrossel hardcoded a `buscarCategorias`.
- Auth na vitrine, SSR, redesign, PrimeNG `p-dataview`.

---

## Comportamento atual

### Serviço — silent success

Arquivo: `frontend/src/app/core/services/produto.service.ts` (serviço inchado: HTTP + motor de mock, ~573 linhas).

| Peça | Evidência | O que acontece |
|------|-----------|----------------|
| Chave LS | `localProdutosKey = "mybuddy_produtos_local"` (~L25) | Fonte de verdade alternativa |
| Seed | `inicializarProdutosLocais()` no construtor (~L285–291, array ~L291–569) | Se `length < 16`, imagens Unsplash, falta `marca` ou `precoAntigo` → grava 16 itens fake |
| `buscarComFiltros` | `api.get("produtos" + query)` (~L31–56) | `map` extrai `res.content`; **`catchError` → `of(obterProdutosLocaisFiltrados(filtros))`** |
| `buscarPorId` | `api.get("produtos/${id}")` (~L59–67) | **`catchError`**: achou no LS → `of(produto)`; senão `throwError("…mock local")` |
| `buscarCategorias` | `api.get("categorias")` (~L100–112) | **`catchError` → `of([{id:1,nome:"Alimentação",…}, …])`** (5 categorias hardcoded) |
| Writes | `criar` / `atualizar` / `deletar` / `avaliarProduto` | Também têm fallback local — **fora desta fatia** |

`ApiService` prefixa `environment.apiUrl` (`/api/` em prod, `http://localhost:8081/api/` em dev). Paths do serviço **não** repetem `/api/`.

### Tela lista — god component sem estados de carga

- Rota pública: `app.routes.ts` ~L73–76 → `Marketplace`.
- `marketplace.ts` ~819 linhas / ~23 KB: busca, filtro, banner, favoritos `mybuddy_favoritos`, drawer, toast, mapeamento de DTO.
- `carregarProdutos()` (~L194–223): `buscarComFiltros().subscribe({ next: mapeia e preenche `todosProdutos` / ofertas / mais vendidos, error: `console.error` })`.
- **Não** há flag de loading da API nem estado de erro de carga. `isSearching` (L77, HTML L85) é modo de filtro/busca, não spinner de HTTP.
- Empty existente: `.no-results` (HTML ~L119–125) só quando `filteredProdutos.length === 0` **já no modo busca**.
- Categorias do carrossel: array estático `categorias: CategoriaVisual[]` (~L264+), **não** `buscarCategorias()`.

### Tela detalhe

- Rota pública: `app.routes.ts` ~L78–83 → `DetalhesProduto`.
- `carregarProduto` (~L76–113): `buscarPorId`; sucesso preenche signal (e inventa `precoAntigo`, specs e avaliações default se o DTO vier incompleto — **não** é o silent success desta fatia); `error` faz `console.error` + `navigate(["/produtos"])`.
- HTML: `@else` com `.carregando` (~L163–168) enquanto `produto` é null — loading da PDP **já existe**.
- `carregarRecomendados` usa lista hardcoded (Unsplash) — **fora de escopo**.

### Testes que cristalizam o bug

`produto.service.spec.ts`:

- L78–85: `buscarComFiltros` em erro HTTP espera `result.length >= 16`.
- L87–107: filtro de fallback no LS.
- L121–129: `buscarPorId` em erro HTTP espera o produto `id === 1` do seed.
- L240–248: `buscarCategorias` em erro HTTP espera 5 categorias mock.

Não há spec de `marketplace.ts`. Playwright (`e2e/login.spec.ts`, `e2e/checkout.spec.ts`) **não** cobre `/produtos`. CI `e2e-frontend` não roda Vitest.

---

## Comportamento desejado

### `ProdutoService` (reads)

Para `buscarComFiltros`, `buscarPorId` e `buscarCategorias`:

1. Chamar a API como hoje (`produtos`, `produtos/{id}`, `categorias` + query params existentes).
2. Sucesso: devolver o payload mapeado (`content` da página ou lista; objeto do detalhe; array de categorias).
3. Erro: **não** `catchError` para `of(...)`. Deixar o erro subir (interceptor global continua; a tela trata).
4. Não ler nem escrever `mybuddy_produtos_local` nesses três métodos.

Writes (`criar` / `atualizar` / `deletar` / `avaliarProduto`) **permanecem** como estão nesta fatia.

Helpers privados de mock podem ficar para os writes. Construtor: preferir não semear 16 produtos só porque a vitrine carregou o serviço (decisão em aberto #1).

### `/produtos`

`carregarProdutos` (ou extração mínima) passa a ter estados explícitos:

- `loading === true` ao disparar; `false` em `next` e `error`.
- `error` preenchido só no `error` do subscribe; limpar ao retentar.
- Success com array vazio → empty (não “sucesso com seed”).
- Retry chama de novo o mesmo método.

Layout: hero/busca/carrossel podem permanecer visíveis; **cards de produto** só no success com dados. Reusar classes já existentes (`.no-results`, `.carregando` da PDP, `app-empty-state` em `shared/`) se o diff for menor que inventar bloco novo.

Filtro/ordenação **client-side** sobre a lista **já recebida da API** continua válido. Não reintroduzir filtro sobre o seed.

### `/produtos/:id`

- Loading atual permanece.
- Sem produto seed.
- 404 vs rede: se for uma linha, empty “não encontrado” vs error “não foi possível carregar”; senão um único error no slot `.carregando` basta. Evitar `navigate(["/produtos"])` que esconde a falha, **a menos** que 404 e a lista seja o empty state natural — aí documentar no PR de implementação.

---

## Contratos de API (já existentes — sem SDD Java)

Prefixo HTTP real: `{apiUrl}` + path do `ApiService`. `apiUrl` já inclui `/api/` (dev: `http://localhost:8081/api/`).

Assunção do grilling: estes endpoints existem e são o contrato. **Só** abrir SDD em `docs/sdd/backend/` se a implementação provar que o contrato está quebrado (shape inesperado, 401 em GET público, 500 sistemático, etc.).

### `GET produtos`

| | |
|--|--|
| Front | `ProdutoService.buscarComFiltros` → `GET produtos?busca&categoriaId&subCategoriaId&petshopId&precoMin&precoMax` |
| Back | `ProdutoController.buscarComFiltros` — `@RequestMapping("/api/produtos")` + `@GetMapping` |
| Auth | `GET /api/produtos/**` `permitAll` (`SecurityConfig`) |
| Query extra no back (não enviada pelo front hoje) | `lastId`; `Pageable` default `size=12`, `sort=id DESC` |
| 200 | `Page<ProdutoResponseDTO>` (`content`, paginação Spring). O front já faz `res.content ?? res`. |
| Erro | 4xx/5xx — o front **não** traduz para lista fake |

Campos úteis do DTO (`ProdutoResponseDTO`): `id`, `nome`, `descricao`, `preco`, `estoque`, `status`, `subCategoriaId`/`Nome`, `categoriaId`/`Nome`, `petshopId`/`Nome`, `imagens`, `notaMedia`, `marca`, `origem`, `porteRaca`, `peso`, `idade`.

`precoAntigo` **não** está no DTO Java; o mapeamento atual usa `p.precoAntigo \|\| p.precoOriginal`. Não inventar campo no back nesta fatia; badge de desconto simplesmente some se a API não mandar o dado.

### `GET produtos/{id}`

| | |
|--|--|
| Front | `buscarPorId(id)` → `GET produtos/{id}` |
| Back | `ProdutoController.buscarPorId` → `produtoService.buscarPorIdDTO` |
| 200 | `ProdutoResponseDTO` |
| 404 | `ResourceNotFoundException` → `GlobalExceptionHandler` `HttpStatus.NOT_FOUND` |
| Erro | não mascarar com seed |

### `GET categorias`

| | |
|--|--|
| Front | `buscarCategorias()` → `GET categorias` |
| Back | `CategoriaController.listarTodas` — `@RequestMapping("/api/categorias")` + `@GetMapping` |
| Auth | `GET /api/categorias/**` `permitAll` |
| 200 | `List<CategoriaResponseDTO>` (`id`, `nome`, `subcategorias`) |
| Erro | não mascarar com as 5 categorias mock |

A vitrine **pode** continuar com o carrossel estático. O método do serviço, se usado (painel petshop já chama), tem de falhar de verdade.

### Fora deste contrato (não tocar)

`POST/PUT/DELETE /api/produtos`, `GET /api/produtos/recomendados` (autenticado), `GET /api/produtos/petshop/{id}`, avaliações, pedidos, pagamentos.

---

## Critérios de aceite (testáveis)

1. **AC-1 — sem silent success na lista.** Com `ApiService.get("produtos")` em `throwError`, `buscarComFiltros` **não** emite array; o subscriber cai em `error`. Não há `of(obterProdutosLocaisFiltrados)`.
2. **AC-2 — sem silent success no detalhe.** Com `get("produtos/{id}")` em erro, `buscarPorId` **não** emite o seed `id === 1` (nem qualquer item de `mybuddy_produtos_local`).
3. **AC-3 — sem silent success em categorias.** Com `get("categorias")` em erro, `buscarCategorias` **não** emite as 5 categorias mock (“Alimentação”, …).
4. **AC-4 — sucesso HTTP inalterado.** 200 com `content` / objeto / lista continua mapeado como hoje (query params de filtro inclusive).
5. **AC-5 — loading na vitrine.** Abrir `/produtos` com GET em voo: o miolo do catálogo mostra loading; **zero** cards do seed.
6. **AC-6 — empty na vitrine.** GET 200 `content: []` (sem filtro): empty visível, não grades fake. Filtro sem match pode reusar `.no-results`.
7. **AC-7 — error na vitrine.** GET 500/rede: mensagem de erro + retry; **proibido** pintar 16 produtos Premier/Bravecto/etc. do seed.
8. **AC-8 — detalhe honesto.** GET `produtos/{id}` 404 ou rede: não renderiza PDP seed. Empty/error trivial **ou** (só 404) volta à lista **sem** ter mostrado fake. Loading `.carregando` permanece no happy path.
9. **AC-9 — layout.** Hero, busca, carrossel, banners, cards, drawer: mesma estrutura; estados novos só no miolo da lista.
10. **AC-10 — specs honestas.** Remover ou inverter os `it("should fall back…")` / `should return mock categories on API error`. Novos testes afirmam `error`. Proibido `expect(result.length).toBeGreaterThanOrEqual(16)` após falha HTTP nos reads.
11. **AC-11 — fora de escopo intacto.** Sem mudanças em checkout, `CartService`, Mercado Pago, controllers Java, Flyway, painel petshop além do efeito colateral de `buscarCategorias`/`buscarComFiltros` deixarem de mascarar erro (o painel **pode** passar a ver falha — aceitável e desejável; não “corrigir” o painel com novo mock).
12. **AC-12 — diff de produto.** PR de implementação toca o recorte (`produto.service.ts` + spec, `marketplace.ts/html` no mínimo; detalhe só se trivial). Sem reformatar o repo.

---

## Plano de testes

### Vitest (obrigatório) — `produto.service.spec.ts`

TDD no recorte ([`.ai/skills/tdd.md`](../../../.ai/skills/tdd.md)):

1. **Red:** testes de erro HTTP em `buscarComFiltros` / `buscarPorId` / `buscarCategorias` esperam `error` (não `next` com seed).
2. **Green:** remover `catchError` → `of(mock)` desses métodos.
3. Manter testes de sucesso (path, query params, `content` vs array).
4. Apagar testes cujo nome/assertiva **celebra** fallback de read.
5. Writes: **não** reescrever a suíte de `criar`/`atualizar`/`deletar`/`avaliarProduto` nesta fatia.
6. Seed no construtor: se a implementação parar de semear, atualizar `should initialize local products in constructor`; se permanecer por causa dos writes, deixar o teste de write intacto.

HTTP: mock de `ApiService` (já é o padrão do spec) ou `HttpTestingController`. **Não** validar “sucesso” via `localStorage` nos reads.

Rodar: `cd frontend && npx vitest run src/app/core/services/produto.service.spec.ts` (e `npm test` no recorte se o runbook de implementação pedir).

### Vitest de tela (opcional, preferível se barato)

Não há spec de `marketplace.ts` hoje. Só vale um spec estreito de `carregarProdutos` (loading → success / empty / error) se a extração mínima existir. **Não** montar a tela-deus inteira “para cobertura”.

### Playwright (opcional)

Novo `frontend/e2e/produtos.spec.ts` **somente** se o time quiser prova de UI:

- `page.route('**/api/produtos*')` → 200 `{ content: [] }` → empty.
- Mesmo route → 500 → error visível, sem cards do seed (assertiva negativa: não ver “Ração Premier Formula…”).
- 200 com 1 item sintético (`ada@example.test` / nomes fake, sem PII) → card visível.

Não depende de `mybuddy_produtos_local`. Não precisa de Keycloak (rota pública). Não misturar com `checkout.spec.ts`.

CI hoje só roda Playwright, não Vitest — **não** é desculpa para pular o spec do serviço.

### Manual (PR de implementação)

1. API no ar, catálogo com dados: `/produtos` igual ao layout atual, dados reais.
2. API no ar, zero produtos: empty, não seed.
3. Backend parado: error + retry; DevTools → Network falha; Application → `localStorage` **não** alimenta a grade.
4. `/produtos/:id` existente / 404 / API down.

---

## Riscos / no-gos

### Riscos

| Risco | Mitigação |
|-------|-----------|
| Painel petshop (`meus-produtos`) quebra visualmente quando a API falha, porque também usa `buscarComFiltros` / `buscarCategorias` | Aceito. Não adicionar mock novo no painel. Fora de escopo “consertar” o CRUD. |
| Construtor continua semeando LS; alguém lê a chave em outro ponto | Grep `mybuddy_produtos_local` no PR de implementação; reads da vitrine não podem usar. |
| `Pageable size=12` vs vitrine que espera “o catálogo todo” para filtrar no cliente | Comportamento **já** é assim no HTTP de sucesso. Não paginar de novo nesta fatia. Se a lista “sumir” itens além da página 0, isso é dívida **posterior**, não silent success. |
| Interceptor de erro global + subscribe local = toast duplicado | Tratar na tela da vitrine; não engolir no serviço. |
| Tentação de “enquanto a API não sobe, deixa o mock” | **No-go.** Flag de dev explícita também está fora do apetite. |

### No-gos (hard)

- `catchError(() => of(mock))` / `of(this.obterProdutosLocais*)` nos três reads.
- Tratar `localStorage` como fonte de verdade do catálogo da vitrine.
- Redesenhar `/produtos` ou a PDP.
- Split completo de `marketplace.ts`.
- Checkout, carrinho persistente, Mercado Pago.
- SDD ou código Java “por higiene”.
- Prettier/ESLint no monorepo.
- Reabrir ADR-001…005, desligar Keycloak, `mockUserRole`.
- PII em fixture/e2e.
- Issue/PR mencionando `@claude` ou Actions de Claude Code.
- Transformar este SDD num rewrite do frontend.

### Rabbit holes (não entrar)

SSR/hidratação da vitrine, religar carrossel a `GET categorias`, `GET produtos/recomendados`, paginação infinita (`lastId`), extração de favoritos, persistir carrinho, auth na vitrine, PrimeNG table, JaCoCo/backend tests.

---

## Runbook

Implementação: [`docs/sprints/frontend/runbook-honestidade-vitrine-produtos.md`](../../sprints/frontend/runbook-honestidade-vitrine-produtos.md) (status `planned`).

Sem runbook não há sprint; sem este SDD não há runbook. **Este documento não autoriza código neste PR.**
