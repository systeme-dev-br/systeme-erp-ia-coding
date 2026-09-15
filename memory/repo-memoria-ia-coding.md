---
name: repo-memoria-ia-coding
description: O diretório local de memória do assistente também é espelhado (publicado) no repositório systeme-dev-br/systeme-erp-ia-coding
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1b4d8295-0b6b-4788-a445-a2b83382be68
  modified: 2026-09-15T02:04:09.284Z
---

O diretório de memória local (`MEMORY.md` + arquivos de detalhe) foi
publicado, a pedido do dono em 2026-09-14, no repositório
**`systeme-dev-br/systeme-erp-ia-coding`** (público), primeiro commit
`c4e9b9f`. Estrutura lá: `README.md` + `memory/MEMORY.md` + `memory/*.md`.

**Desde 2026-09-14, a sincronização é automática via hook local.** Um hook
`PostToolUse` (matcher `Write|Edit`) em `~/.claude/settings.json` roda
`~/.claude/scripts/sync-memory-to-github.sh` (`async: true`) a cada escrita
de arquivo — o script só age se o arquivo tocado for um `.md` dentro de
`~/.claude/projects/-home-amorim-desenvolvimento-systeme-erp/memory/`
(no-op silencioso para qualquer outro arquivo/projeto). Quando age, ele
copia todos os `.md` da memória para o checkout persistente em
`~/.claude/repos/systeme-erp-ia-coding` (clonado sob demanda se ainda não
existir) e faz `git add`+`commit`+`push` só se houve mudança de conteúdo.
Log de cada execução em `~/.claude/scripts/sync-memory-to-github.log`.

A memória local continua sendo a fonte de verdade viva; o repo é o espelho
publicado, agora mantido em sincronia sem intervenção manual.

Setup do hook validado em 2026-09-15 (prova de disparo real via harness,
não só teste manual do script).
