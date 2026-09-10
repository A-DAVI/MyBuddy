# CONVENTIONS.md — FAÇA / NÃO FAÇA (MyBuddy)

Regras mecânicas. Filosofia e Git humano: [`CONTRIBUTING.md`](../CONTRIBUTING.md).  
Contexto para agentes: [`AGENTS.md`](../AGENTS.md).

## Commits e módulos (allowlist)

Formato: `<tipo>(escopo): <descrição curta>  [MY-XXX]` ou `[MYB-XXX]`.

Tipos: `feat` | `fix` | `hotfix` | `refactor` | `chore` | `docs` | `test` | `style` | `perf` | `ci` | `revert`.

**Escopos permitidos:**

`backend` | `frontend` | `mobile` | `infra` | `docs` | `sdd` | `ci` | `auth` | `db` | `test`

- Um commit = uma mudança lógica.
- Imperativo no presente, sem ponto final na primeira linha.
- Sem dump de formatação misturado com lógica.
- Referência Jira obrigatória em código de produto (`[MY-XXX]` / `[MYB-XXX]`). Docs/processo puro podem usar `docs(sdd):` / `docs:` sem card se o time acordar no PR — preferir card quando existir.

Detalhes e exemplos: [`.github/COMMIT_CONVENTION.md`](../.github/COMMIT_CONVENTION.md).

Branches: `feat|fix|hotfix|docs|chore|test|ci/MY-XXX-kebab` a partir de `Developer` (produto).

---

## Backend (Java 21 / Spring Boot)

### FAÇA

- Manter o estilo do pacote `com.Mybuddy.Myb` (Controller / Service / Repository / Model / DTO / Security).
- Novas tabelas/índices Postgres via **Flyway** (`db/migration/V*`), nunca `ddl-auto=update` no que o time compartilha.
- Mongo para adoção/usuários/eventos/serviços; JPA/Postgres para marketplace (produtos, pedidos, pagamentos).
- Validar input (`jakarta.validation`), erros previsíveis, sem engolir exceção.
- Testes JUnit 5 + Spring Boot Test; Testcontainers quando o teste precisa de Mongo/Postgres/Keycloak de verdade.
- Manter o quality gate do JaCoCo (CI: cobertura de linha ≥ 61%).
- Resource server + JWT do Keycloak; não reinventar login “simples” no controller.

### NÃO FAÇA

- Reescrever o backend “porque o frontend está bagunçado”. Backend é o lado mais confiável neste ciclo.
- Drive-by: não refatorar `Security`, `AuthService` ou todos os `*Service` num PR de tela.
- Hardcode de secret, token Mercado Pago, senha de banco ou admin Keycloak.
- `System.out` / logs com PII (e-mail, CPF, token, dados de pagamento).
- Mudar ADR-001 (Keycloak) ou o split Mongo/Postgres sem ADR novo.
- Apagar exclusões de teste do `ci.yml` sem runbook de CI.

---

## Frontend (Angular 21)

### FAÇA

- Features em `frontend/src/app/features/`; reuso em `shared/`; HTTP/auth/guards em `core/`.
- Keycloak / `AuthService` como caminho real de sessão. Papel vem do token/perfil, não de `localStorage` de mock.
- Catálogo, pets, pedidos, doações: **API primeiro**. Empty/error state quando a API falha.
- Componentes pequenos; extrair filtros, listas e drawers das telas-deus (`marketplace.ts`, `perfil.ts`, cadastro).
- Shape brief se a mudança cria ou altera superfície visível ([`docs/sdd/frontend/README.md`](sdd/frontend/README.md)).
- Vitest para unidade; Playwright para fluxo crítico que o PR toca.
- Prettier/ESLint só nos arquivos da mudança (`frontend/.prettierrc`, `printWidth` 150).

### NÃO FAÇA

- God component (fetch + mock + layout + pagamento no mesmo TS).
- `mockUserRole` / auth fake em rota que o usuário final usa.
- `localStorage` (`mybuddy_produtos_local`, pedidos, cupons) como fonte de verdade do catálogo.
- `catchError` que devolve mock e finge sucesso.
- Commit único “aplica Prettier no frontend inteiro”.
- Trocar Angular por outro framework (ADR-002).
- Misturar rascunho de marketplace com dados reais sem deixar o estado explícito (dev-only, flag, nunca silencioso).

---

## Mobile (Flutter)

### FAÇA

- Seguir a estrutura já usada (BLoC, Dio, go_router, get_it).
- Tokens em storage seguro (`flutter_secure_storage`), não em texto solto.
- `flutter test` / `flutter analyze` no recorte alterado.

### NÃO FAÇA

- Duplicar regras de negócio que o backend já garante (preço, estoque, papel).
- Fixtures com dados de tutores/ONGs reais.

---

## Infra / auth / CI

### FAÇA

- Compose como fonte de verdade de serviços (ADR-003); Caddy na borda (ADR-005).
- Realm e clients Keycloak documentados; alterações de realm-export conscientes (impacto em todo o time).
- Segredos só em env/secrets do host — `.env.example` sem valores reais de produção.

### NÃO FAÇA

- Workflows `anthropics/claude-code-action`, `claude.yml`, `claude-code-review.yml`, cron de auditoria/resumo, labels que assumem Claude Code Action.
- Esvaziar `.github/workflows/ci.yml`.
- Expor portas internas em produção; o proxy é o Caddy.

---

## Dados sensíveis e fixtures

**Não commitar:** `.env`, `JWT_SECRET`, `KC_CLIENT_SECRET`, tokens Mercado Pago, chaves SSH, dumps de produção, export de usuários reais.

**Fixtures / seeds / Playwright:** nomes, e-mails e pets **sintéticos** (`ada@example.test`). Sem CPF, cartão, endereço real, foto de menor identificável, conversas reais de chat.

Uploads: não versionar `backend/uploads` com conteúdo de usuário.

---

## SDD, sprint e UI

- Mudança de comportamento ou de superfície: SDD em `docs/sdd/<modulo>/` + runbook em `docs/sprints/<modulo>/`.
- Sem SDD → sem runbook; sem runbook → sem sprint de implementação.
- UI: Shape brief (problema, apetite, esboço, rabbit holes, no-gos) **antes** do código.
- Inventário Impeccable não substitui grilling nem SDD.
