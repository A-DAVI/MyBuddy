# Guia de Contribuição — MyBuddy

Obrigado por contribuir com o MyBuddy! Este guia explica como colaborar de forma consistente com o projeto.

Regras mecânicas (FAÇA / NÃO FAÇA, allowlist de escopos): [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md).  
Contexto para agentes: [`AGENTS.md`](AGENTS.md). Comandos: [`CLAUDE.md`](CLAUDE.md).

---

## Filosofia e fluxo de trabalho

Trabalhamos calmos: **inventar pouco, decidir explícito, implementar só o combinado**.

```
Impeccable (inventário) → grilling (decisão) → SDD → runbook → código → testes
```

| Etapa | O que é | O que não é |
|-------|---------|-------------|
| **Impeccable `/audit`** | Inventário da superfície (UI) | Priorização nem autorização de rewrite |
| **Grilling** | Stress-test do plano em sessão humana/IDE | Implementação; não usa `@claude` nem Action |
| **SDD** | Spec em [`docs/sdd/`](docs/sdd/README.md) | Código |
| **Runbook** | Passos em [`docs/sprints/`](docs/sprints/README.md) | Lista de desejos |

**Regra de ouro:** não há sprint sem runbook; não há runbook sem SDD.

**UI:** ao criar ou editar superfície visível, **Shape brief obrigatório** (problema, apetite, esboço, rabbit holes, no-gos). Modelo em [`docs/sdd/frontend/README.md`](docs/sdd/frontend/README.md).

Foco deste ciclo: dívida do **frontend Angular**. O backend não se reescreve “de passagem”. Ver [`docs/ROADMAP.md`](docs/ROADMAP.md).

Issue para grilling/SDD: template *Grilling → SDD* (label `sdd-pendente`). Skills: [`.ai/skills/`](.ai/skills/).

---

## Pré-requisitos

- Docker e Docker Compose
- Java 21
- Node.js 22+
- Angular CLI
- Maven
- Flutter 3.41+ (apenas para `mobile/`)

---

## Configurando o ambiente local

```bash
# 1. Clone o repositório
git clone https://github.com/EderHenriq/MyBuddy.git
cd MyBuddy

# 2. Configure as variáveis de ambiente
cp .env.example .env
# Edite o .env com suas configurações locais — nunca commite o arquivo

# 3. Suba a infraestrutura (Redis entra: o backend usa cache)
docker compose up -d postgres mongodb keycloak redis

# 4. Rode o backend
cd backend
./mvnw spring-boot:run

# 5. Rode o frontend
cd frontend
npm install
npm start
```

Stack completa em containers: `docker compose up --build` (ver [`README.md`](README.md) e [`CLAUDE.md`](CLAUDE.md)).

Hot-reload local (sem Caddy): `docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build` — frontend http://localhost:4200, backend http://localhost:8081, Keycloak http://localhost:8080.

---

## Fluxo de trabalho com Git

### Branches

O projeto segue o modelo:

```
main        → produção
Developer   → integração (branch base para PRs de produto)
feature/*   → novas funcionalidades
fix/*       → correções de bug
hotfix/*    → correções urgentes em produção
```

Também aparecem `docs/*`, `chore/*`, `test/*`, `ci/*` quando o tipo descreve melhor o trabalho.

Sempre crie sua branch a partir da `Developer` (código de produto):

```bash
git checkout Developer
git pull origin Developer
git checkout -b feat/MY-XXX-descricao-curta
```

PRs de **produto** apontam para `Developer`. Alterações de processo que precisam valer na branch padrão do GitHub podem apontar para `main` — sem misturar rewrite de aplicação no mesmo PR.

### Padrão de branch

```
<tipo>/MY-<número>-<descricao-curta>
<tipo>/MYB-<número>-<descricao-curta>
```

Exemplos:
```
feat/MY-42-filtro-especie-animais
feat/MYB-42-filtro-especie-animais
fix/MY-87-corrige-login-keycloak
hotfix/MY-101-token-expirado
docs/sdd-vitrine-adocao
```

