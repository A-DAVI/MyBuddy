# docs/sprints — runbooks

Aqui vivem os **runbooks de implementação**.  
Regra: **não há sprint sem runbook; não há runbook sem SDD.**

SDD correspondente: [`docs/sdd/`](../sdd/README.md).  
Como começar um sprint: [`.ai/skills/sprint-start.md`](../../.ai/skills/sprint-start.md).

## Módulos

| Pasta | Runbooks de |
|-------|-------------|
| [frontend/](frontend/README.md) | Angular |
| [backend/](backend/README.md) | API / Flyway |
| [mobile/](mobile/README.md) | Flutter |
| [infra/](infra/README.md) | Compose, Caddy, CI |
| [auth/](auth/README.md) | Keycloak / sessão |

## Esqueleto mínimo

`docs/sprints/<modulo>/<slug>.md`:

```markdown
# Runbook: <título>

- SDD: docs/sdd/<modulo>/...
- Apetite / fatia:
- Branch sugerida:

## Pré-condições
## Passos (ordem, arquivos prováveis, o que não tocar)
## Verificação (comandos de teste/lint + checagem manual)
## Rollback
## Fora desta fatia
```

O runbook não é o SDD: é a lista executável. Se o passo não cabe no apetite, volta ao grilling — não “só mais um arquivo”.
