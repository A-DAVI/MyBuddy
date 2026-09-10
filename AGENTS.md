# AGENTS.md — contexto versionado do MyBuddy

Este arquivo é a memória operacional para humanos e agentes de código.  
Não reabra decisões de arquitetura “porque o modelo preferiria outra stack”.  
Não reescreva produto neste repositório sem passar pelo loop abaixo.

## O loop (obrigatório)

1. **Inventário** — auditoria Impeccable (`/audit` ou equivalente) da superfície. É lista, não decisão.
2. **Decisão** — grilling: stress-test do plano (apetite, buracos, no-gos). Ver [`.ai/skills/grilling.md`](.ai/skills/grilling.md).
3. **SDD** — spec em [`docs/sdd/`](docs/sdd/README.md). Sem SDD não há runbook.
4. **Runbook** — sprint em [`docs/sprints/`](docs/sprints/README.md). Sem runbook não há sprint.
5. **Código + testes** — só depois dos quatro itens acima, no escopo do runbook.

Regra de ouro: **inventário via Impeccable; decisão via grilling; implementação só após SDD + runbook.**

Regra de sprint: **não há sprint sem runbook; não há runbook sem SDD.**

UI (criar ou editar superfície): **Shape brief obrigatório** (problema, apetite, esboço, rabbit holes, no-gos). Ver [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md) e [`docs/sdd/frontend/README.md`](docs/sdd/frontend/README.md).

Este PR de scaffolding **não** inventa SDD de rewrite do frontend. O método fica pronto; o rewrite se planeja com calma, superfície por superfície.

---

## Visão do produto

MyBuddy é o hub do ecossistema pet local: **adoção**, **eventos/feiras**, **guia de serviços**, **marketplace**, **doações**. Conecta adotantes, ONGs, protetores, pet shops e clínicas.

Não é um sistema fiscal. Linguagem, entidades e analogias devem permanecer no domínio pet (pets, ONGs, pedidos, Keycloak, Mercado Pago) — não importar vocabulário de NFe/CFOP/tributos de outros repositórios.

---

## Stack (fato, não proposta)

| Camada | Tecnologia |
|--------|------------|
| Backend | Java 21, Spring Boot 3.5, Maven (`backend/`) |
| API | REST, OAuth2 Resource Server, SpringDoc OpenAPI |
| Auth | Keycloak 26 (realm `mybuddy`), `keycloak-angular` / `keycloak-js` |
| Dados | MongoDB 7 (adoção, usuários, eventos, serviços); PostgreSQL 16 (marketplace, pedidos, pagamentos); Redis 7 (cache) |
| Migrations | Flyway em `backend/src/main/resources/db/migration/` (+ Java migrations em `backend/src/main/java/db/migration/`) |
| Pagamentos | Mercado Pago (Checkout Pro, webhooks HMAC) |
| Frontend | Angular 21 + TypeScript, SSR, PrimeNG, Vitest, Playwright, ESLint/Prettier (`frontend/`) |
| Mobile | Flutter 3.41+ / Dart, BLoC, Dio, go_router (`mobile/`) |
| Infra | Docker Compose, Caddy reverse proxy, GitHub Actions (`.github/workflows/ci.yml`) |

Comandos do dia a dia: [`CLAUDE.md`](CLAUDE.md).  
Setup humano: [`CONTRIBUTING.md`](CONTRIBUTING.md).  
Regras mecânicas: [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md).

---

## Decisões que NÃO se reabrem de leve

Estão em [`docs/tecnico/ADR/`](docs/tecnico/ADR/). Novo ADR só com contexto + alternativas; não “trocar porque o agente gosta mais”.

| ADR | Decisão |
|-----|---------|
| [ADR-001](docs/tecnico/ADR/ADR-001-keycloak-como-idp.md) | Keycloak como IdP; backend é resource server |
| [ADR-002](docs/tecnico/ADR/ADR-002-angular-como-framework-web.md) | Angular (não React) no web |
| [ADR-003](docs/tecnico/ADR/ADR-003-containerizacao-com-docker.md) | Docker Compose como ambiente canônico |
| [ADR-004](docs/tecnico/ADR/ADR-004-mercadopago-como-gateway-pagamento.md) | Mercado Pago Checkout Pro + webhooks |
| [ADR-005](docs/tecnico/ADR/ADR-005-caddy-reverse-proxy.md) | Caddy na frente (não Nginx como proxy externo) |

Também estáveis neste ciclo:

- **Backend é comparativamente confiável.** Não prescrever rewrite de Java/Spring “de passagem” em PRs de frontend ou docs.
- **Mongo vs Postgres:** Mongo para o núcleo de adoção/usuários/eventos/serviços; Postgres para marketplace relacional. Não unificar os dois “para simplificar”.
- **CI existente permanece.** Não esvaziar `.github/workflows/ci.yml` nem acrescentar workflows que assumam Claude Code Action (`anthropics/claude-code-action`, menções `@claude`, labels `claude`, crons de auditoria/resumo).
- **Git:** `main` (produção), `Developer` (integração), `feature/*` \| `fix/*` \| `hotfix/*`. Conventional Commits com `[MY-XXX]` ou `[MYB-XXX]`.

---

## Onde está a dívida (e o que NÃO fazer)

Foco ativo: **frontend Angular** — overhauls guiados por IA, telas-deus, mocks/`localStorage` misturados com API. Ver [`docs/ROADMAP.md`](docs/ROADMAP.md).

