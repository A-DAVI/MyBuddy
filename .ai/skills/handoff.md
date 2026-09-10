# Skill: handoff

Passar trabalho entre humano ↔ agente ou entre sessões **sem** perder o loop.

## Incluir sempre

- Objetivo da fatia (uma frase).
- Links: inventário, grilling, SDD, runbook, PR/branch, card `[MY-XXX]` / `[MYB-XXX]`.
- Feito / não feito / bloqueado.
- Comandos já rodados (`./mvnw test`, `npm test`, e2e).
- Arquivos **proibidos** nesta fatia.
- Riscos (auth, pagamento, migration).

## Não incluir

- Segredos, dumps, tokens, `.env`.
- “Continua o rewrite do front” sem SDD.
- Instrução para acionar `@claude` em issue.

Modelo curto:

```markdown
## Handoff
- Fatia:
- SDD / runbook:
- Branch:
- Próximo passo concreto:
- Não fazer:
```
