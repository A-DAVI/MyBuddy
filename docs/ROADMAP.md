# ROADMAP — ciclo atual (MyBuddy)

Documento curto de **foco**, não de backlog inventado. Sem números de issue fictícios.

## Foco ativo: dívida do frontend Angular

O web (`frontend/`, Angular 21) é a dívida principal deste ciclo: telas grandes nascidas de overhauls com IA, mocks e `localStorage` misturados com API, auth de demonstração ainda no caminho (`mockUserRole`), cobertura de teste baixa em componentes/serviços.

O **backend** (Java 21 / Spring Boot) é comparativamente confiável. Não há, neste ciclo, mandato de rewrite da API. Correções pontuais e testes sim; “aproveitar e redesenhar o domínio” não.

Mobile (Flutter) segue em paralelo quando houver SDD próprio — não é o gargalo de processo que este scaffolding endereça primeiro.

## Como o time avança (sem pular etapa)

1. **Inventário** — Impeccable `/audit` na superfície (vitrine, perfil, checkout, cadastro, …). Ver [`.ai/skills/impeccable-audit.md`](../.ai/skills/impeccable-audit.md).
2. **Decisão** — grilling ([`.ai/skills/grilling.md`](../.ai/skills/grilling.md)). Apetite limitado; um pedaço por vez.
3. **SDD + runbook** — [`docs/sdd/`](sdd/README.md) e [`docs/sprints/`](sprints/README.md).
4. **Código + testes** no recorte do runbook.

Não usar este ROADMAP como desculpa para um SDD único de “reescrever o Angular”. Cada superfície ganha o próprio grilling.

## Fora de foco (agora)

- Copiar domínio ou docs de outros produtos.
- Trocar Keycloak, Angular, Caddy ou o split Mongo/Postgres (ADRs).
- GitHub Actions de Claude Code / menções `@claude`.
- Marketplace “do zero” sem inventário + grilling da superfície atual.

## Referências

- Processo: [`CONTRIBUTING.md`](../CONTRIBUTING.md), [`AGENTS.md`](../AGENTS.md)
- ADRs: [`docs/tecnico/ADR/`](tecnico/ADR/)
- Cobertura frontend (histórico): [`docs/tecnico/auditoria-cobertura-frontend.md`](tecnico/auditoria-cobertura-frontend.md)
- Setup: [`README.md`](../README.md), [`CLAUDE.md`](../CLAUDE.md)
