---
name: escopo-reduzido-honesto
description: "Quando o trabalho se revela maior que o planejado, o usuário prefere entrega reduzida e explícita a cobertura ampla e superficial"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5faa90af-de15-434c-97b3-799034f8e278
  modified: 2026-09-05T03:06:27.441Z
---

Ao descobrir, durante a implementação, que o problema é maior do que a issue supunha: reduzir o escopo ao que dá para fazer bem, documentar explicitamente o que ficou de fora, e abrir issue de acompanhamento — em vez de espalhar uma correção rasa por tudo.

**Why:** foi a escolha do usuário quando a issue de enforcement de tenant (koinonia-kids #155) se revelou inviável como planejada — só 9 de ~65 tabelas tinham `church_id` direto. Ele escolheu "reduzir ao que já é real hoje, abrir issue nova maior para o resto" em vez de um middleware genérico que daria falsa segurança. O mesmo padrão se repetiu depois na #158.

**How to apply:** parar antes de codar, mostrar os números concretos do levantamento, propor o corte, e registrar a parte não coberta na issue e no PR (nunca esconder). O corpo da issue original deve ser editado para refletir o escopo real, com comentário explicando a mudança.
