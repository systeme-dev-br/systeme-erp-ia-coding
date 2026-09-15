# systeme-erp-ia-coding

Memória persistente de assistentes de IA (Claude Code) que trabalham no
**Système ERP** — decisões, contexto de projeto e histórico entre sessões,
versionados para não se perder e para dar contexto a qualquer sessão nova.

## Estrutura

- `memory/MEMORY.md` — índice: uma linha por assunto, com link para o
  arquivo de detalhe correspondente.
- `memory/*.md` — um arquivo por assunto (incremento, decisão, achado ou
  preferência de trabalho), com frontmatter `name`/`description`/`type`.

Tipos de memória: `user` (perfil/preferências de quem pede o trabalho),
`feedback` (correções e confirmações de abordagem), `project` (estado de
incrementos e decisões em andamento) e `reference` (onde achar informação em
sistemas externos).

## Como isto é usado

Cada sessão do Claude Code neste projeto lê `memory/MEMORY.md` no início da
conversa e consulta os arquivos linkados quando relevante, para não repetir
perguntas já respondidas nem redescobrir decisões já tomadas. O conteúdo é
atualizado ao longo do trabalho — a cada ponto de validação do dono do
projeto (gate, PR, decisão), não só no fechamento de um incremento.

Este repositório não contém segredos, credenciais ou dados pessoais — só
contexto de engenharia e decisão sobre o Système ERP.
