# Skill: frontend-review (Angular)

Review de PR que toca `frontend/src/**`. Processo: [pr-review-contribution.md](pr-review-contribution.md).

## Bloqueia merge (a menos que o SDD aceite explicitamente)

1. **God component** — mais responsabilidade empilhada em `marketplace.ts`, `perfil.ts`, cadastro, checkout, landing.
2. **Auth mock** — `mockUserRole`, bypass de guard, sessão só no `localStorage`.
3. **Catálogo local como verdade** — `mybuddy_produtos_local` / pedidos / cupons como fallback silencioso.
4. **`catchError` → mock de sucesso** — esconde API fora do ar.
5. **Mega-format** — milhares de linhas Prettier sem mudança de comportamento.
6. **Drive-by backend** no mesmo PR “para o front funcionar”.
7. Superfície nova/alterada **sem** Shape brief / SDD quando o template de PR pede.

## Pedir

- Estados de loading / vazio / erro visíveis.
- Papel e token via `AuthService` / Keycloak.
- Testes Vitest no recorte; Playwright se o fluxo do CI/usuário mudou.
- Diff pequeno o bastante para o revisor humano acompanhar.

## Não pedir neste ciclo

- Migração para React (ADR-002).
- Cobertura 100% de todos os 30 specs em falta — só o que a fatia toca, alinhado ao runbook.
