---
name: f1-09-governanca-acesso
description: "Estado do incremento SDD F1-09 (governança de acesso) do systeme-erp-backend — SoD, relatório de acessos, exportação e revisão"
metadata:
  node_type: memory
  type: project
  modified: 2026-09-18T20:52:00Z
---

Incremento SDD **f1-09-governanca-acesso** — issue F1-09 do plano de fases,
histórias RBAC-GOV-012/013 (`docs/historias/usuarios-papeis-permissoes.md`).

## Status

**FECHADO — ciclo SDD 14/14** (2026-09-18). Consolidação @ `6005fbb` (PR #94);
aprendizados no passo 14 (PR pendente).

## Entrega

- Migrations `public/000009` + `tenant/000010` (SoD, governança, revisão,
  exportação)
- `SoDVerificador` preventivo na atribuição RBAC + varredura detectiva
  (`cmd/sod-varredura` + API)
- Exceções SoD documentadas; parâmetros por empresa (`somente_detectivo`)
- Relatório de acessos (3 visões), comparador via trilha, destaques de risco
- Exportação CSV síncrona/assíncrona; sessão de revisão com revogação em lote
- `exigirGovernancaEmpresa` (membership + permissão) em rotas `/governanca/*`

## PRs

| # | Conteúdo |
| --- | --- |
| #81–#84 | Triagem, PRD, TechSpec, plano |
| #85–#92 | Tasks 01–08 (implementação) |
| #93 | Review/QA |
| #94 | Consolidação contratos vivos |

## Contratos

`dados`, `modelo-dados-multiempresa`, `empresas` — impactos aplicados no passo
13. Histórico: `sdd/historico/2026-09-18-f1-09-governanca-acesso/`.

## Qualidade

21 SCN / 21 TST, review e QA APROVADOS, 0 bugs, 0 P0/P1.

## Aprendizados

- `exigirGovernancaEmpresa`: revisar fakes de teste e contagens agregadas ao
  exigir membership
- `sdd-metricas.sh` pós-arquivamento: cópia temporária em `sdd/incrementos/`
  (promovido ao prompt 14)
