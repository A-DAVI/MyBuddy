# Skill: sprint-start

Use **antes** da primeira linha de código da fatia.

## Gate

- [ ] Inventário feito (Impeccable se UI).
- [ ] Grilling com decisão **seguir** (não “talvez”).
- [ ] SDD em `docs/sdd/<modulo>/`.
- [ ] Runbook em `docs/sprints/<modulo>/`.
- [ ] UI: Shape brief no SDD ou arquivo irmão.
- [ ] Branch a partir de `Developer` (`feat|fix|.../MY-XXX-...`).
- [ ] Lista do que **não** tocar (em especial `backend/` em sprint de front).

**Sem runbook não há sprint. Sem SDD não há runbook.**

## Durante

- Seguir o runbook na ordem; desvio vira nota + possível re-grilling.
- TDD no recorte ([tdd.md](tdd.md)).
- Hooks de path ([../hooks.md](../hooks.md)).
- Commits Conventional + `[MY-XXX]` / `[MYB-XXX]`.

## Não começar se

- O objetivo for “reescrever o frontend”.
- O runbook incluir drive-by no Spring Security “por higiene”.
- Alguém propuser Prettier no tree inteiro como primeiro passo.
