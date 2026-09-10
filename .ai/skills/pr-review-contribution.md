# Skill: PR review (contribuição)

Alinhado a [`.github/PULL_REQUEST_TEMPLATE.md`](../../.github/PULL_REQUEST_TEMPLATE.md) e [`CONTRIBUTING.md`](../../CONTRIBUTING.md).

## Autor (antes de pedir review)

- [ ] Escopo = runbook; sem surpresa de módulo.
- [ ] SDD + runbook linkados se comportamento/superfície mudou.
- [ ] Shape/UX se UI.
- [ ] Conventional Commits + card Jira.
- [ ] Testes/lint do recorte; CI local equivalente.
- [ ] Zero segredos; fixtures sintéticas.
- [ ] Sem mega-Prettier; sem drive-by backend.
- [ ] PR de produto aponta para `Developer` (convenção do time).

## Revisor

- Ler SDD/runbook **antes** do diff se o checklist marcar mudança de escopo.
- Frontend: [frontend-review.md](frontend-review.md).
- Auth/pagamento/migration: desconfiança extra.
- Pedir recorte se o PR mistura docs de processo com rewrite de tela — este scaffolding não é desculpa para os dois no mesmo PR de produto.

## Não bloquear por

- Preferência de rename cosmética fora do runbook.
- “Deveria ser React.”
- Falta de workflow `@claude`.
