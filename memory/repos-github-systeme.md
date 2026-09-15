---
name: repos-github-systeme
description: Todos os repositórios ficam na org systeme-dev-br; os remotes antigos da org koinonia-kids só funcionam por redirect
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5faa90af-de15-434c-97b3-799034f8e278
  modified: 2026-09-05T03:07:07.919Z
---

Tudo vive na organização **`systeme-dev-br`** no GitHub (conta `messiasneto74`, `gh` já autenticado com escopo `repo`).

- `systeme-dev-br/koinonia-kids-{docs,backend,frontend,infra}` — o `docs` é o rastreador central de issues dos quatro repos.
- `systeme-dev-br/systeme-erp-docs` — backlog de histórias do Système ERP (criado em 2026-09-04).
- Também na org: `koinonia-plus-*`, `teralia-*`, `site`.

Os repositórios do Koinonia Kids foram renomeados da org antiga `koinonia-kids` (`koinonia-kids/docs` → `systeme-dev-br/koinonia-kids-docs`). As URLs antigas ainda funcionam por redirect do GitHub, mas **o `PROJECT_CONTEXT.md` local do koinonia-kids ainda diz que a org é `koinonia-kids`** — está desatualizado (snapshot de 2026-08-14, muito defasado no geral; não confiar nele sem conferir o estado remoto).

Toolchains locais: Go 1.25.9 em `~/.local/go` (instalado em 2026-09-04, symlinks em `~/.local/bin`) e Flutter 3.47.2 em `~/desenvolvimento/flutter`.
