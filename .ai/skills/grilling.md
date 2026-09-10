# Skill: grilling (decisão)

Grilling **estressa o plano**. Não implementa, não formata o repo, não reescreve o backend.

Pré-requisito de UI: inventário Impeccable ([impeccable-audit.md](impeccable-audit.md)).

## Onde grilar

Sessão humana ou IDE. Issue opcional: [`.github/ISSUE_TEMPLATE/grilling_sdd.md`](../../.github/ISSUE_TEMPLATE/grilling_sdd.md) (label `sdd-pendente`).  
Não mencionar `@claude`, não usar Action, não depender de label `claude`.

## Perguntas (adaptar ao MyBuddy)

1. Qual superfície **uma**? (não “o Angular todo”)
2. Qual o apetite (tempo/risco)? O que cai se estourar?
3. O inventário mostra mock/`localStorage`/god component — isso é **escopo** ou **armadilha**?
4. A API já faz isso? Se sim, por que o front finge dados locais?
5. Auth: Keycloak no caminho real? Há tentação de `mockUserRole`?
6. Backend precisa mudar? Se a resposta não for um “sim” estreito, **não** abrir SDD de Java neste ciclo.
7. Rabbit holes: SSR, Mercado Pago, upload, multi-role, Prettier em massa.
8. No-gos: ADRs, CI, PII, rewrite de `Service` no backend.
9. Como vamos **ver** que acabou? (teste + critério de aceite, não “tela mais bonita”)
10. O time está calmo o bastante para **não** implementar nesta sessão?

## Saídas possíveis

| Decisão | Segue |
|---------|--------|
| Seguir | SDD em `docs/sdd/<modulo>/` + Shape se UI |
| Recortar | Novo apetite; re-grilar o pedaço menor |
| Parar | Inventário arquivado; sem sprint |

## Depois

[sprint-start.md](sprint-start.md) só com SDD **e** runbook.  
Proibido “já que grillamos, vou commitando o refactor”.
