---
name: systeme-erp-repos-codigo-f1
description: "Repos de código do Système ERP criados no bootstrap da fase F1 (backend/frontend/infra) — stack, layout, estado"
metadata: 
  node_type: memory
  type: project
  originSessionId: 80cb4e67-356f-4613-abe5-a522474b4926
  modified: 2026-09-10T17:17:54.805Z
---

**Bootstrap da fase F1 feito em 2026-09-10.** Antes disso o Système ERP só tinha
`systeme-erp-docs` (documental). O `docs/arquitetura/f0-02-repositorios.md`
descrevia 3 repos que **não existiam**. Foram criados de fato, com scaffold
mínimo compilando/lint/test verde e CI ativa (todos com default branch `main`,
clonados em `~/desenvolvimento/systeme-erp-{backend,frontend,infra}`).

## systeme-erp-backend
Go 1.22 + chi + pgx + **sqlc** + PostgreSQL 15 (**schema-per-tenant**: `public` +
`tenant_<uuid>`) + Redis. Camadas `internal/{domain,application,adapter,infra,shared}`.
- `cmd/api` — chi + graceful shutdown, `/healthz` `/readyz`, grupo `/api/v1` (F1+ registra módulos).
- `cmd/migrate` — **esqueleto**; runner real = **golang-migrate + wrapper por tenant** (aplica `public` e cada `tenant_<uuid>` via `SET search_path`), entra em F1-01.
- `sqlc.yaml`: `db/query/*.sql` → `internal/adapter/postgres/db`, engine postgresql, sql_package pgx/v5.
- `migrations/{public,tenant}/` (vazias), `Makefile`, `Dockerfile` (distroless), `.golangci.yml` (revive ativo — cuidado com unused-parameter), `config/config.yaml`, `.env.example`.
- CI `.github/workflows/ci.yml`: lint (golangci-lint + `go mod tidy` diff) + test (Postgres 15 + Redis services) + build. **Verde.**
- Lição: `.golangci.yml` tem `revive` → parâmetro de função não usado quebra o lint (o `New(log)` do router e `func(r chi.Router)` de grupo vazio pegaram; corrigido usando o log e o `_`).

## systeme-erp-frontend
Vite 5 + React 18 + TypeScript estrito + **Redux Toolkit / RTK Query** + react-router-dom 6 + Tailwind + Vitest.
- `src/services/api.ts` — `createApi` base (baseQuery com `Authorization: Bearer` do localStorage + `X-Empresa-ID`, tagTypes `Empresa`/`Filial`/`Usuario`); endpoints por feature via `api.injectEndpoints` (F1+).
- `src/store/{store,hooks}.ts`, `src/router.tsx` (App shell + Home placeholder), `src/styles/globals.css` (tailwind), alias `@/` → `src/`.
- `tsconfig.json` `types: [vite/client, vitest/globals, @testing-library/jest-dom]`; `src/vite-env.d.ts`.
- CI: `npm ci` → lint + typecheck + test + build. **Verde.** `*.tsbuildinfo` gitignored.

## systeme-erp-infra
- `docker-compose.yml` — Postgres 15, Redis 7, pgAdmin (5050, `dev@systeme.local`/`dev`), MailHog (8025/1025), healthchecks.
- `docker-compose.full.yml` — overlay que builda a API (`../systeme-erp-backend`) e o web (`../systeme-erp-frontend`).
- `scripts/reset-db.sh`, CI valida os composes. **Verde.**
- O Makefile do backend chama `../systeme-erp-infra/docker-compose.yml` (repos precisam ser irmãos).

## Fluxo SDD e F1 — MIGRADO PARA O BACKEND (decisão do dono 2026-09-10)
A partir da fase F1 o fluxo SDD (`sdd/` completo — prompts, governanca,
contratos vivos, historico, aprendizados, metricas.csv, evals — + `.compozy/`,
`AGENTS.md`, `.github/workflows/sdd-guard.yml`) **vive em `systeme-erp-backend`**,
para o `sdd-guard` enxergar o diff de código. `systeme-erp-docs` fica só com
`docs/historias` (backlog) + `docs/arquitetura` (referência) + `docs/governanca`
(registro histórico).

Migração feita via **2 PRs pareadas, aguardando merge manual do dono**:
- **`systeme-erp-backend#1`** (`chore/sdd-migration`, commit `3065013`) — traz
  `sdd/` (337 arq.) + `.compozy/` + `AGENTS.md` + `sdd-guard.yml`; `.claude/`
  no `.gitignore`; `.claude/{agents,skills,settings.json}` copiados locais
  (untracked). `sdd-guard.sh validate-policy` OK da nova localização. CI verde.
- **`systeme-erp-docs#195`** (`chore/sdd-migration-out`, commit `0591af3`) —
  remove os 341 arquivos daqui; `README.md` reescrito (4 repos do produto).
  Check `guard` vermelho esperado (remoção em massa) → merge manual.

Ordem de merge: backend#1 primeiro, depois docs#195.

**`sdd-guard` ainda não cruza repos:** frontend/infra continuam fora do worktree
que o guard audita. F1-01 kickoff decide como gatear diff cross-repo (referência
por SHA + revisão humana é a hipótese).

**PR #194** (`chore/f1-bootstrap-repos`) em `systeme-erp-docs` sincroniza
`f0-02-repositorios.md` — aguarda merge manual do dono.

**Pendências:** branch protection nos 3 repos novos (ação do dono). **Próximo:
F1-01 — Cadastro de empresas e filiais** (issue do plano de fases): 1ª migration
`public` (contas/empresas) + `tenant` (filiais), o `cmd/migrate` real, o módulo
`empresas` (domain + usecase + handler + queries sqlc), telas no frontend.
Depende de F0-06 (`modelo-dados-multiempresa`, `modelo-dados-fundacao`).
