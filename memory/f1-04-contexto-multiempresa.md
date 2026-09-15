---
name: f1-04-contexto-multiempresa
description: "Estado do incremento SDD F1-04 (contexto multiempresa) do systeme-erp-backend — troca de contexto, concessão/revogação de acesso por empresa, consolidado de grupo econômico"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1b4d8295-0b6b-4788-a445-a2b83382be68
  modified: 2026-09-15T12:11:28.666Z
---

Incremento SDD **f1-04-contexto-multiempresa** — issue F1-04 do plano de
fases (`docs/historias/plano-fases-issues.md`, labels `backend`,
`frontend`, `mvp`, dependência F1-01, já fechado), histórias
`EMP-CTX-006/007/008` (`docs/historias/gestao-empresas-multiempresa.md`).
Escolhido pelo dono 2026-09-15 como próximo incremento, seguindo a ordem
numérica do plano após o fechamento de [[f1-03-certificado-digital]].
**Escopo desta rodada: só backend** (decisão do dono) — frontend fica para
incremento separado, mesmo padrão do [[f1-01-frontend-cadastro-empresas-filiais]].

## As 3 histórias

- `EMP-CTX-006`: trocar de empresa por um seletor, sem novo login;
  permissão pode variar por empresa; lembra a última empresa usada; troca
  automática ao abrir link de outra empresa; aviso de alterações não
  salvas.
- `EMP-CTX-007` (Should, não Must): modo consolidado somente leitura de um
  grupo econômico — faturamento, contas a pagar/receber, caixa, DRE
  resumido, drill-down até o documento, eliminação intragrupo opcional.
- `EMP-CTX-008`: matriz usuário×empresa×papel; concessão por grupo inteiro
  ("incluir novas empresas"); proteção do último administrador por
  empresa; revogação imediata; auditoria.

## Achado crítico da triagem (define o corte real de escopo)

- **"Grupo econômico" não existe como entidade no modelo implementado.**
  Hoje uma empresa (tenant) = matriz + filiais no MESMO schema; não há
  conceito de várias empresas-matriz (múltiplos tenants) formando um
  grupo. As 3 histórias tratam "grupo" como se já existisse — é ambíguo o
  suficiente para ser decidido no PRD, não cortado sozinho na triagem.
- **RBAC granular é F1-08**, ainda não implementado (só existe
  `public.usuario_empresa.role`, cache com enum `admin/operador/consultor`,
  desde a migration `000001_fundacao_identidade` do F0-06/F1-01) — mesmo
  o contrato vivo `modelo-dados-multiempresa` já dizendo que a fonte de
  verdade DEVERIA ser `tenant.usuario_papel` (que não existe no código).
  As próprias histórias já citam "papéis por empresa" como dependência
  futura (`usuarios-papeis-permissoes.md`), confirmando que o escopo real
  esperado agora é mais simples que o RN sugere.
- **Autenticação real é F1-07** (ainda header `X-Usuario-ID` provisório) —
  "revogação encerra sessão imediatamente" (EMP-CTX-008 RN-03) hoje só
  significa que a checagem de acesso (já feita a cada request) passa a
  negar; não há sessão real para encerrar.
- **EMP-CTX-007 (consolidado) depende de financeiro/relatórios (F4-F9)**,
  que não existem — forte candidato a corte quase total (talvez só a
  fundação de "quais empresas entram na consolidação" + isolamento de
  leitura, sem nenhum indicador real).

Essas 4 lacunas foram registradas como PRM-001..004 no `incremento.yaml`,
**não resolvidas** na triagem — ficam para o PRD (passo 01) decidir,
seguindo o padrão de "3 decisões de escopo tomadas sem devolver pergunta"
já usado no F1-02.

## Classificação (passo 00, `2effe73`)

`rigor: large` (múltiplas capacidades + ambiguidade real de "grupo
econômico" a resolver) / `risco: alto` (autorização/segurança: matriz de
acesso, proteção do último admin, revogação) / `autonomia: autonomo_ate_pr`
/ `alvo: branch`. Rota completa: TechSpec, review e PR humanos
obrigatórios; deploy dispensado.

## Progresso

- **Branch**: `sdd/f1-04-contexto-multiempresa-plan` (de `main` `8ad424d`
  do `systeme-erp-backend`, pós-fechamento do F1-03), empurrada.
- **Passo 00 (triagem) FEITO** (`2effe73`): `incremento.yaml`, `brief.md`,
  `impacto-contratual/empresas/contrato.md` (ALTERADO, resumo — detalhado
  no PRD). `.compozy/tasks/f1-04-contexto-multiempresa/` criado vazio.

**Próximo**: aguardando "sim" do dono para o passo 01 (PRD) — que precisa
resolver as 4 perguntas de escopo acima antes de qualquer TechSpec/código.
