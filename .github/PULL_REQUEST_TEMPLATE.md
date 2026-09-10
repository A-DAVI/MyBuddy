## Task Jira

> Vincule a task relacionada a este PR

- **Card:** [MYB-XXX](https://projetodesoftware420.atlassian.net/browse/MYB-XXX) <!-- ou MY-XXX -->
- **Tipo:** <!-- Feature | Bug | Hotfix | Refactor | Chore | Docs -->

---

## O que foi feito?

> Descreva de forma objetiva o que este PR implementa ou resolve.

<!-- Ex: Implementa endpoint de listagem de animais com filtros por espécie e porte -->

---

## Escopo das mudanças

> Marque os módulos afetados por este PR

- [ ] `backend` — Java / Spring Boot
- [ ] `frontend` — Angular
- [ ] `mobile` — Flutter
- [ ] `auth` — Keycloak / sessão
- [ ] `infra` — Docker / CI / configurações
- [ ] `docs` / `sdd` — documentação ou spec

---

## Processo (SDD / runbook / Shape)

> Obrigatório quando o PR muda comportamento, contrato de API ou superfície de UI. Docs triviais (typo) podem marcar N/A.

- [ ] Há SDD em `docs/sdd/` **ou** N/A (justifique)
- [ ] Há runbook em `docs/sprints/` **ou** N/A (justifique)
- [ ] UI: Shape brief preenchido (problema, apetite, esboço, rabbit holes, no-gos) **ou** N/A
- [ ] Inventário Impeccable + grilling foram feitos se a fatia é de interface **ou** N/A

---

## Checklist de Testes e qualidade

> Confirme que os cenários abaixo foram validados antes de abrir o PR

- [ ] Testei localmente e o comportamento está conforme esperado
- [ ] Cobri os casos de sucesso e os casos de erro
- [ ] Não há erros ou warnings novos no console
- [ ] Validei em mais de um ambiente (se aplicável)
- [ ] Testes automatizados foram criados ou atualizados (se aplicável)
- [ ] Lint/testes do recorte passaram (`./mvnw test`, `npm test` / `npm run lint`, `flutter test` conforme o módulo)
- [ ] Não incluí dump Prettier de árvore inteira nem refactor de backend fora do escopo

---

## Checklist de Code Review

> Para o **autor** preencher antes de solicitar revisão

- [ ] O código segue [`docs/CONVENTIONS.md`](../docs/CONVENTIONS.md) e o [`CONTRIBUTING.md`](../CONTRIBUTING.md)
- [ ] Não há código comentado ou `TODO` esquecido sem justificativa
- [ ] Variáveis, métodos e classes têm nomes claros e descritivos
- [ ] Não há lógica duplicada que poderia ser extraída
- [ ] Dados sensíveis **não** estão no diff (tokens, senhas, chaves, `.env`, PII em fixtures)
- [ ] Migrations de banco (se houver) foram testadas e são reversíveis / documentadas no runbook
- [ ] Frontend: sem auth mock em caminho real, sem `localStorage` como fonte de verdade do catálogo

---

## Notas para o Revisor

> Algum ponto de atenção, decisão técnica ou contexto que o revisor deve saber?

<!-- Ex: Optei por X em vez de Y porque... | Preciso de opinião sobre a abordagem em ... -->

---

## Como testar

> Passo a passo para o revisor reproduzir e validar as mudanças

```
1. 
2. 
3. 
```
