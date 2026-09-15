---
name: f1-04-contexto-multiempresa
description: "Estado do incremento SDD F1-04 (contexto multiempresa) do systeme-erp-backend — troca de contexto, concessão/revogação de acesso por empresa, consolidado de grupo econômico"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1b4d8295-0b6b-4788-a445-a2b83382be68
  modified: 2026-09-15T19:48:07.671Z
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

## Decisões de escopo FECHADAS pelo dono (2026-09-15)

- **Grupo econômico: cortado.** Sem noção de grupo — troca de contexto e
  concessão de acesso sempre por empresa individual. Registrado como
  decisão pendente para um incremento futuro em
  `docs/historias/plano-fases-issues.md` §7, PR **`systeme-erp-docs#196`**
  ("adia grupo econômico do F1-04 para incremento futuro") — aberto, não
  mesclado ainda.
- **EMP-CTX-007 (consolidado): cortada inteiramente.** Dependia do grupo
  (cortado) e de módulos financeiros/relatórios inexistentes (F4-F9).
- Consequência: **F1-04 entrega só `EMP-CTX-006` (troca de contexto) e
  `EMP-CTX-008` (concessão/revogação de acesso por empresa individual)**.
  Rigor reclassificado de `large` para `medium` (a ambiguidade que
  justificava `large` foi resolvida); risco continua `alto`
  (autorização/segurança).
- Commit `bffb680` na branch `sdd/f1-04-contexto-multiempresa-plan`
  fechou essas decisões no `brief.md`/`incremento.yaml`/impacto contratual.

## Achado crítico da triagem (contexto por trás das decisões acima)

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

## Classificação (passo 00, `2effe73`, reclassificado em `bffb680`)

`rigor: medium` (2 capacidades, sem mais a ambiguidade de "grupo
econômico") / `risco: alto` (autorização/segurança: matriz de acesso,
proteção do último admin, revogação) / `autonomia: autonomo_ate_pr` /
`alvo: branch`. Rota completa: TechSpec, review e PR humanos obrigatórios;
deploy dispensado.

## Progresso

- **Branch**: `sdd/f1-04-contexto-multiempresa-plan` (de `main` `8ad424d`
  do `systeme-erp-backend`, pós-fechamento do F1-03), empurrada.
- **Passo 00 (triagem) FEITO** (`2effe73`, decisões de escopo fechadas em
  `bffb680`): `incremento.yaml`, `brief.md`, `impacto-contratual/empresas/
  contrato.md` (ALTERADO). `.compozy/tasks/f1-04-contexto-multiempresa/`
  criado vazio.

- **Passo 01 (PRD) FEITO** (`1933240`): 7 RF (listar empresas p/ seletor,
  registrar/consultar última empresa usada, auditar troca de contexto,
  conceder acesso por empresa/filial com papel, revogar acesso, impedir
  remover último admin, auditar concessão/revogação), 7 BR, 4 RNF. Sem
  pergunta aberta bloqueante (ambiguidades já fechadas na triagem).
  Fora de escopo explícito: RBAC fino, sessão real, convite de usuário
  novo, notificação real ao usuário revogado, "perfil contador" com
  preset de papel, UI inteira. PRM-006/007 novas (preferência simples por
  usuário para "última empresa"; concessão pressupõe usuário já
  existente). `fase: especificacao`, `metricas.data_especificado`
  registrada.

- **Correção no PRD antes da TechSpec** (`cdc1fcd`): achado ao explorar o
  código — `public.usuario_empresa` não tem escopo por filial, só por
  empresa (tenant) inteira. RF-004/005 ajustados de "empresa ou filial"
  pra só "empresa"; registrado em "Fora de escopo".
- **Passo 02 (TechSpec) FEITO** (`87511d3`): estende `usuarios`/
  `usuario_empresa` em vez de tabelas novas — coluna
  `usuarios.ultima_empresa_id` (**ADR-001**) e revogação como soft-delete
  reaproveitando `status='removido'` (já no `CHECK` desde a fundação,
  nunca usado até agora) (**ADR-002**). Proteção do último admin é
  checagem síncrona na aplicação (não trigger), cobrindo tanto revogação
  quanto rebaixamento de role via concessão — achado importante
  registrado no fluxo de dados/riscos. 4 endpoints novos: `PUT
  /empresas/{id}/contexto`, `GET /contexto`, `POST/DELETE
  /empresas/{id}/acessos[/{usuarioId}]`. `ListarEmpresas` (RF-001)
  ALTERADO pra marcar `ultima_usada`.

