---
name: achados-auditoria-sem-issue
description: Três problemas reais achados na auditoria do Koinonia Kids que não viraram issue e não estão registrados em lugar nenhum
metadata: 
  node_type: memory
  type: project
  originSessionId: 5faa90af-de15-434c-97b3-799034f8e278
  modified: 2026-09-05T03:06:58.655Z
---

Achados da auditoria de 2026-09-04 (Koinonia Kids) que **não** viraram issue no GitHub e se perdem se ninguém registrar:

1. **`koinonia-kids-infra` tem a `main` vazia.** Os 3 PRs de infra (docker-compose, Keycloak) foram todos mesclados em `develop`; a `main` só tem o template inicial de maio/2026. Quem clonar `main` não sobe o ambiente local.
2. **Zero tags em todos os 4 repositórios**, apesar de 259 PRs mesclados — o `AGENTS.md` exige que `main` corresponda à última tag.
3. **Cobertura de teste real do backend: 34,9%** (medida com `go test -cover`), contra os 80% documentados como meta. O CI roda `go test ./...` sem `-cover` e sem threshold, então nada verifica isso. `adapters/http` e `adapters/postgres` de vários módulos (`imports`, `incident`, `totem`, `transport`, `offline`) têm 0%.

**Why:** a auditoria gerou issues só para os dois problemas estruturais (Fases 13 e 14); estes três ficaram apenas no relato em conversa.

**How to apply:** se o usuário pedir "o que ainda falta", mencionar estes três. Não abrir issue por conta própria — ele decide o que priorizar. Ver [[koinonia-kids-fases-13-14]].
