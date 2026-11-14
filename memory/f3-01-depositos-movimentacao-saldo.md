# F3-01: depósitos, movimentação e saldo

## Status

**FECHADO — ciclo SDD 14/14** (consolidação 13/11/2026 America/Sao_Paulo; métricas e passo 10 retroativo no PR de aprendizados).

`rigor: large` / `risco: alto` / `autonomia: autonomo_ate_pr` / `alvo: branch` / backend-only (PRM-001).

## Entrega

EST-DEP-001, EST-MOV-002, EST-SALDO-003: depósitos por filial, ledger de movimentos
imutáveis (entrada/saída/ajuste/estorno), transferência interna atômica, consulta de
saldo (incl. lote), RBAC `estoque.*`, trilha de auditoria, endereçamento opcional.

Contratos vivos promovidos em `systeme-erp-backend` `main` `b94a986` (#320):
`modelo-dados-suprimentos`, `dados`, `produtos`, `empresas`.

## PRs principais

| PR | Conteúdo |
| --- | --- |
| #278–#282, #317, #318 | tasks 01–06 (implementação) |
| #319 | review + QA |
| #320 | consolidação (passo 13) |
| passo 14 | `metricas.csv` + pacote PR retroativo + esta memória |

Evidence SHA implementação: `aeaa8b8`. Merge SDD review/QA: `0feaab1`.

## QA / review

APROVADO. SCN-069..080 (integração Postgres, CI sem SKIP).

## Fora de escopo (documentado)

- Reserva, transferência entre filiais, saldo negativo político: F3-02 (PRM-004).
- CMV / Kardex completo: F3-03 / F3-06 (PRM-003).
- Telas de estoque: incremento frontend futuro.

## Aprendizados SDD (resumo)

- `execucao.base_sha` realinhado após consolidações alheias na `main` (AUD-005).
- `Baixar` migration: passos ≠ delta de versão (gaps 000026..029).
- Modelo uma-PR-por-task: usar `implementacao-rastreio.md` + merge-report; pacote passo 10 pode ser retroativo no histórico.

Histórico: `sdd/historico/2026-10-21-f3-01-depositos-movimentacao-saldo/`.
