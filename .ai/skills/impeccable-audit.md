# Skill: Impeccable audit (inventário)

**Fina de propósito.** Use o Impeccable do ambiente (`/audit` ou o comando equivalente da skill instalada no IDE) para **inventariar** uma superfície.  
**Não** copie a árvore completa de skills/scripts do Impeccable para este repositório.

## Papel no loop

```
Impeccable /audit  →  inventário
grilling           →  decisão
SDD + runbook      →  plano
código             →  só depois
```

O audit **não** prioriza sprint, **não** autoriza rewrite, **não** substitui Shape brief.

## Quando usar

- Antes de grilling de qualquer tela Angular (ou UI Flutter, se o Impeccable nativo estiver disponível).
- Quando o time não consegue listar estados (vazio, erro, loading, auth) da superfície.

## O que entregar

Lista curta, com evidência:

- Superfície / rotas / componentes grandes.
- Problemas observados (a11y, hierarquia, mocks, `localStorage`, auth fake, responsivo).
- O que **não** foi olhado.

Próximo passo obrigatório: [grilling.md](grilling.md) — não abrir PR de código só com o PDF/markdown do audit.

## O que não fazer

- Vendorar `.agents/skills/impeccable/**` inteiro.
- Tratar score do audit como backlog.
- Auditar o backend Java com esta skill (não é o papel do Impeccable).
- Inventar SDD de marketplace/perfil “completo” a partir de um único `/audit`.