- **Passo 03 (plano+tasks) FEITO** (`cb2c790`): `execucao.md` (base `main`
  `8ad424d`, ordem schema/repo→troca de contexto→concessão/revogação→
  HTTP→regressão), `INDEX.md`, **5 tasks**, **2 `feature/*`** com **14
  SCN/TST** no total (4+9+1 por comportamento — task_02 troca de
  contexto, task_03 concessão/revogação, task_01 isolamento). Impacto
  contratual detalhado (5 comportamentos, Gherkin completo).
  `compozy tasks validate` → all valid (5); `compozy sync` ok.
  `fase: planejamento`.

- **Passo 04 (auditoria) FEITO** (`e782da7`, agente
  `cz-auditor-especificacao` em contexto isolado): **PRONTO**. Confirmou
  que a correção de escopo do PRD (empresa, não filial) está propagada
  sem resíduo em todos os artefatos; ADR-001/ADR-002 refletidos
  coerentemente nas tasks; proteção do último admin cobre concessão que
  rebaixa E revogação/autorrevogação (não só prosa); matriz do INDEX.md
  fecha sem SCN órfão. Único achado: 1ª versão do relatório usou notação
  de faixa (`TST-001..004`) — corrigido para IDs individuais.
  `status: especificado`, `fase: auditoria`.

**Gate humano de especificação APROVADO** pelo dono (2026-09-15, "sim" no
chat, registrado em `gates.especificacao` do `incremento.yaml`) —
implementação liberada.

- **Passo 06/task_01 FEITO** (`0f602bf`): schema/portas/persistência.
  Migration `public/000002_contexto_multiempresa` (`usuarios.
  ultima_empresa_id`, ADR-001); queries sqlc `ConcederMembership`
  (upsert)/`RevogarMembership` (soft-delete `execrows`)/`ContarAdminsAtivos`/
  `DefinirUltimaEmpresa`/`UltimaEmpresaDoUsuario`; `port.MembershipRepo`/
  `port.UsuarioRepo` estendidas e implementadas no repo Postgres. 5 testes
  de integração contra Postgres real, incluindo SCN-014 (isolamento entre
  duas empresas). Precisou atualizar 3 duplas de teste (`fakeMemberships`,
  `semAcesso`, `acessoEmpresa`) que implementam essas portas, para não
  quebrar a compilação ao estender a interface — padrão a repetir nas
  próximas tasks que tocarem `MembershipRepo`/`UsuarioRepo`. `go build`/
  `go vet`/`go test`/`make lint` e `sdd-guard.sh pre-complete` verdes.
  `task_01.md`/`INDEX.md` marcados `completed`, `compozy tasks
  validate`/`sync` ok. Commit empurrado para
  `sdd/f1-04-contexto-multiempresa-plan`.

- **Passo 06/task_02 FEITA** (`3ea4e23`): usecase de troca de contexto.
  `TrocarContexto` (`garantirAcesso`/BR-005 → captura empresa anterior →
  `DefinirUltimaEmpresa` → trilha `troca_contexto` best-effort com
  `empresa_anterior` nos metadados) e `ConsultarContexto`. `ListarEmpresas`
  passou a devolver `EmpresaListada` (`entity.Empresa` + `ultima_usada`) —
  mudança de assinatura sem quebrar o handler HTTP existente, que só
  repassa a lista pra `escreverJSON` sem acessar campos. 4 testes com
  fakes (SCN-001..004/TST-001..004) verdes; precisou de um novo
  `fakeUsuarios` em `fakes_test.go` (não existia ainda no pacote
  `usecase_test`). `go build`/`vet`/`test`/`make lint` e `sdd-guard.sh
  pre-complete` verdes.

- **Passo 06/task_03 FEITA** (`8fedd70`): concessão/revogação de acesso.
  `garantirAdmin` (mesmo padrão de `garantirAcesso`/`garantirTitular`);
  `garantirNaoRemoveUltimoAdmin` compartilhado por rebaixamento e
  revogação/autorrevogação; `validarRole`. `ConcederAcesso` resolve
  usuário por e-mail (`*ErroUsuarioNaoEncontrado` se não existir, mesmo
  padrão de tipo de erro dedicado do F1-03) e faz upsert de papel;
  `RevogarAcesso` exige confirmação explícita (`confirmacao_obrigatoria`,
  mesmo padrão do F1-03) e devolve `*ErroAcessoNaoEncontrado` se já não
  ativo. Ambos auditam via trilha best-effort, metadata só com
  `usuario_afetado`/`papel` (nunca e-mail bruto). 9 testes com fakes
  (SCN-005..013/TST-005..013) verdes. `go build`/`vet`/`test`/`make lint`
  e `sdd-guard.sh pre-complete` verdes.

