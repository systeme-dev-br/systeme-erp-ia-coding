---
name: f1-05-relacionamentos-ciclo-vida-empresa
description: "Estado do incremento SDD F1-05 (relacionamentos e ciclo de vida da empresa) do systeme-erp-backend — endereços/contatos/contas bancárias, papel contador reduzido, inativar/reativar/excluir empresa"
metadata:
  node_type: memory
  type: project
  originSessionId: 1b4d8295-0b6b-4788-a445-a2b83382be68
  modified: 2026-09-15T20:58:03.540Z
---

Incremento SDD **f1-05-relacionamentos-ciclo-vida-empresa** — issue F1-05
do plano de fases (`docs/historias/plano-fases-issues.md`, labels
`backend`, `mvp`, dependência F1-01, fechado), histórias
`EMP-REL-010`, `EMP-REL-011`, `EMP-CICLO-009`
(`docs/historias/gestao-empresas-multiempresa.md`). Escolhido pelo dono
2026-09-15 como próximo incremento após [[f1-04-contexto-multiempresa]].
**Backend-only** — o próprio plano de fases já classifica F1-05 sem label
`frontend` (diferente do F1-04, que precisou de decisão explícita).

## As 3 histórias

- `EMP-REL-010`: vincular contador/escritório de contabilidade (papel
  "Contador", leitura+exportação, múltiplas empresas por login,
  exportações agendadas, "portal do contador").
- `EMP-REL-011`: endereços múltiplos por tipo (fiscal/entrega/cobrança),
  contatos por papel (fiscal/financeiro/comercial/TI), contas bancárias
  (PIX, padrão de recebimento/pagamento, mascaramento).
- `EMP-CICLO-009`: inativar/reativar/excluir definitivamente uma empresa
  preservando histórico; efeitos em billing (vaga do plano) e exportação
  completa de saída.

## Decisão de escopo do dono (2026-09-15)

Perguntei especificamente sobre `EMP-REL-010` (a única das 3 com
ambiguidade real de escopo — depende de módulos inexistentes e sem eles
sobra pouco além do que o F1-04 já cobre). Resposta:
**"Reduzir a vínculo + papel"** — entrega o papel "contador"
(leitura + exportação básica) e vínculo/desvínculo com revogação
imediata; **corta** exportação agendada e "portal do contador" (dependem
de `plataforma-notificacoes.md` e do módulo fiscal, nenhum implementado).

## Cortes autônomos (mesmo padrão do F1-04, registrados como PRM sem
perguntar — dependências óbvias de módulos inexistentes)

- `EMP-CICLO-009`: corta efeitos de billing (contagem/limite de vaga do
  plano) e exportação completa de saída — dependem de
  `plataforma-assinatura-planos-billing.md`/`plataforma-importacao-exportacao.md`,
  nenhum implementado. Entrega: inativar/reativar/excluir definitivamente
  (só sem movimento, dupla confirmação, auditada).
- Papel "contador": 4º valor do vocabulário coarse já existente
  (`admin/operador/consultor` → `+contador`), não RBAC granular novo
  (RBAC fino é F1-08).
- Exclusão definitiva reaproveita o valor `removida` **já previsto no
  `CHECK` de `empresas.status` desde a migration `000001_empresas_filiais`
  (F1-01)**, nunca exercitado por nenhum usecase até agora — achado
  importante da triagem, mesmo padrão do `status='removido'` do F1-04.

## Achado técnico relevante da triagem

- `entity.Endereco` hoje é um valor único embutido em `empresas`/`filiais`
  (sem tipo, sem múltiplos endereços) — `EMP-REL-011` precisa de
  modelagem nova (tabelas `empresa_endereco`/`empresa_contato`/
  `empresa_conta_bancaria`, decisão de schema adiada para a TechSpec).
- Não existe tabela de contato nem de conta bancária ainda.

## Classificação (passo 00, commit `818b191`)

`rigor: medium` / `risco: alto` (dados bancários sensíveis,
exclusão definitiva destrutiva, papel novo com implicação de
autorização) / `autonomia: autonomo_ate_pr` / `alvo: branch`. Rota
completa: TechSpec, review e PR humanos obrigatórios; deploy dispensado.

## Progresso

- **Branch**: `sdd/f1-05-relacionamentos-ciclo-vida-empresa-plan` (de
  `main` `3713988`, pós-fechamento do F1-04), empurrada.
- **Passo 00 (triagem) FEITO** (`818b191`): `incremento.yaml`,
  `brief.md`, `impacto-contratual/empresas/contrato.md` (ALTERADO). 5
  premissas registradas (PRM-001 corte billing/exportação, PRM-002 papel
  contador não é RBAC fino, PRM-003 reaproveita status `removida`,
  PRM-004 sem UI, PRM-005 redução de `EMP-REL-010` por decisão do dono).

- **Passo 01 (PRD) FEITO** (`2c942a3`): 13 RF, 10 BR, 4 RNF. Resolvi sem
  precisar perguntar (extrapolações razoáveis, documentadas como BR/PRM):
  "bloquear operação nova" (RF-008/BR-006) = recusar os 6 endpoints de
  escrita hoje existentes (regime, filial, numeração, certificado,
  responsável fiscal, concessão/revogação de acesso); "sem movimento"
  para exclusão definitiva (BR-008/PRM-006) = ausência de filial,
  certificado, responsável fiscal, numeração reservada ou membership
  além do titular — definição operacional pelas superfícies hoje
  implementadas, revisável quando módulos fiscais/financeiros existirem;
  papel contador reaproveita as MESMAS rotas de acesso do F1-04 (sem
  rota própria) e não distingue comportamento de consultor além do
  rótulo (PRM-007, RBAC fino é F1-08 — o valor é o rótulo estável para
  políticas futuras). Endereços/contatos/contas bancárias adicionais são
  só no nível da empresa (matriz), não filial. Sem pergunta aberta
  bloqueante.

**Próximo: passo 02 (TechSpec).**
