---
name: memory-md-a-cada-validacao
description: O dono quer que MEMORY.md/memória seja atualizada a cada ponto de validação humana, não só no fechamento do incremento
metadata:
  type: feedback
  originSessionId: 80cb4e67-356f-4613-abe5-a522474b4926
  modified: 2026-09-11T13:16:48.879Z
---

Sempre que algo for enviado para a validação do dono (gate humano, PR para
revisão/merge, decisão que espera resposta dele), a memória (`MEMORY.md` +
arquivo de memória do incremento/projeto em questão) deve estar atualizada
**naquele momento** — não só quando o incremento fecha.

**Por quê:** dito explicitamente pelo dono em 2026-09-11, logo após o
fechamento do F1-01, antes de começar o incremento seguinte (frontend do
F1-01). Antes disso a memória só era atualizada em pontos de fechamento
(fim de incremento/série); ele quer visibilidade contínua a cada gate, não
só no resumo final.

**Como aplicar:** ao chegar num ponto que pede validação do dono — abrir
uma PR para ele mesclar, pedir aprovação de gate humano (especificação,
merge, etc.), ou qualquer AskUserQuestion que dependa da decisão dele —
atualizar o arquivo de memória do incremento ativo (e o índice `MEMORY.md`
se o resumo mudou) com o estado até aquele ponto, antes ou junto de pedir a
validação. Não esperar o incremento inteiro fechar para registrar.

Caminho do índice: `/home/amorim/.claude/projects/-home-amorim-desenvolvimento-systeme-erp/memory/MEMORY.md`.
