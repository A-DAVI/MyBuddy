# CLAUDE.md — cheatsheet operacional (MyBuddy)

Agnóstico de ferramenta: vale no terminal, IDE ou qualquer agente.  
Contexto de produto e processo: [`AGENTS.md`](AGENTS.md).  
Regras mecânicas: [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md).

## Pré-requisitos

- Docker + Docker Compose
- Java 21
- Node.js 22+ (CI do frontend hoje usa Node 20 — preferir 22+ no local)
- Maven Wrapper (`backend/mvnw`; também há `./mvnw` na raiz)
- Angular CLI via `npx ng` / scripts npm
- Flutter 3.41+ (só para `mobile/`)

## Variáveis de ambiente

```bash
cp .env.example .env
# Nunca commitar .env, tokens Mercado Pago, JWT_SECRET, KC_CLIENT_SECRET
```

## Docker Compose

Infra só (dev local com backend/frontend na máquina):

```bash
docker compose up -d postgres mongodb keycloak redis
docker compose ps
```

Stack completa (Caddy na 80/443, frontend+backend em container):

```bash
docker compose up --build
# override de portas em docker-compose.override.yml (dev)
```

URLs típicas (compose + override):

| Serviço | URL |
|---------|-----|
| Frontend (Caddy / nginx do front) | http://localhost |
| Frontend (ng serve) | http://localhost:4200 |
| Backend API | http://localhost:8081 |
| Health | http://localhost:8081/actuator/health |
| Keycloak | http://localhost:8080 |
| Postgres | localhost:5432 (`mybuddy` / `mybuddy`) |
| MongoDB | localhost:27017 |
| Redis | localhost:6379 |

```bash
docker compose logs -f backend
docker logs mybuddy-backend --tail 50
docker compose down          # para
docker compose down -v       # reset de volumes
```

Realm `mybuddy` importa de `docker/keycloak/realm-export.json`.  
Banco `keycloak` nasce via `docker/init-db.sql`.

## Backend (Java 21 / Spring Boot)

Porta `8081` (`backend/src/main/resources/application.properties`).  
Perfil Docker: `SPRING_PROFILES_ACTIVE=docker` + `application-docker.properties`.

```bash
cd backend
./mvnw spring-boot:run
./mvnw test
./mvnw verify   # inclui JaCoCo check (mín. 61% linha no CI)
```

O CI exclui alguns testes de container/Keycloak — não apagar essa exclusão sem SDD de CI.

Flyway: `backend/src/main/resources/db/migration/` e `backend/src/main/java/db/migration/`.  
`ddl-auto=validate` no perfil local; não ligar `create-drop` em caminho de time.

## Frontend (Angular 21)

```bash
cd frontend
npm install
npm start              # ng serve
npm test               # Vitest via Angular unit-test builder
npm run test:coverage
npm run lint
npm run e2e            # Playwright
npm run e2e:ui
npm run build
```

Formatar **somente** os arquivos da mudança (Prettier `printWidth` 150). Não gerar commit de reformat do tree inteiro.

## Mobile (Flutter)

```bash
cd mobile
flutter pub get
flutter analyze
flutter test
flutter run
```

## Testes — o que o CI já faz

`.github/workflows/ci.yml` (não desmontar):

1. Backend `mvn verify` com Mongo de serviço + JaCoCo ≥ 61%.
2. Frontend Playwright (`npm run e2e`) com Chromium.

Rodar o recorte local equivalente **antes** do PR.

## Git (atalho)

```bash
git checkout Developer
git pull origin Developer
git checkout -b feat/MYB-XXX-descricao-curta
```

Commit: [`CONTRIBUTING.md`](CONTRIBUTING.md) e [`.github/COMMIT_CONVENTION.md`](.github/COMMIT_CONVENTION.md).

```
<tipo>(escopo): <descrição>  [MY-XXX]
<tipo>(escopo): <descrição>  [MYB-XXX]
```

Escopos: ver allowlist em [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md).

## Processo (não pular)

Inventário Impeccable → grilling → SDD → runbook → código.  
Skills: [`.ai/skills/`](.ai/skills/). Hooks: [`.ai/hooks.md`](.ai/hooks.md).
