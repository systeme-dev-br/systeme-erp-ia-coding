---
name: f1-08-papeis-permissoes-alcadas
description: "Estado do incremento SDD F1-08 (papéis, permissões e alçadas) do systeme-erp-backend — RBAC fino, Autorizador, escopo filiais"
metadata:
  node_type: memory
  type: project
  modified: 2026-09-17T13:45:00Z
---

Incremento SDD **f1-08-papeis-permissoes-alcadas** — issue F1-08 do plano de
fases (`docs/historias/plano-fases-issues.md`), histórias RBAC-PAP-004/005 e
RBAC-PERM-006/007/008 (`docs/historias/usuarios-papeis-permissoes.md`).

## Status

**FECHADO — ciclo SDD 14/14** (2026-09-17). Consolidação @ `6508775` (PR #64);
aprendizados no passo 14.

## Entrega

- Catálogo global `public.permissoes` + templates `public.papeis_template`
- Schema tenant RBAC (`papeis`, `usuario_papel`, `alcadas`,
  `solicitacoes_aprovacao`) com seed e backfill
- Serviço `Autorizador` com cache in-request (ADR-002) e MFA para papel crítico
- API `/api/v1/empresas/{id}/rbac/*` (papéis, atribuições, alçadas)
- Escopo `filial_ids` em consultas; retrofit F1-04 a F1-07 (`ExigirPermissao`)
- `GET /contexto` enriquecido (`permissoes_efetivas`, `escopo_dados`, `papeis`)

## PRs

| PR | Conteúdo |
| --- | --- |
| #57–#62 | tasks 01–06 (implementação) |
| #63 | review/QA/evidência SDD |
| consolidação | contratos vivos + histórico (branch `cursor/f1-08-consolidacao-89ba`) |

Merge implementação: `165c72c`. Merge evidência: `3214bab`.

## Contratos consolidados

- `sdd/contratos/dados/contrato.md` — catálogo consultável, baseline RBAC, trilha
- `sdd/contratos/modelo-dados-multiempresa/contrato.md` — enforcement `usuario_papel`, escopo
- `sdd/contratos/empresas/contrato.md` — `papel_id`, permissões granulares, último admin efetivo

## Fora de escopo (PRMs)

- SoD/relatório de acesso → F1-09
- UI admin de papéis
- ABAC dinâmico; delegação temporária (F1-06)

## Testes

18 SCN / 21 TST — QA APROVADO, 0 bugs.
