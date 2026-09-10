# Inventário Impeccable — frontend Angular

**Data:** 2026-09-10  
**Branch de origem:** `main` (`4f22597`)  
**Papel:** inventário (lista + evidência). **Não** é grilling, **não** é SDD, **não** é backlog de sprint.

**Método:** o comando nativo Impeccable `/audit` (passe visual) não está neste ambiente. Equivalente por código, alinhado a [`.ai/skills/impeccable-audit.md`](../../../.ai/skills/impeccable-audit.md) e [`.ai/skills/frontend-review.md`](../../../.ai/skills/frontend-review.md): rotas, tamanhos, `mockUserRole` / `localStorage` / `catchError`, HTTP vs fallback, PrimeNG, specs.

**Escopo:** `frontend/src/**`. Mobile Flutter e backend Java **não** foram olhados.

---

## Resumo executivo

- `app.routes.ts` mistura LP/institucional **públicos**, catálogo/checkout/doações **sem guard**, home/pets/perfil com `authGuard`, e painéis `admin` / `ong-panel` / `petshop-panel` com `roleGuard`. Checkout e marketplace não exigem sessão.
- Três telas-deus no sentido da skill: `marketplace.ts` (819 linhas / 23 KB), `perfil.ts` (709 / 24 KB), `cadastro-escolha-perfil.ts` (445 / 13 KB). Cada uma empilha fetch, layout, mock e regras de negócio.
- Login **bate Keycloak de verdade** (`grant_type=password` + `GET usuarios/meu-perfil`). O cheiro de auth fake está em `SessionService`: papel persistido em `localStorage` com a chave `mockUserRole`, lido no construtor e usado pelo layout dos painéis e por `NotificationService`.
- Catálogo, pedidos, cupons, doações e home de pets recentes **tentam HTTP e, no `catchError`, devolvem mock/localStorage como sucesso**. O usuário não vê API fora do ar.
- `/eventos` e `/servicos` são arrays hardcoded no componente (sem `*Service`). `/perfil` busca o usuário na API, mas pets/favoritos/mensagens/solicitações são signals estáticos (Unsplash).
- PrimeNG está em `app.config` + preset Aura, mas o uso real é pontual (`login` autocomplete, `modal`, rota `/style-guide`). Telas grandes são CSS próprio por feature.
- Testes: **1 spec em `features/`** (`styleguide.spec.ts` — `should create`). Login e checkout/pagamento têm Playwright. CI (`e2e-frontend`) **não** roda Vitest. Telas-deus sem unidade.

---

## Tabela