- **Passo 06/task_04 FEITA** (`d5e681d`): handlers HTTP. 4 rotas novas
  (`PUT`/`GET .../contexto`, `POST`/`DELETE .../acessos[/{usuarioId}]`),
  handlers finos (regra já em task_02/03). Slugs dedicados
  `usuario_nao_encontrado`/`acesso_nao_encontrado` mapeados em
  `erros.go`; `sem_acesso_a_empresa` (404, não 403) implementado como
  override só na rota de troca de contexto, decisão explícita da
  TechSpec. **Achado real corrigido**: `ConsultarContexto` (task_02) não
  cruzava com o acesso atual — uma última-empresa cujo acesso foi
  revogado continuava sendo devolvida como válida por `GET /contexto`;
  corrigido fechando um risco que a própria TechSpec já descrevia como
  mitigado (não estava, na prática). 12 testes de contrato HTTP com
  fakes dedicados (`fakeUsuariosCtx`/`fakeMembershipsCtx`). README
  atualizado (tabela de rotas F1-01 a F1-04). `go build`/`vet`/`test`/
  `gofmt`/`make lint` e `sdd-guard.sh pre-complete` verdes.

- **Passo 06/task_05 FEITA** (`a871d29`): regressão transversal. `sqlc
  generate` sem diff; `make build`/`make test -race -count=1` (suíte
  completa, Postgres real)/`make lint` verdes; `sdd-guard.sh
  scan-secrets` contra os 43 arquivos do incremento sem achado; `gofmt
  -l` vazio. Matriz de rastreabilidade fechada: 14 SCN/14 TST, todos
  ligados a teste automatizado, sem órfão. Nenhum contrato vivo alterado
  (fica para o passo 13/consolidação).

**TODAS AS 5 TASKS DE IMPLEMENTAÇÃO FEITAS** (`0f602bf`, `3ea4e23`,
`8fedd70`, `d5e681d`, `a871d29`, todas na branch
`sdd/f1-04-contexto-multiempresa-plan`).

- **Passo 07 (review) FEITO** (`4a85303`, agente `cz-revisor-implementacao`
  em contexto isolado, Evidence SHA `a871d29`): **APROVADO**. Rerodou
  build/vet/`test -race`/lint/scan-secrets de forma independente — tudo
  verde. Confirmou os 3 caminhos da proteção do último admin, isolamento
  real entre tenants, ausência de e-mail/`permissoes` em resposta/trilha.
  **1 achado P2 não bloqueante** (`REVIEW-001`): a correção de
  `ConsultarContexto` feita durante task_04 (fecha o risco RSK-002 da
  TechSpec — última empresa "órfã" após revogação) é legítima e está no
  diff, mas não tem teste dedicado ao caminho
  troca→revogação→consulta. **Decisão registrada**: aceito como risco
  residual documentado, sem corrigir agora — corrigir exigiria reexecutar
  o passo 07 (regra do prompt 09: mudança de código após o Evidence SHA
  do review invalida o review), e o impacto é baixo (nenhuma rota usa
  `ConsultarContexto` para autorizar; todas rechecam acesso a cada
  chamada). `fase: review`.

- **Passo 08 (QA) FEITO** (`baded55`, agente `cz-qa` em contexto isolado,
  mesmo Evidence SHA `a871d29`): **APROVADO**. Validou como consumidor
  real — API HTTP de verdade contra Postgres/Redis reais via
  `compose-up`, 3 empresas/tenants provisionados de fato. **14/14 SCN
  passaram** + o cenário extra sugerido pelo achado P2 do review
  (troca→revogação→consulta) também passou, confirmando na prática que
  `ConsultarContexto` não expõe uma última-empresa cujo acesso foi
  revogado. **0 bugs.** Suíte automatizada e `scan-secrets` reexecutados
  de forma independente, verdes. Ambiente derrubado ao final, `.env`
  restaurado ao template, nada deixado rodando.
- **`data_validado` marcado** (`sdd-metricas.sh`) e transição aplicada
  via `sdd-fluxo.sh --run`: `status: validado`, `fase: pr` (commit
  `fec0a96`).

**Review e QA aprovados, 0 P0/P1 aberto (o único achado, P2, foi aceito
como risco residual documentado e depois confirmado sem regressão pelo
QA).**

- **Passo 10 (preparar PR) FEITO** (`34a8f4e` pacote +
  **`systeme-erp-backend#15` ABERTA** + `e211155` registra gate/PR no
  `incremento.yaml`). `pr-package.md`/`pr-body.md` completos. Gate
  `especificacao` já cobria "abertura de PR" no escopo — usado como
  autorização, sem pedir "sim" extra pontual para o `gh pr create` em si
  (autonomia `autonomo_ate_pr`). `gates.pr.gate_humano.status: aprovado`,
  `pr.status: aberto`, `pr.numero: 15`. CI disparada (`build`/`guard`/
  `lint`/`test`), pendente no momento do registro.

**PR #15 aberta, aguardando CI e revisão remota. Merge NUNCA é feito pelo
agente — passo 11 é gate humano.** Próximo: acompanhar CI e, quando o
dono validar (revisão remota + merge manual), seguir para consolidação
(passo 13).
