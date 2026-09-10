# Skill: diagnosing-bugs

Isolar **antes** de patchar.

## 1. Reproduzir

- Ambiente: compose vs. `ng serve` + `./mvnw spring-boot:run`.
- URL, papel (adotante / ONG / pet shop / admin), anônimo vs. logado.
- Network: 401/403 (Keycloak), 5xx (API), CORS, timeout.

## 2. Camada

| Sintoma | Olhar primeiro |
|---------|----------------|
| Tela com dados “bonitos” após falha de rede | mock / `localStorage` no service Angular |
| Loop de login / token | Keycloak realm, clock, `AuthService`, cookie `mybuddy_session` |
| 403 em endpoint | `JwtAuthConverter`, roles, client |
| Pedido/pagamento | Mercado Pago webhook + Postgres; não só o checkout mock |
| Pet/ONG/evento | Mongo + repositories `mongo/` |
| “Funciona no Docker, não no ng serve” | `environment.ts`, CORS, `KEYCLOAK_URL`, porta 4200 vs 80 |

## 3. Disciplina

- Um hipótese por vez; log sem PII.
- Não “corrigir” o backend inteiro porque um DTO veio null.
- Se o bug é dívida estrutural (god page, mock), o fix tático não vira rewrite — abre grilling.
- Fixtures de reprodução: dados sintéticos.

## 4. Depois do fix

Teste que falharia antes. Commit `fix(<escopo>):` com card Jira.