| Superfície | Rota | Severidade | Evidência | Nota |
|------------|------|------------|-----------|------|
| Landing | `/` | P2 | `app.routes.ts:39-42` | Público. LP + SSR prerender (`app.routes.server.ts:4-7`). |
| Home autenticada | `/home` | P1 | `app.routes.ts:44-47`; `home.ts:54-124`, `150-154`; `pet.service.ts:19-55` | `authGuard`. Pets recentes: HTTP com fallback mock; eventos/vets/lembretes/produtos são constantes no TS. `catchError(() => of([]))` esvazia a lista sem erro visível. |
| Feed de pets | `/pets` | P1 | `app.routes.ts:48-52`; `pets.ts:92-112` | `authGuard`. `PetService.buscarTodos()` é HTTP puro. Erro só faz `console.error` + `carregando=false` (lista vazia, sem estado de erro). Filtros disparam `setTimeout` e **não** reconsultam a API. `pet-details` existe e **não** está nas rotas. |
| Marketplace / vitrine | `/produtos` | P0 | `app.routes.ts:72-76`; `marketplace.ts:68-115`, `194-245`, `819` linhas / 23172 B; `produto.service.ts:25-56`, `285-291` | **Sem guard.** God component: busca, filtro, banner, favoritos `mybuddy_favoritos`, drawer, toast. `ProdutoService` no `catchError` devolve `mybuddy_produtos_local` (catálogo seed se `< 16` itens). |
| Detalhe de produto | `/produtos/:id` | P0 | `app.routes.ts:77-83`; `produto.service.ts:59-67` | Público. Mesmo fallback local no GET por id. |
| Meus pedidos | `/meus-pedidos` | P0 | `app.routes.ts:85-90`; `pedido.service.ts:42-50` | **Sem `authGuard`.** Lista HTTP `pedidos/meus` → fallback `mybuddy_pedidos_local`. |
| Checkout | `/checkout` | P0 | `app.routes.ts:13-17`; `checkout.ts:35-40`, `123-208` | **Sem `authGuard`.** Carrinho só em memória (`CartService`). CEP mock (`87013000`). `realizarPagamentoMock()`; se `criarPedido` falha, **avança mesmo assim** (`checkout.ts:197-200`). Cartão/CVV em campos de tela. |
| Pagamento MP (doação/pet) | `/checkout/pagamento` | P1 | `app.routes.ts:18-24`; `e2e/checkout.spec.ts:63-68`; `payment.service.ts:30-32` | Público. E2E entra com query `petId`/`amount` **sem login**. `PaymentService.createPayment` é HTTP real (`payments/create`). |
| Confirmação / pendente | `/checkout/confirmacao`, `/pendente` | P1 | `app.routes.ts:25-36` | Sem guard. `checkout/endereco` é stub (`endereco.html`: `works!`) e **não** está nas rotas. |
| Perfil | `/perfil` | P1 | `app.routes.ts:98-103`; `perfil.ts:229-370`, `450-456`, `709` linhas / 23593 B | `authGuard`. Dados cadastrais: `UserService.buscarPerfil()` (HTTP). Pets, favoritos, mensagens e solicitações: arrays Unsplash no próprio componente. 8 abas no mesmo arquivo. |
| Login | `/auth/login` | P2 | `login.ts:91-114`; `auth.service.ts:67-92`; `auth.guards.ts:5-15` | HTTP Keycloak ROPC + perfil. Erro 401/rede visível. Playwright cobre sucesso e falha. Sem spec Vitest da tela. |
| Cadastro wizard | `/auth/cadastro` | P1 | `app.routes.ts:105-125`; `cadastro-escolha-perfil.ts:343-389`, `445` linhas; HTML 358 linhas / 15625 B | Público. `AuthService.registrar` → `POST auth/cadastro`. God component (passos, papéis, org, senha). Redirects `.../adotante\|ong\|petshop` voltam para a mesma rota. |
| Recuperar senha | `/auth/recuperar-senha` | P1 | `app.routes.ts:127-132`; `recuperar-senha.html:1` | Rota viva; template é `<p>recuperar-senha works!</p>;`. |
| Doações | `/doacoes` | P0 | `app.routes.ts:91-97`; `donation.service.ts:46-72`, `103-231` | Público. `getStats` / `getCampaigns` / `getOngsParceiras`: HTTP + `catchError` → números e campanhas inventados. `createSingleDonation` / subscribe **sem** fallback (HTTP). |
| Eventos | `/eventos` | P1 | `app.routes.ts:58-64`; `eventos.component.ts:28-71` | Público. **Zero HTTP** — array no componente (Unsplash). |
| Serviços | `/servicos` | P1 | `app.routes.ts:65-71`; `servicos.component.ts:29-80` | Público. **Zero HTTP** — array no componente. |
| Style guide | `/style-guide` | P2 | `app.routes.ts:53-57`; `styleguide.ts`; `app.routes.server.ts:20-23` | Público + prerender. Showcase PrimeNG. Único spec de feature (`should create`). |
| Empty state (dev) | `/empty-state` | P2 | `app.routes.ts:6-12` | Componente shared exposto como rota de produção. |
| ONG pets (fora do painel) | `/ong/pets`, `/ong/pets/novo` | P1 | `app.routes.ts:139-152` | Só `authGuard` — **não** `roleGuard`. Qualquer logado acessa cadastro de pet ONG. |
| Painel admin | `/admin/**` | P1 | `app.routes.ts:153-197`; `role.guard.ts:5-25`; `admin.service.ts:19-33` | `roleGuard` + `ROLE_ADMIN`. HTTP `admin/*` sem fallback local observado. Layout lê papel de `SessionService` (`mockUserRole`), não do guard. Notificações 100% mock. |
| Painel ONG | `/ong-panel/**` | P1 | `app.routes.ts:198-235`; `ong.service.ts:15-25` | `ROLE_ONG` \| `ROLE_ADMIN`. HTTP `ong/*`. Duplicata de `MeusPets` também em `/ong/pets`. Vários irmãos stub (`ong/dashboard`, `editar-pet`, `doacoes-recebidas`…) **fora** das rotas. |
| Painel petshop | `/petshop-panel/**` | P0 | `app.routes.ts:236-285`; `petshop.service.ts:16-127`; `pedido.service.ts:53-61`, `92-115` | `ROLE_PETSHOP` \| `ROLE_ADMIN`. Produtos/pedidos/chats/cupons: HTTP + mock/`localStorage` no erro. `cadastrar-produto` / `editar-produto` são stubs não roteados. |
| Institucional | `/sobre`, `/faq`, `/contato` | P2 | `app.routes.ts:286-302` | Públicos. `politica-privacidade` e `termos-uso` são stubs **sem rota**. |
| 404 | `**` | P2 | `app.routes.ts:303-307`; `not-found.html:1` | `<p>not-found works!</p>;`. |
| Auth / sessão (transversal) | guards + `AuthService` | P0 | `session.service.ts:16-33`; `auth.service.ts:32-60`, `100-108`; `auth.interceptor.ts:5-24`; `app.config.browser.ts:27-39` | Cookie `mybuddy_session` + Keycloak opcional. `setRole` grava `mockUserRole`. Guard de papel **não** chama `estaLogado()`; lista vazia → redirect `/`. Interceptor usa `document.cookie` sem `isPlatformBrowser`. |
| Catálogo local (serviço) | usado por `/produtos` e painel | P0 | `produto.service.ts:25-56`, `142-150`, `285-291` (573 linhas / 19602 B) | Seed local se catálogo `< 16` ou imagens Unsplash. CRUD inteiro tem `catchError` → `of(mock)`. Spec **afirma** o fallback (`produto.service.spec.ts:78`). |
| Pedidos / cupons (serviço) | checkout + petshop | P0 | `pedido.service.ts:25-38`, `92-106`, `143-160` | `criarPedido` falha → pedido local “sucesso”. Cupons `BUDDY10` / `MEUPET20` hardcoded no fallback. |
| Notificações | layout dos painéis | P1 | `notification.service.ts:19-47` | `effect` recarrega mock por `SessionService.userRole`. Sem HTTP. |
| Carrinho | drawer em `/produtos` | P2 | `cart.service.ts:17-82` | Só `signal` em memória; some no refresh. Sem persistência nem API. |
| Features mortas | (sem rota) | P2 | pastas `features/adotante/**`, `features/public/**`, `marketplace/{carrinho,catalogo,detalhes-pedido}` | HTML `works!`. `app.routes.server.ts` ainda prerenderiza `public/**` e `institucional/**` que as rotas do browser não expõem. |

