# Runbook: Honestidade da vitrine `/produtos`

- **Status:** `planned`
- **SDD:** [`docs/sdd/frontend/SDD - Honestidade da Vitrine Produtos.md`](../../sdd/frontend/SDD%20-%20Honestidade%20da%20Vitrine%20Produtos.md)
- **Inventário:** [`inventory-impeccable-2026-09-10.md`](inventory-impeccable-2026-09-10.md) — em `main` via [PR #2](https://github.com/A-DAVI/MyBuddy/pull/2)
- **Apetite / fatia:** uma superfície (vitrine). Matar silent success nos **reads** de `ProdutoService` + estados loading / empty / error em `/produtos` (PDP só se trivial). Sem redesenho.
- **Branch sugerida (implementação):** a partir de `Developer`, `feat/MYB-XXX-honestidade-vitrine-produtos` (card Jira quando existir). **Este** PR de docs não implementa.
- **Módulo:** frontend Angular. **Não tocar** `backend/`, `mobile/`, checkout, Mercado Pago.

---

## Pré-condições

- [x] Inventário Impeccable da vitrine (P0 `/produtos` e `/produtos/:id`).
- [x] Grilling com decisão **seguir** (não relitigar).
- [x] SDD + Shape no arquivo acima.
- [ ] Card Jira `[MY-XXX]` / `[MYB-XXX]` no PR de **código** (docs de processo podem seguir sem card).
- [ ] Branch de produto a partir de `Developer`, não deste PR de spec.

Ler antes de codear, nesta ordem:

1. SDD → [Contexto](../../sdd/frontend/SDD%20-%20Honestidade%20da%20Vitrine%20Produtos.md#contexto)
2. SDD → [Shape / UX](../../sdd/frontend/SDD%20-%20Honestidade%20da%20Vitrine%20Produtos.md#shape--ux) (estados e layout)
3. SDD → [Não-objetivos](../../sdd/frontend/SDD%20-%20Honestidade%20da%20Vitrine%20Produtos.md#não-objetivos) e [No-gos](../../sdd/frontend/SDD%20-%20Honestidade%20da%20Vitrine%20Produtos.md#no-gos-hard)
4. SDD → [Comportamento desejado](../../sdd/frontend/SDD%20-%20Honestidade%20da%20Vitrine%20Produtos.md#comportamento-desejado) e [Contratos](../../sdd/frontend/SDD%20-%20Honestidade%20da%20Vitrine%20Produtos.md#contratos-de-api-já-existentes--sem-sdd-java)
5. SDD → [Critérios de aceite](../../sdd/frontend/SDD%20-%20Honestidade%20da%20Vitrine%20Produtos.md#critérios-de-aceite-testáveis) (AC-1 … AC-12)
6. Skill TDD: [`.ai/skills/tdd.md`](../../../.ai/skills/tdd.md)

---

## Passos (ordem)

Não pular o passo 1. Não “já ir limpando” o painel petshop, o checkout ou o god component inteiro.

### 1. Vitest em vermelho — reads honesto (SDD: Plano de testes, AC-1–4, AC-10)

Arquivo: `frontend/src/app/core/services/produto.service.spec.ts`

- Inverter/apagar os casos que **celebram** fallback:
  - `buscarComFiltros` → “should fall back to localStorage on API error”
  - filtros “on fallback” (`busca`, `precoMax`) **se** só existem para o mock
  - `buscarPorId` → “should fall back to localStorage product on API error”
  - `buscarCategorias` → “should return mock categories on API error”
- Substituir por: HTTP `throwError` → subscriber `error`; **não** emitir lista/objeto mock.
- Manter os testes de 200 / query params / `content`.
- **Não** reescrever a suíte de `criar` / `atualizar` / `deletar` / `avaliarProduto` nesta fatia.

Rodar (deve falhar enquanto o `catchError` existir):

```bash
cd frontend && npx vitest run src/app/core/services/produto.service.spec.ts
```

### 2. Serviço — matar silent success nos reads (SDD: Comportamento desejado / Serviço)

Arquivo: `frontend/src/app/core/services/produto.service.ts`

Mínimo:

- `buscarComFiltros` (~L45–56): remover `catchError` que faz `of(this.obterProdutosLocaisFiltrados(filtros))`. Manter `map` de `res.content`.
- `buscarPorId` (~L59–67): remover `catchError` para mock local. Deixar o erro HTTP subir.
- `buscarCategorias` (~L100–112): remover `catchError` das 5 categorias fake.

Não usar `mybuddy_produtos_local` nesses três métodos.

Opcional barato: não chamar `inicializarProdutosLocais()` no construtor se isso não quebrar os writes ainda no escopo do painel. **Não** apagar helpers de write “por higiene”.

Grep no PR: `mybuddy_produtos_local`, `obterProdutosLocais`, `catchError` neste arquivo — reads limpos.

Vitest do passo 1 deve ficar verde.

### 3. Flags de estado na lista (SDD: Shape estados, AC-5–7, AC-9)

Arquivos: `frontend/src/app/features/marketplace/marketplace.ts`, `marketplace.html`  
Opcional: `marketplace.scss` só se uma classe mínima for necessária (reusar `.no-results` / `app-empty-state` / spinner já visto em `.carregando` da PDP).

- Em `carregarProdutos()` (~L194–223): `loading` ao entrar; limpar `error`; `next` → dados + `loading=false`; `error` → mensagem + `loading=false` (**não** só `console.error`).
- Template: no **miolo das grades** (não no hero):
  - loading visível
  - empty quando a API (ou o filtro) devolver zero
  - error + controle de retry (chama `carregarProdutos` de novo)
- **Não** redesenhar hero, busca, carrossel, banners, cards, drawer.
- Extração opcional: só a carga da lista para um helper/método claro. **Proibido** split do god component (banner, toast, FAB, favoritos).

Carrossel `categorias` hardcoded (~L264): **não** religar a `buscarCategorias` neste runbook.

### 4. Detalhe — mínimo honesto (SDD: Shape detalhe, AC-8)

Arquivos: `frontend/src/app/features/marketplace/detalhes-produto/detalhes-produto.ts` (+ html só se trivial)

- Loading `.carregando` já existe — manter.
- Após o passo 2, falha de API não mostra seed. Completar empty/error **somente** se for poucas linhas (texto no slot `.carregando`, sem nova PDP).
- Evitar `navigate(["/produtos"])` que esconde erro de rede; 404→lista é aceitável se documentado no PR.
- **Não** mexer em `carregarRecomendados` hardcoded.

Se o passo 4 inflar, **parar** e deixar só o serviço + lista. A PDP honesta pelo serviço já cumpre o grilling (“UI no detalhe só se trivial”).

### 5. Playwright opcional (SDD: Plano de testes)

Só se AC-5–7 não ficarem óbvios no Vitest/manual:

- `frontend/e2e/produtos.spec.ts` com `page.route('**/api/produtos*')` para 200 vazio, 500 e 1 item sintético.
- Sem PII. Sem Keycloak. Sem depender de `localStorage`.

```bash
cd frontend && npm run e2e -- e2e/produtos.spec.ts
```

### 6. Verificação do recorte

```bash
cd frontend
npx vitest run src/app/core/services/produto.service.spec.ts
npx eslint src/app/core/services/produto.service.ts src/app/core/services/produto.service.spec.ts src/app/features/marketplace/marketplace.ts src/app/features/marketplace/marketplace.html
# + detalhe se o passo 4 tiver diff
```

Prettier **somente** nos arquivos tocados (`printWidth` 150). Nunca no tree.

Manual: SDD → Plano de testes → Manual.

### 7. O que **não** tocar

| Caminho / tema | Por quê |
|----------------|---------|
| `backend/**` | Sem SDD Java; `ProdutoController` assume-se válido |
| `frontend/src/app/features/checkout/**` | Fora do apetite |
| `cart.service.ts`, Mercado Pago | Fora |
| `petshop/meus-produtos` CRUD | Fora (efeito colateral de erro visível é OK) |
| Writes em `produto.service.ts` (`criar`/`atualizar`/`deletar`/`avaliarProduto`) | Painel, não vitrine |
| Split completo de `marketplace.ts` | Rabbit hole |
| `session.service.ts` / `mockUserRole` | Outra superfície |
| `.github/workflows/ci.yml` | Não esvaziar / não adicionar Claude Action |
| Prettier mega-diff | No-go |

Se o contrato HTTP estiver **provado** quebrado (shape, 401 em GET público, 500 estável), **parar** e abrir SDD em `docs/sdd/backend/` — não “consertar Java de passagem”.

---

## Verificação (fechamento da fatia)

Comandos:

- [ ] `npx vitest run src/app/core/services/produto.service.spec.ts` verde
- [ ] Nenhum teste de read afirma fallback LS/mock
- [ ] Lint/Prettier só no recorte
- [ ] Playwright smoke **ou** checklist manual dos quatro cenários (API ok / vazio / down / PDP)

Manual:

- [ ] `/produtos` + API ok → layout atual, dados reais
- [ ] `/produtos` + `content: []` → empty, sem seed
- [ ] `/produtos` + API down → error + retry, sem 16 cards Premier/Bravecto
- [ ] `/produtos/:id` → loading; 404/rede sem PDP seed

Diff:

- [ ] AC-12: sem `backend/`, sem checkout, sem dump de formatação
- [ ] Grep: reads sem `catchError` → `of(local)`

---

## Checklist de fechamento

- [ ] AC-1 … AC-12 do SDD atendidos (ou desvio registrado e re-grillado — não “só mais um arquivo”)
- [ ] Shape: loading / success / empty / error na lista; layout preservado
- [ ] SDD + este runbook linkados no PR de implementação (template: Processo SDD/runbook/Shape)
- [ ] Conventional Commit `fix(frontend):` ou `feat(frontend):` + `[MYB-XXX]` no PR de **código**
- [ ] Sem segredos / PII
- [ ] Handoff preenchido se a sessão parar no meio ([`.ai/skills/handoff.md`](../../../.ai/skills/handoff.md))

---

## Rollback

Reverter o PR de implementação. O silent success volta com o `catchError`; não há migration nem feature flag. Este runbook/SDD permanece — o código é que desfaz.

---

## Fora desta fatia

Checkout, carrinho persistente, CRUD petshop, Mercado Pago, rewrite Java, split do god component, religar carrossel a `GET categorias`, paginação `lastId`, auth na vitrine, Prettier global.

Próximas superfícies (inventário, **não** este runbook): perfil `/perfil`, checkout `/checkout`.

---

## Handoff (quando a implementação começar)

- Fatia: honestidade da vitrine `/produtos`
- SDD / runbook: caminhos no topo deste arquivo
- Branch de spec: `cursor/sdd-honestidade-vitrine-produtos-f722` (docs only)
- Próximo passo concreto: passo 1 (Vitest em vermelho) numa branch `feat/` a partir de `Developer`
- Não fazer: relitigar o grilling; tocar backend; redesenhar a vitrine
