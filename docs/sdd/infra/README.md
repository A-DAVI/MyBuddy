# SDD — infra

Docker Compose, Caddy, Keycloak realm **como artefato de infra**, CI, env de deploy.

Não esvaziar `.github/workflows/ci.yml`.  
Não adicionar GitHub Actions que usem `anthropics/claude-code-action`, menções `@claude` ou crons de auditoria/resumo.

Mudança de borda (Caddy) ou de persistência (volumes, Postgres/Mongo/Redis) precisa de critérios de aceite e plano de rollback no runbook.