Exemplos atuais (inventário, não ordem de implementação):

- Telas grandes: `frontend/src/app/features/marketplace/marketplace.ts`, `perfil/perfil.ts`, `auth/cadastro-escolha-perfil/`.
- Papel mock em `SessionService` (`localStorage` key `mockUserRole`).
- Catálogo/pedidos com fallback para `localStorage` / mocks em `ProdutoService`, `PedidoService`, `PetshopService`, `DonationService`.

**Não** transformar esse inventário num SDD gigante de rewrite neste scaffolding. Grilling + SDD por superfície, com apetite limitado.

Backend: manter, endurecer testes, corrigir bugs. Sem “aproveitar e refatorar o pacote Service inteiro”.

---

## Anti-padrões (agentes e PRs)

### Frontend

- **God components** — telas de centenas de linhas com filtro, fetch, mock, layout e pagamento no mesmo arquivo. Extrair; não inchá-las.
- **Auth mock em caminho de produção** — `mockUserRole` / bypass de Keycloak em rotas reais. Mock só em teste.
- **`localStorage` como fonte de verdade do catálogo** — produtos, pedidos, cupons e pets vêm da API. Storage no browser é cache/UX (rascunho, preferências), não o banco.
- **Silent fallback para mock** — `catchError(() => of(mock))` esconde API quebrada. Falha deve ser visível (empty/error state), não um catálogo de mentira.
- **Dump Prettier em mega-commit** — formatar só os arquivos da mudança. `printWidth: 150` em `frontend/.prettierrc`; não reformatar o monorepo “para ficar bonito”.
- **Misturar Feature com Shared** — cards/reuso em `shared/`; regras de vitrine/checkout em `features/`.

### Backend / cross-cutting

- **Drive-by backend** — PR de UI não “limpa” controllers, security ou Flyway sem SDD.
- **Reabrir ADR** no mesmo PR que implementa tela.
- **PII em fixture** — sem CPF, e-mail real, telefone, cartão, token Mercado Pago, senha Keycloak. Dados sintéticos.
- **Segredos no git** — `.env`, tokens, `KC_CLIENT_SECRET`, `JWT_SECRET`, chaves SSH. Usar `.env.example`.
- **CI paralelo** — não adicionar `claude.yml`, `claude-code-review.yml`, cron de auditoria ou resumo diário baseado em Action de terceiro.

### Processo

- Implementar sem SDD + runbook quando o escopo muda comportamento ou superfície.
- Tratar output do Impeccable como backlog já priorizado — **é inventário**; grilling decide.
- Copiar domínio/docs de outros produtos (tributário, outro SaaS) para cá.

---

## Onde mora a documentação

| Caminho | Função |
|---------|--------|
| [`README.md`](README.md) | Produto + setup Docker |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Filosofia + Git + setup local |
| [`CLAUDE.md`](CLAUDE.md) | Cheatsheet de comandos |
| [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md) | FAÇA / NÃO FAÇA mecânico |
| [`docs/ROADMAP.md`](docs/ROADMAP.md) | Foco do ciclo (dívida Angular) |
| [`docs/sdd/`](docs/sdd/README.md) | Specs por módulo |
| [`docs/sprints/`](docs/sprints/README.md) | Runbooks por módulo |
| [`docs/tecnico/`](docs/tecnico/) | ADRs, análises, cobertura, produção |
| [`docs/academico/`](docs/academico/) | Material acadêmico (não apagar) |
| [`docs/PRODUCTION.md`](docs/PRODUCTION.md) | Operação do ambiente de produção |
| [`docs/tecnico/auditoria-cobertura-frontend.md`](docs/tecnico/auditoria-cobertura-frontend.md) | Cobertura Vitest (histórica) |
| [`.ai/`](.ai/README.md) | Skills e hooks (agnósticos de ferramenta) |
| [`.github/COMMIT_CONVENTION.md`](.github/COMMIT_CONVENTION.md) | Conventional Commits + Jira |

`docs/academico` e `docs/tecnico` **permanecem**. Docs acadêmicos podem estar datados (HTML legado, MySQL); a stack vigente é a deste arquivo e dos ADRs.

---

## Impeccable + grilling (encaixe)

- **Impeccable** avalia UI (a11y, hierarquia, estados, consistência). Skill fina: [`.ai/skills/impeccable-audit.md`](.ai/skills/impeccable-audit.md). **Não** vendorar a árvore inteira do Impeccable neste repo.
- Output do audit = **inventário** (o que existe, o que dói, evidência).
- **Grilling** = sessão humana/IDE que decide o que entra no SDD. Não chama Action, não usa label `claude`, não menciona `@claude` em issue/PR.
- Depois: SDD → runbook → código. Skills: [`.ai/skills/`](.ai/skills/).

---

## Como contribuir código (resumo)

Detalhe em [`CONTRIBUTING.md`](CONTRIBUTING.md) e [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md).

- Branches a partir de `Developer` para produto; Conventional Commits.
- PR com checklist de SDD/runbook se o escopo muda; Shape se UI; testes/lint; zero segredos.
- Qualidade: backend `./mvnw test` / `verify` (JaCoCo linha ≥ 61% no CI); frontend `npm test`, `npm run lint`, `npm run e2e` quando a superfície pede; mobile `flutter test`.