Severidade neste arquivo = **cheiro observado para grilling**, não prioridade de sprint.

---

## God components (tamanho + cheiro)

Arquivos em `frontend/src/app/features/` (bytes no disco, `wc -l` nas linhas). Specs excluídos.

| Arquivo | Bytes | Linhas | Responsabilidades no mesmo arquivo |
|---------|------:|-------:|-----------------------------------|
| `perfil/perfil.ts` | 23593 | 709 | Abas (8), form, upload, HTTP de perfil, pets/msgs/solicitações estáticos, config por papel |
| `marketplace/marketplace.ts` | 23172 | 819 | Vitrine, busca, filtros, banners, favoritos LS, carrinho, mapeamento de DTO |
| `marketplace/marketplace.scss` | 22376 | — | Folha própria (~vitrine inteira) |
| `marketplace/marketplace.html` | 22000 | 456 | Hero, lojas, ofertas, catálogo, FAB, toast |
| `perfil/perfil.html` | 19337 | 479 | Todas as abas no template |
| `auth/cadastro-escolha-perfil/cadastro-escolha-perfil.html` | 15625 | 358 | Wizard multi-step |
| `auth/cadastro-escolha-perfil/cadastro-escolha-perfil.ts` | 12975 | 445 | Perfil, org, validação, `POST` cadastro |
| `doacoes/pagina-institucional/pagina-institucional.scss` | 12738 | — | Página institucional grande |
| `checkout/checkout.html` | 10893 | 227 | Endereço + pagamento + resumo |
| `landing-page/landing-page.ts` | 8395 | 251 | LP |
| `checkout/checkout.ts` | 6403 | 209 | CEP fake, cupom, multi-loja, pagamento mock |
| `home/home.ts` | 6244 | 200 | HTTP pets + blocos estáticos + Leaflet |

