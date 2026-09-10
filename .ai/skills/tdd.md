# Skill: TDD (recorte MyBuddy)

Teste que falha → código mínimo → refactor do recorte.  
Não gerar suíte gigante fora do runbook.

## Backend

```bash
cd backend && ./mvnw test
# CI também roda verify + JaCoCo ≥ 61% linha
```

- JUnit 5, Mockito, `spring-security-test`, Testcontainers quando o teste exige Mongo/Postgres/Keycloak reais.
- Nomes e dados sintéticos; sem PII.
- Não baixar o quality gate para “passar o PR”.

## Frontend

```bash
cd frontend
npm test
npm run test:coverage   # quando o runbook pedir
npm run e2e             # fluxo crítico (Playwright)
```

- Vitest + TestBed no recorte.
- HTTP: `HttpTestingController` — não “sucesso” via mock de `localStorage`.
- E2E não deve depender de catálogo gravado no browser como fonte de verdade.

## Mobile

```bash
cd mobile && flutter test && flutter analyze
```

## UI

TDD de componente não substitui Shape brief. Comportamento (filtro, guard, error state) ganha teste; pixel não precisa de snapshot em massa neste ciclo.
