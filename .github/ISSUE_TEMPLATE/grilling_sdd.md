---
name: Grilling → SDD
about: Inventário (Impeccable) e grilling para decidir um SDD. Não implementa código.
title: "docs(sdd): <superfície ou módulo>"
labels: ["sdd-pendente"]
---

## Superfície / módulo

<!-- Ex.: vitrine de adoção Angular, checkout, perfil, Keycloak roles, Flyway X -->

- [ ] `frontend`
- [ ] `backend`
- [ ] `mobile`
- [ ] `infra`
- [ ] `auth`

## Inventário (Impeccable)

Auditoria de UI é **inventário**, não decisão. Cole o resumo ou o caminho do relatório.

- Superfície / rotas:
- Achados (god component, mock, `localStorage`, a11y, estados):
- O que **não** foi olhado:

Se não for UI, descreva o inventário técnico (endpoints, ADRs, testes).

## Apetite e no-gos

- Apetite (o que cabe nesta fatia):
- No-gos (ex.: não reescrever backend; não desligar Keycloak; sem catálogo em `localStorage`):
- Rabbit holes:

## Grilling (sessão humana ou IDE)

Faça o grilling **offline neste issue ou na IDE**, com o skill [`.ai/skills/grilling.md`](../../.ai/skills/grilling.md).  
Não use este card para disparar automação de agente no GitHub.

- [ ] Plano estressado (perguntas do skill)
- [ ] Decisão: seguir / recortar / parar
- [ ] Resumo da decisão:

## SDD e runbook (depois da decisão)

- SDD (caminho futuro): `docs/sdd/<modulo>/`
- Runbook (caminho futuro): `docs/sprints/<modulo>/`
- UI: Shape brief será parte do SDD

## Fora de escopo deste issue

- Implementação / PR de produto
- Rewrite completo do frontend Angular
- Alterar `.github/workflows/ci.yml` ou adicionar Actions de terceiros