Fora de `features/`, o serviço mais inchado é `core/services/produto.service.ts` (19602 B, 573 linhas) — HTTP + motor de mock local.

---

## Auth: HTTP real vs mock no caminho

| Peça | O que faz | Evidência |
|------|-----------|-----------|
| Login | `POST` token Keycloak (`grant_type=password`), cookie `mybuddy_session`, depois `GET usuarios/meu-perfil` | `auth.service.ts:67-92`, `100-108`; `login.ts:91-96` |
| Keycloak JS | `provideKeycloak` + `check-sso` só no browser config | `app.config.browser.ts:27-39` |
| `estaLogado()` | token em memória **ou** `keycloak.authenticated` | `auth.service.ts:48-50` |
| `authGuard` | redireciona a `/auth/login` se não logado | `auth.guards.ts:5-15` |
| `roleGuard` | compara `route.data.roles` com `obterPapeisUsuario()`; **não** exige login explícito | `role.guard.ts:5-25` |
| Papel na UI | `localStorage['mockUserRole']` no boot; `AuthService.obterPerfil` chama `sessionService.setRole` | `session.service.ts:16-33`; `auth.service.ts:106-108` |
| Painéis | nome do papel e notificações vêm de `SessionService`, não do JWT na hora do render | `dashboard-layout.ts:24-41`, `31-32` |
| Interceptor | Bearer se URL contém `/api/` | `auth.interceptor.ts:15-21` |

Não há `if (dev) skipGuard` encontrado. O risco é **chave de mock no storage + fallback silencioso**, não um bypass explícito no guard.

---

## API vs fake (fluxos críticos)

| Fluxo | Serviço / tela | HTTP? | Fallback silencioso? |
|-------|----------------|-------|----------------------|
| Login | `AuthService.loginComCredenciais` | Sim, Keycloak | Não (`throwError`) |
| Cadastro | `AuthService.registrar` | Sim, `auth/cadastro` | Não (erro na UI) |
| Pets / adoção (lista) | `PetService.buscarTodos` | Sim | Não no serviço; a **tela** some com o erro |
| Pets recentes (home) | `PetService.buscarRecentes` | Sim | Sim — 3 pets placeholder |
| Marketplace | `ProdutoService.buscarComFiltros` | Tenta | Sim — `mybuddy_produtos_local` |
| Carrinho | `CartService` | Não | N/A (memória) |
| Checkout pedido | `PedidoService.criarPedido` | Tenta | Sim — pedido local; checkout ainda força sucesso |
| Cupom | `PedidoService.validarCupom` | Tenta | Sim — `BUDDY10` / `MEUPET20` |
| Perfil (ficha) | `UserService.buscarPerfil` | Sim | Não observado no serviço |
| Perfil (pets/msgs) | signals em `perfil.ts` | Não | Dados de demonstração |
| Doações (lista/stats) | `DonationService.get*` | Tenta | Sim — campanhas/ONGs mock |
| Doações (pagar) | `DonationService.create*` / `PaymentService` | Sim | Não no create |
| Eventos / serviços | componente | Não | Catálogo hardcoded |
| Notificações | `NotificationService` | Não | Mock por papel |

---

## Shared UI / design

- Tokens globais em `frontend/src/styles.scss` (`:root`) + `_grid` / `_mixins` / `_forms` + preset PrimeNG Aura (`styles/mypreset.ts`, `app.config.ts:52-59`).
- PrimeNG importado em 3 sítios de UI: `login.ts` (`AutoComplete`), `shared/components/modal` (`p-dialog` / `p-button`), `styleguide`. Telas-deus **não** usam `p-table` / `p-dataview`.
- Cada feature grande tem SCSS próprio (marketplace 22 KB, home 14 KB, LP 13 KB, doações 12 KB, perfil 11 KB) — segundo sistema visual na prática.
- `/style-guide` é rota pública de produção (e prerender).
- Ícones: Material Icons + Font Awesome no `styles.scss` global.

