# SDD — frontend (Angular)

Specs de **superfície web**. Sem Shape brief não começa implementação de UI.

## Shape brief (obrigatório)

Arquivo irmão ou seção no SDD, curto:

1. **Problema** — quem sofre o quê na tela atual.
2. **Apetite** — tempo/risco que o time aceita (ex.: “uma tela, sem redesenhar o design system”).
3. **Esboço** — fluxo, estados (loading/vazio/erro/sucesso), não mockup pixel-perfect obrigatório.
4. **Rabbit holes** — o que vai explodir o escopo (SSR, Keycloak, Mercado Pago, god component).
5. **No-gos** — o que esta fatia **não** faz (ex.: não desligar Keycloak; não persistir catálogo em `localStorage`).

Inventário vem do Impeccable; este brief + grilling viram o SDD.

## Cuidados deste módulo

- Não tratar mock/`localStorage` como requisito de produto.
- Não misturar rewrite de backend no mesmo SDD.
- Telas já inchadas (`marketplace`, `perfil`, cadastro): preferir extração a “mais uma feature no mesmo arquivo”.
