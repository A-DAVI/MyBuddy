# SDD — auth

Keycloak (realm `mybuddy`), clients, roles (adotante, ONG, pet shop, admin), resource server, sessão no Angular (`AuthService` / `keycloak-angular`) e no Flutter.

## No-gos típicos

- Auth mock (`mockUserRole` / bypass) em caminho de produção.
- Guardar token em `localStorage` como substituto do IdP.
- Reabrir ADR-001 sem ADR novo.

Cadastro ainda pode passar pelo backend (`/api/auth/cadastro` + sync Keycloak) — documentar o fluxo, não duplicá-lo em paralelo.
