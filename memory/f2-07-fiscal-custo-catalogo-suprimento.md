# F2-07: fiscal ampliado, custo, catálogo e suprimento

**FECHADO — ciclo SDD 14/14** (2026-11-04). Backend-only (PRM-001).

## Entrega

- Fiscal ampliado versionado (IPI, PIS/COFINS, ST por UF), simulação do item via matriz F2-10.
- Custos com RBAC (`produtos.custo.*`), formador de preço, mascaramento sem `produtos.custo.ver`.
- Catálogo comercial (descrição rica, ficha, mídias filesystem — ADR-001).
- Vínculo produto×fornecedor preferencial + histórico de preço (`produtos.fornecedor.gerir`).

24 cenários SCN-081..104 / TST-081..104; 0 reauditorias; lead ~171h.

## PRs (backend)

| PR | Conteúdo |
| --- | --- |
| #285–#286 | SDD especificação |
| #293–#297 | tasks 01–05 |
| #298 | review, QA, pacote |
| #299 | consolidação contratos + histórico |
| #300 | aprendizados + `sdd/metricas.csv` |

`main` pós-fechamento inclui `838b631` (#300).

## Histórico SDD

`sdd/historico/2026-10-27-f2-07-fiscal-custo-catalogo-suprimento/`  
Contratos vivos: `produtos`, `parceiros`, `dados`, `testes-fiscais`.

## Fora do escopo (ainda)

- Telas frontend catálogo/custo/fornecedor.
- Custo médio ledger (F3-03 / PRM-004).

## Lições (passo 14)

- Merge squash do PR SDD: alinhar Evidence SHA ao commit na `main` (`3fc532e`), não ao commit intermediário.
- `sdd/metricas.csv` no passo 14 dispara `mudanca de produto sem incremento SDD afetado` no guard — merge manual esperado (ver `sdd-guard-authority-gap.md`).