---

## Testes

Referência histórica: [`docs/tecnico/auditoria-cobertura-frontend.md`](../../tecnico/auditoria-cobertura-frontend.md) (2026-06-26, MYB-206). Naquela data: ~17% das unidades com spec; meta MYB-208 (50% linhas) pendente. **O arquivo está desatualizado** em relação ao tree atual (hoje há specs extras em `cart`, `loading`, `session`, `notification`, `api`, `produto`, e vários shared).

**Agora (contagem de arquivos, 2026-09-10):**

| Recorte | Specs | Observação |
|---------|------:|------------|
| `features/**` | 1 | Só `styleguide.spec.ts`. Sem spec de marketplace, perfil, cadastro, checkout, login, pets, home, doações, painéis. |
| `core/services` | 8 / ~17 serviços | Há spec de `produto` **que documenta o fallback local como comportamento**. Sem spec de `auth.service`, `pedido`, `donation`, `petshop`, guards. |
| `shared/components` | 12 / 25 | Cards/header/footer/modal/paginator. Sem spec de `cart-drawer`, `hero-section`, `empty-state`, `not-found`. |
| Playwright | 2 arquivos | `e2e/login.spec.ts` (token/perfil mockados na rota). `e2e/checkout.spec.ts` cobre **`/checkout/pagamento`**, não o checkout de marketplace. |
| CI | — | `.github/workflows/ci.yml` job `e2e-frontend`: Playwright only. `npm test` (Vitest) **não** entra no CI. |

Buracos óbvios vs telas grandes: `marketplace.ts` 819 linhas, `perfil.ts` 709, `cadastro-escolha-perfil.ts` 445, `checkout.ts` 209 — zero Vitest.

---

## Candidatos naturais para **primeira fatia de grilling**

Inventário apenas. O time escolhe **uma** superfície; grilling decide apetite / no-gos ([`.ai/skills/grilling.md`](../../../.ai/skills/grilling.md)). Não há ordem de sprint aqui.

1. **Vitrine marketplace (`/produtos`)** — uma tela, um serviço. God component + `ProdutoService` com `mybuddy_produtos_local` e `catchError` → mock. Grilling pode perguntar: API já lista produtos? O fallback é escopo ou armadilha? Não precisa abrir painel petshop nem checkout no mesmo apetite.
2. **Perfil (`/perfil`)** — `authGuard` já existe; ficha HTTP vs abas de pets/mensagens/solicitações estáticas. Fatia possível: “uma aba, dados da API ou empty/error visível”, sem redesenhar o design system.
3. **Checkout marketplace (`/checkout`)** — sem sessão, pagamento simulado, `PedidoService` que “cria” pedido local e a tela que ignora erro. Fatia estreita: ou auth no caminho, ou falha visível (não os dois + Mercado Pago + multi-loja).

Outras superfícies (cadastro wizard, doações, `/ong/pets` sem `roleGuard`) cabem em grilling **depois**, não nesta lista de 2–3.

---

## Fora de escopo / não olhado

- **Mobile Flutter** (`mobile/`) — não aberto.
- **Backend Java** (`backend/`) — contratos, segurança, Flyway, JaCoCo: não auditados. Não se afirma se os endpoints `produtos` / `pedidos` / `campanhas` existem ou estão corretos.
- Passe visual Impeccable (a11y real, contraste, teclado, responsivo no browser).
- SSR/hidratação além da leitura de `app.routes.server.ts` e do interceptor/`document`.
- Keycloak realm Docker, Mercado Pago sandbox, Caddy.
- Qualidade dos specs shared existentes (só presença/ausência).
- `features/admin/**` tela a tela (só rotas + `AdminService` HTTP).
- Performance de bundle / Core Web Vitals.

---

## Próximo passo (processo)

Este arquivo **não** autoriza código. Loop: grilling desta (ou outra) superfície → Shape + SDD em `docs/sdd/frontend/` → runbook em `docs/sprints/frontend/` → então PR de produto.
