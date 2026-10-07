# F2-06: estrutura do item

**FECHADO — ciclo SDD 14/14** (2026-10-07). Backend-only (PRM-001): unidades com fator versionado, grade/SKU, kit com composição versionada, rastreabilidade (lote/série), RBAC e isolamento.

## PRs (systeme-erp-backend)

| PR | Conteúdo |
| --- | --- |
| #203–#214 | tasks 01–06 (implementação) |
| #215 | review APROVADO, QA 42 SCN, pacote PR |
| #216 | merge-report passo 11 |
| #217 | consolidação contratos + arquivamento passo 13 |
| passo 14 | aprendizados + `sdd/metricas.csv` (PR pendente nesta sessão) |

Evidence SHA implementação: `6a320d2`. Merge SDD #215: `8bbd4bc`. Consolidação: `dd4ec48` (#217).

## Contratos vivos atualizados

- `sdd/contratos/produtos/contrato.md` — +10 comportamentos (unidades, grade, SKU, kit, rastreabilidade)
- `sdd/contratos/dados/contrato.md` — tabelas e permissões RBAC

Histórico: `sdd/historico/2026-10-01-f2-06-estrutura-item/`.

## Métricas

`lead_total_h=149`, `reauditorias=8`, `revisoes_plano=11`, `issues_review=0` (CSV), 42 SCN.

## Aprendizados-chave

- Tasks 05/06 ficaram `done`/`pending` — bloqueou `pre-consolidate` até `completed` + realinhamento Evidence SHA (padrão F2-01).
- REVIEW-001: review antes do merge da task_06 (#214).
- 8 reauditorias; review final sem P0–P3 de código.

## Próximo no plano F2

F2-07 a F2-12 (6 issues backend restantes na fase). Dependências: F2-09/F2-10 para trilha fiscal.
