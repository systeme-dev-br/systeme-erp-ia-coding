# F2-08: operação do cadastro de item

## Status

**FECHADO — ciclo SDD 14/14** (consolidação 12/11/2026 America/Sao_Paulo; aprendizados em PR separada).

`rigor: large` / `risco: medio` / `autonomia: autonomo_ate_pr` / `alvo: branch`.

## Entrega

OPS-012 (estado operacional + exclusão controlada), OPS-013 (busca operacional + resolver embalagem/GTIN), OPS-014 (importação CSV em lote com desfazer).

Contratos vivos promovidos em `systeme-erp-backend` `main` `8b573bb` (#314): `produtos`, `dados`, `modelo-dados-plataforma`.

Frontend (`systeme-erp-frontend` `main` ~`2cb316e`): listagem/estado, busca, wizard de importação.

## PRs principais

| Repo | PRs |
| --- | --- |
| backend | #292, #310, #311 (impl.); #312 review+QA; #313 pacote+merge-report; #314 consolidação; aprendizados (passo 14) |
| frontend | #35–#37 (tasks 04–06) |

Merge código backend referência: `8d5be80`. Merge SDD passo 11: `5fa2426`.

## QA / review

APROVADO. SCN-105..119 (integração Postgres). TST-UI-105..108.

## Débitos documentados

- **REVIEW-001 (P2):** filtro por `status` na listagem só no cliente; API sem query.
- **PRM-004:** saldo/preço na busca dependem de F3-01/F3-07.

## Aprendizados SDD (resumo)

- Cross-repo: citar frontend e SHAs distintos no pacote SDD.
- Guard #313: não alterar `execucao.md` após auditoria; alinhar Evidence SHA quando tasks SDD ainda mudam; atualizar `base_sha` quando contratos na `main` avançam (F2-07).
- Teste Vitest: evitar `clone().formData()` em upload CSV.

Histórico: `sdd/historico/2026-10-28-f2-08-operacao-cadastro-item/`.
