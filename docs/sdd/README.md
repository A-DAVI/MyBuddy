# docs/sdd — Spec-Driven Design (MyBuddy)

Aqui vivem as **specs**. Código só depois de SDD **e** runbook (`docs/sprints/`).

Filosofia: [`CONTRIBUTING.md`](../../CONTRIBUTING.md) · regras: [`CONVENTIONS.md`](../CONVENTIONS.md) · agentes: [`AGENTS.md`](../../AGENTS.md)

## Módulos

| Pasta | Uso |
|-------|-----|
| [frontend/](frontend/README.md) | Superfícies Angular (Shape brief obrigatório) |
| [backend/](backend/README.md) | API Java/Spring — só quando o comportamento da API muda |
| [mobile/](mobile/README.md) | Flutter |
| [infra/](infra/README.md) | Docker, Caddy, CI, compose |
| [auth/](auth/README.md) | Keycloak, JWT, papéis, sessão |

## Antes de escrever SDD

1. Inventário Impeccable da superfície (se UI).
2. Grilling com decisão explícita (seguir / recortar / parar).
3. Só então este documento.

Não abrir SDD de “rewrite completo do frontend” sem grilling que aceite esse apetite. O padrão é **uma superfície** (ex.: vitrine de adoção, checkout, perfil).

## Esqueleto mínimo de um SDD

Nome sugerido: `docs/sdd/<modulo>/<slug>.md` (slug em kebab-case, sem número de issue inventado).

```markdown
# SDD: <título>

- Módulo:
- Superfície / rotas / endpoints:
- Inventário (link Impeccable ou nota):
- Decisão de grilling (link ou resumo):
- Shape brief (se UI):

## Problema
## Escopo
## Fora de escopo
## Desenho (comportamento + contratos)
## Rabbit holes
## No-gos
## Critérios de aceite
## Runbook
- caminho: docs/sprints/<modulo>/...
```

Issue de grilling: template [`.github/ISSUE_TEMPLATE/grilling_sdd.md`](../../.github/ISSUE_TEMPLATE/grilling_sdd.md) (label `sdd-pendente`).
