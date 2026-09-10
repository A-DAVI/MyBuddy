# SDD — backend (Java / Spring Boot)

Specs quando o **contrato da API**, o modelo de dados ou a segurança mudam.

O backend é o lado mais estável do MyBuddy neste ciclo. SDD aqui é para mudança deliberada — não para “limpeza” oportunista.

## Incluir no SDD

- Endpoints, DTOs, validações, papéis Keycloak.
- Flyway vs documentos Mongo.
- Efeito em frontend/mobile (quem consome).
- Testes (JUnit / Testcontainers) e impacto no JaCoCo.

## Não incluir por padrão

- Rewrite de pacotes `Service`/`Controller` inteiros.
- Troca de IdP ou de banco (isso é ADR).
