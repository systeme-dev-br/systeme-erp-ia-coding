---
name: koinonia-kids-fases-13-14
description: Estado das Fases 13 (segurança de API) e 14 (integração real do frontend) do Koinonia Kids em 2026-09-05
metadata: 
  node_type: memory
  type: project
  originSessionId: 5faa90af-de15-434c-97b3-799034f8e278
  modified: 2026-09-05T03:06:48.474Z
---

A auditoria de 2026-09-04 encontrou que (1) nenhuma rota de negócio do backend validava token ou `church_id`, e (2) o app Flutter usava só repositórios em memória, com a tela de login sem nenhuma chamada real. Isso motivou as Fases 13 e 14.

**Mesclado até 2026-09-05:** F13-01 (middleware de autenticação), F13-02 (tenant nas 5 rotas com `church_id` na query), F13-03 (CORS), F13-04 parcial (tenant por recurso em `child` e `incident`), F14-01 (cliente Dio), F14-02 (login/token/refresh reais), F14-03 (contexto de igreja do usuário autenticado).

**Aberto:** issue #158 (tenant nos ~20 módulos restantes — `checkin` sozinho tem ~45 rotas) e issues #163 a #170 (F14-04 a F14-11: trocar 21 repositórios `InMemory` por `Http`, ~180 métodos de interface; `checkin` tem 61 métodos e `family_app` 26). PR de docs #162 e PR de design #129 ficaram abertos aguardando merge.

**Why:** o `plano-fases-issues.md` só reflete isso depois que o PR #162 for mesclado; até lá, o estado real está fragmentado entre issues e PRs.

**How to apply:** antes de retomar, conferir no GitHub o que já foi mesclado — o usuário mescla manualmente e pode ter avançado entre sessões. Ver [[achados-auditoria-sem-issue]] para os problemas que não viraram issue.
