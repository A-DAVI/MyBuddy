# `.ai/` — skills e hooks (MyBuddy)

Pasta **agnóstica de ferramenta**: Markdown que qualquer humano ou agente segue.  
Não é vendor do Impeccable nem árvore `.claude/`. Não assume GitHub Action, `@claude` ou labels `claude`.

| Arquivo | Função |
|---------|--------|
| [hooks.md](hooks.md) | Gatilhos por path (o que fazer / o que evitar) |
| [skills/](skills/) | Playbooks curtos do loop de processo |

Contexto do repo: [`AGENTS.md`](../AGENTS.md).  
Comandos: [`CLAUDE.md`](../CLAUDE.md).  
Regras: [`docs/CONVENTIONS.md`](../docs/CONVENTIONS.md).

## Skills

| Skill | Quando |
|-------|--------|
| [impeccable-audit.md](skills/impeccable-audit.md) | Inventário de UI **antes** do grilling |
| [grilling.md](skills/grilling.md) | Decidir o plano (não implementar) |
| [sprint-start.md](skills/sprint-start.md) | Checagem antes de código |
| [frontend-review.md](skills/frontend-review.md) | Review de PR Angular |
| [diagnosing-bugs.md](skills/diagnosing-bugs.md) | Isolar bug por camada |
| [handoff.md](skills/handoff.md) | Passar contexto entre pessoas/agentes |
| [research.md](skills/research.md) | Pesquisar sem copiar outro produto |
| [tdd.md](skills/tdd.md) | Testes primeiro no recorte |
| [pr-review-contribution.md](skills/pr-review-contribution.md) | Review alinhado ao template de PR |

Loop: **Impeccable (inventário) → grilling (decisão) → SDD → runbook → código.**
