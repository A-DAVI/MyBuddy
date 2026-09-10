# Hooks (agnósticos) — MyBuddy

Não são Git hooks instalados: são **gatilhos de disciplina** quando o diff toca certos paths. Vale para humano e agente.

## `backend/**/*.java`

- Seguir FAÇA/NÃO FAÇA de backend em [`docs/CONVENTIONS.md`](../docs/CONVENTIONS.md).
- Não refatorar pacotes vizinhos “já que estou aqui”.
- Se mudar contrato público (controller/DTO/security): precisa SDD em `docs/sdd/backend/` ou `docs/sdd/auth/` + runbook.
- Rodar `./mvnw test` no módulo/teste afetado.
- Sem logs de PII; sem secrets hardcoded.

## `backend/src/main/resources/db/migration/**` e `backend/src/main/java/db/migration/**`

- Migration nova, versionada, reversível ou com rollback descrito no runbook.
- Não editar migration já aplicada em ambientes compartilhados — só forward.
- Testar contra Postgres do compose (não só H2, se o SQL for específico).

## `frontend/src/**`

- Shape brief se a superfície muda.
- Proibido: god-file extra, mock auth em prod, `localStorage` como catálogo, fallback silencioso para mock.
- Format/lint **só nos arquivos tocados** (Prettier `printWidth` 150). Proibido mega-diff de formatação.
- `npm test` / `npm run lint` no recorte; e2e se o fluxo crítico mudou.

## `mobile/**`

- Não duplicar regra de preço/estoque/papel que a API já aplica.
- Sem PII em fixtures (`mobile/test/**`).
- `flutter test` / `flutter analyze` no recorte.

## `.github/workflows/**`

- Não remover o job de `ci.yml` sem SDD de infra/CI.
- Não adicionar `anthropics/claude-code-action`, `@claude`, `claude.yml`, `claude-code-review.yml`, cron de auditoria ou resumo diário.

## `docs/sdd/**` e `docs/sprints/**`

- SDD e runbook andam em par quando o escopo é de implementação.
- Não escrever SDD de rewrite total do Angular neste ciclo sem grilling que aceite o apetite.

## Fixtures, seeds, Playwright, `*Initializer*`

- Dados sintéticos. Sem CPF, e-mail real, cartão, token, conversa de usuário.
- Não reativar seed de produção disfarçada de “dev convenience”.

## Sempre

- Conventional Commits + escopo da allowlist.
- Não commitar `.env`, uploads de usuário, dumps.