---

## Convenção de Commits

O projeto usa [Conventional Commits](https://www.conventionalcommits.org/) com referência ao card do Jira (`[MY-XXX]` ou `[MYB-XXX]`). Allowlist de escopos e o que não fazer: [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md). Texto completo: [`.github/COMMIT_CONVENTION.md`](.github/COMMIT_CONVENTION.md).

### Estrutura

```
<tipo>(escopo): <descrição curta>  [MY-XXX] #<ação-jira>
<tipo>(escopo): <descrição curta>  [MYB-XXX] #<ação-jira>
```

### Tipos

| Tipo | Quando usar |
|------|-------------|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `hotfix` | Correção urgente em produção |
| `refactor` | Melhoria sem alterar comportamento |
| `chore` | Manutenção, deps, configs |
| `docs` | Documentação |
| `test` | Testes |
| `style` | Formatação sem lógica |
| `perf` | Performance |
| `ci` | CI/CD |

### Escopos do monorepo

| Escopo | Módulo |
|--------|--------|
| `backend` | Java / Spring Boot |
| `frontend` | Angular |
| `mobile` | Flutter |
| `auth` | Keycloak / JWT |
| `infra` | Docker, Caddy |
| `db` | Migrations Flyway |
| `docs` | Documentação |
| `sdd` | Specs e runbooks |
| `ci` | GitHub Actions existente |
| `test` | Apenas testes |

### Exemplos

```bash
# Feature
git commit -m "feat(frontend): adiciona filtro por espécie na vitrine  [MY-55] #done"

# Bug fix
git commit -m "fix(auth): corrige refresh token expirado  [MY-88] #done"

# Chore
git commit -m "chore: atualiza dependências do Angular  [MY-60]"
```

### Smart Commits (Jira)

| Ação | Comando |
|------|---------|
| Mover para em progresso | `#in-progress` |
| Concluir | `#done` |
| Registrar tempo | `#time 2h 30m` |

---

## Abrindo um Pull Request

1. Certifique-se de que os testes do recorte passam localmente (e lint no frontend)
2. Crie o PR apontando para a branch `Developer` (produto)
3. Preencha o template de PR obrigatório — SDD/runbook se o escopo muda; Shape se UI; sem segredos
4. Aguarde ao menos uma aprovação antes de mergear
5. Não faça squash de commits — mantenha o histórico

Review: [`.ai/skills/pr-review-contribution.md`](.ai/skills/pr-review-contribution.md).

---

## Rodando os testes

```bash
# Backend
cd backend
./mvnw test
./mvnw verify   # quality gate JaCoCo (CI: ≥ 61% linha)

# Frontend
cd frontend
npm test
npm run lint
npm run e2e

# Mobile
cd mobile
flutter test
```

O CI (`.github/workflows/ci.yml`) roda em push/PR para `Developer` e `main` — **não desmontar** esse workflow. Não adicionar Actions de Claude Code (`anthropics/claude-code-action`, `@claude`, crons de auditoria/resumo).

---

## Estrutura do repositório

```
MyBuddy/
├── backend/                 # Java 21 + Spring Boot 3 + Keycloak + Maven
├── frontend/                # Angular 21 (SSR)
├── mobile/                  # Flutter 3.41+
├── docs/
│   ├── academico/           # Documentação acadêmica (preservar)
│   ├── tecnico/             # ADRs, análises, cobertura
│   ├── sdd/                 # Specs (após grilling)
│   ├── sprints/             # Runbooks
│   ├── CONVENTIONS.md
│   └── ROADMAP.md
├── docker/                  # Configurações Docker
├── .ai/                     # Skills e hooks de processo
├── .github/                 # CI existente, templates
├── AGENTS.md
├── CLAUDE.md
└── docker-compose.yml
```

Não apagar `docs/academico` nem `docs/tecnico`.

---

## Dúvidas

Abra uma issue (bug ou grilling/SDD) ou o canal do time no Jira.
