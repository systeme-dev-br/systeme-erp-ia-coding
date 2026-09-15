---
name: f1-02-parametros-identidade-empresa
description: "Estado do incremento SDD F1-02 (parâmetros e identidade da empresa) do systeme-erp-backend — regime tributário com vigência, numeração de documentos, identidade visual, calendário de dias úteis"
metadata: 
  node_type: memory
  type: project
  originSessionId: 86c7204f-466c-447b-a41b-0a713b18d020
  modified: 2026-09-15T00:30:31.551Z
---

Incremento SDD **f1-02-parametros-identidade-empresa** — issue F1-02 do
plano de fases (`docs/historias/plano-fases-issues.md`, label `backend`),
depende só de F1-01 (já fechado, ver [[f1-01-cadastro-empresas-filiais]] e
[[f1-01-frontend-cadastro-empresas-filiais]]). Escolhido pelo dono
2026-09-13 como próximo incremento, seguindo a ordem numérica do plano.

## As 4 histórias que compõem a issue

- `EMP-PARAM-003`: regime tributário (CRT) com **vigência por período**
  (histórico) + parâmetros fiscais gerais (IPI/ICMS, `pCredSN`,
  arredondamento). Hoje (F1-01) `empresas.regime_tributario`/
  `filiais.regime_tributario` é um valor único sem histórico — este
  incremento precisa migrar isso preservando o contrato vivo já
  consolidado (checklist de ativação, `regime_efetivo` de filial).
- `EMP-PARAM-005`: numeração/séries de documentos por empresa, com reserva
  **atômica** e sequência **fiscal estrita** (sem lacunas).
- `EMP-LOGO-012`: identidade visual (logo/cor/textos) para DANFE/propostas/
  relatórios, com upload validado e sanitizado (recusar SVG malicioso).
- `EMP-CAL-013`: calendário de feriados + horário de funcionamento por
  empresa/filial, para cálculo de "próximo dia útil" (consumo futuro por
  financeiro/cobrança/CRM, que ainda não existem).

## Classificação (passo 00, `40c239f`)

`rigor: large` (4 domínios de regra de negócio distintos + decisão
arquitetural não trivial na migração do regime) · `risco: medio` (dado
fiscal-adjacente, mas **sem emissor fiscal real ainda** — NF-e/NFC-e são
F2+/F5+, então nenhum efeito de emissão em produção) · `autonomia:
autonomo_ate_pr` (authority checker já reconhece o gate `especificacao`,
PR #162/AG-001) · `alvo: branch`. Rota completa: 00 → 01 → 02 → 03 → 04 →
06 → 07 → 08 → 10 → 11 → 13 → 14 (TechSpec e review obrigatórios pela
migração + decisão técnica relevante).

**PRM-001 crítica**: arquitetura de migração do regime tributário
(histórico de vigência) fica pro TechSpec, sem quebrar o que o F1-01 já
entrega. **PRM-002**: sem emissor fiscal real ainda, por isso risco médio
não alto. **PRM-003**: issue é só backend — sem UI nesta entrega (mesmo
padrão do F1-01, frontend seria incremento separado depois). **PRM-005**:
candidatos a corte de escopo (reinício anual, importar calendário futuro,
marca d'água, textos padrão reutilizáveis) — decisão adiada pro PRD, não
cortada ainda.

## Progresso

- **Branch**: `sdd/f1-02-parametros-identidade-empresa-plan` (de `main`
  `931f4af` do `systeme-erp-backend`), empurrada.
- **Passo 00 (triagem) FEITO** (`40c239f`): `incremento.yaml`, `brief.md`,
  `impacto-contratual/empresas/contrato.md` (ALTERADO, 4 comportamentos
  novos). `.compozy/tasks/f1-02-parametros-identidade-empresa/` criado
  vazio.
- **Passo 01 (PRD) FEITO** (`cbcf554`): 13 RF, 13 BR, 6 RNF, organizados
  pelas 4 capacidades. Decisões de escopo tomadas (não deixadas como
  pergunta aberta): reinício anual/marca d'água/textos padrão/número
  inicial de migração cortados (PRM-008); só numeração genérica nasce
  agora, séries fiscais específicas ficam para F5+ (PRM-009); alerta de
  troca de regime só expõe dado estruturado via API, sem UI (PRM-010,
  reforça PRM-003); identidade visual sem histórico de versões — módulo
  futuro de documento decide como preservar aparência (PRM-010); numeração
  e identidade visual só por matriz nesta entrega, filial herda (PRM-011).
  Sem reclassificação — `rigor: large`/`risco: medio` do passo 00
  confirmados.

- **Passo 02 (TechSpec) FEITO** (`3c405b3`): arquitetura por capacidade
  (regime/numeração/identidade visual/calendário), endpoints REST novos sob
  `/empresas/{id}/...` mantendo `PATCH /empresas/{id}` como atalho de
  compatibilidade para regime simples. **4 ADRs**: **ADR-001** (regime migra
  de coluna única para tabela append-only `empresa_regime_historico`, sem
  `vigencia_fim` armazenado — calculado por `LEAD()`; colunas antigas
  `regime_tributario` removidas após backfill; checklist/`regime_efetivo`
  passam a ler via função, comportamento observável preservado); **ADR-002**
  (feriados nacionais fixos+móveis calculados localmente via algoritmo de
  Páscoa, sem serviço externo nem tabela); **ADR-003** (logo como `BYTEA` no
  schema do tenant, sem introduzir object storage novo no `systeme-erp-infra`);
  **ADR-004** (reserva de número via `UPDATE ... RETURNING` single-statement,
  não `SELECT FOR UPDATE` nem sequência nativa). `make test` já roda com
  `-race` — usado como critério de aceite da concorrência real de RF-007.
  Nova migration única `000002_parametros_empresa` cobre as 4 capacidades,
  toda tabela com `empresa_id NOT NULL` (convenção herdada da série F0-06).

- **Passo 03 (plano+tasks) FEITO** (`1268e37`): `execucao.md` (base `main`
  `931f4af`, ordem migration→4 capacidades independentes→consolidação,
  RSK-001 backfill de regime, RSK-002 atomicidade sob carga real, RSK-003
  SVG malicioso), `INDEX.md`, **6 tasks** (task_01 migration/schema+backfill,
  task_02 regime, task_03 numeração, task_04 identidade visual, task_05
  calendário, task_06 consolidação+verificação manual), **4 `feature/*`**
  com **22 SCN/TST** no total (7+6+4+5 por capacidade). `compozy tasks
  validate` → all valid (6) só depois de corrigir 1 achado: `complexity:
  large` não é enum válido do schema v2 (valores reais são `low/medium/
  high/critical`, não `low/medium/large` como a nota do ciclo
  f1-01-frontend registrou — corrigido para `high` na task_02; registrar
  essa correção no passo 14 deste ciclo). Range agregado nas tasks usa
  "SCN-001 a SCN-022" (com "a", não ".."), evitando de propósito o gotcha
  de regex do guard já promovido nos prompts 07/08.

- **Passo 04 (auditoria) FEITO** (`c65445f`, agente `cz-auditor-especificacao`
  em contexto isolado): **PRONTO**, sem P0/P1/P2, só 2 P3 sem impacto
  (cenários Gherkin extras sem tag, notação `RF-001..004` em prosa —
  confirmado que o gotcha do guard é só pra faixa agregada `SCN-*`/`TST-*`
  na rastreabilidade das tasks, não pra `RF-*`/`BR-*` em prosa). Confirmou
  empiricamente o enum real de `complexity` (`low/medium/high/critical`).
  `status: especificado`/`fase: auditoria`.
- **Gate humano de especificação APROVADO** (`a8ccf61`, dono, 2026-09-13):
  escopo cobre as 6 tasks (implementação, testes, commits, push, abertura
  de PR); merge continua sujeito a gate próprio.
- **`task_01` travou na execução** (fork), motivo real e legítimo: o
  TechSpec/ADR-001 originais mandavam remover
  `empresas.regime_tributario`/`filiais.regime_tributario` na mesma
  migration do backfill, mas 12 arquivos Go (queries `sqlc` geradas, repos,
  usecases `ativacao.go`/`empresa.go`/`filial.go`, handlers/DTOs) ainda
  leem/escrevem essas colunas — e `task_01` foi escopada sem tocar código
  de aplicação (isso é `task_02`). Removê-las ali quebraria
  `sqlc generate`/build. O fork corretamente parou em vez de decidir
  sozinho ou forçar.
- **Revisão do plano 2 aplicada** (`c253853`, dono aprovou a resolução):
  padrão **expand/contract**. `task_01` (migration `000002_parametros_empresa`)
  passa a criar as 6 tabelas + backfill de `empresa_regime_historico` **sem
  remover** as colunas antigas. `task_02` ganha, ao final, a migration
  `000003_remove_regime_tributario_colunas` (remove as 2 colunas, só depois
  que `sqlc`/repos/usecases/handlers já leem via `RegimeRepo`; `down.sql`
  reconstrói a partir do histórico). Editados: `_techspec.md` (§Componentes,
  §Dados e migração, §Sequenciamento), `adrs/adr-001-...md` (seção
  "Revisão" acrescentada, decisão original mantida), `task_01.md` (perde
  subtask de remoção/reconstrução), `task_02.md` (ganha a segunda
  migration + critério de sucesso via `grep`), `execucao.md` (revisão do
  plano 1→2, tabelas de arquivos/contratos/provas atualizadas, linha nova
  em "Revisões do plano"). `impacto-contratual/empresas/contrato.md`
  **não mudou** (comportamento observável final é o mesmo, só o
  sequenciamento interno de migrations mudou). Sem mudança de classificação.
  `compozy tasks validate` → all valid (6) depois da revisão.

- **Reauditoria 1 FEITA** (`dc433a9`, `cz-auditor-especificacao` de novo, contexto
  isolado): **PRONTO**. Confirmou que o expand/contract resolve o conflito
  original; achou e corrigiu direto um gap real (AUD-001, P2): `task_02.md`
  cobria os consumidores do fluxo de *vigência* do regime mas não os do fluxo
  de *criação* (`POST /empresas`/`POST .../filiais`, que também leem/escrevem
  `regime_tributario`) — corrigido em `2faadb8`. `reauditorias: 1`. Avaliação
  independente: retomar `task_01` **não precisa** de nova aprovação humana
  (sequenciamento técnico dentro do mesmo escopo já aprovado).
- **`task_01` EXECUTADA E FECHADA** (`a319bbd`): migration
  `migrations/tenant/000002_parametros_empresa.{up,down}.sql` — as 6 tabelas
  (`empresa_regime_historico`, `empresa_tipo_documento`,
  `empresa_tipo_documento_ajuste`, `empresa_identidade_visual`,
  `empresa_calendario_feriado`, `empresa_horario_funcionamento`) + backfill de
  `empresa_regime_historico` a partir do `regime_tributario` legado, **sem**
  remover as colunas antigas (expand/contract intacto). Teste de integração
  novo (`internal/infra/migracao/parametros_empresa_test.go`,
  `testdb.BancoEfemero`) cobre tenant novo, backfill sobre dado legado
  (empresa+filial, usuário titular via `public.usuario_empresa`) e down
  simétrico sem tocar as colunas antigas. Também validado contra o Postgres
  real de dev: 2 tenants pré-existentes na versão 1 migraram limpo para 2,
  reaplicação idempotente.
  **3 achados técnicos corrigidos na execução** (SQL da TechSpec só quebrou ao
  rodar de verdade — corrigidos diretamente, sem reabrir plano):
  1. `PRIMARY KEY` de `empresa_horario_funcionamento` usava `COALESCE(...)` —
     Postgres não aceita expressão em `PRIMARY KEY`; trocado por `id`
     surrogate + `UNIQUE INDEX` sobre a expressão.
  2. `CHECK` exigindo `aliquota_simples` para Simples Nacional bloqueava o
     próprio backfill (empresa legada do F1-01 pode ter Simples sem alíquota
     — campo não existia antes); `CHECK` removido, RF-004/BR-004 seguem como
     validação de aplicação na `task_02` (mesmo padrão do desvio já registrado
     em `migrations/tenant/000001`).
  3. 2 asserções pré-existentes de `migracao_test.go` presumiam 1 única
     migration de tenant (versão 1); atualizadas pra versão 2 preservando a
     intenção de cada cenário — qualquer migration nova exigirá o mesmo ajuste.
  Também corrigido um erro de processo do próprio executor: faltava a
  transição `status: em_execucao`/`fase: implementacao` no `incremento.yaml`
  ao iniciar a task (prompt 06 "Seleção") — sem ela, o `sdd-guard.sh
  pre-implement` reprova qualquer mudança de código como "artefato mudou
  depois da auditoria".

- **`task_02` EXECUTADA E FECHADA** (`357b9c5`): regime tributário com
  vigência por período completo, dois fluxos de consumo migrados:
  - **Fluxo de vigência**: `entity.RegimeVigencia`/`ImpactoRegime`
    (`entity/regime.go`), `port.RegimeRepo` (`Vigente`/`Definir`/`Historico`,
    em `port/port.go`), `postgres.RegimeRepo` (`adapter/postgres/regime_repo.go`
    — `Vigente(filialID)` cai pra matriz via `RegimeVigenteEm` sem
    fallback embutido, chamado 2x quando necessário), `usecase.DefinirRegime`/
    `HistoricoRegime` (`usecase/regime.go` — valida `alicota_simples_obrigatoria`,
    `motivo_obrigatorio` em retroativo, monta `impacto` com
    `muda_vocabulario_fiscal`), handlers `PUT/GET .../regime[/historico]`
    (matriz e filial, `handler/regime.go`+`regime_dto.go`).
  - **Fluxo de criação** (achado da reauditoria, AUD-001): `POST
    /empresas`/`POST .../filiais` continuam aceitando `regime_tributario` no
    corpo (contrato intacto) mas passam a delegar pra `RegimeRepo.Definir`
    após criar — tocou `entity.Empresa/Filial` (campo mantido, populado por
    `popularRegime`/`popularRegimeFilial` em vez de coluna), `port.NovaEmpresa/
    NovaFilial` (campo `RegimeTributario` removido, não é mais gravado no
    Inserir), `usecase/{empresa.go,filial.go}`, `postgres/{empresa_repo.go,
    filial_repo.go,mapper.go}`. `ativacao.go`/`checklist` e `regime_efetivo`
    de filial passam a ler via `RegimeRepo.Vigente` — RNF-005 preservada
    (testes SCN-008/009/010 do F1-01 continuam verdes sem alteração, mais um
    teste de regressão explícito novo).
  - Ao final, com `grep -rn "regime_tributario" internal/ db/` mostrando só
    nomes de campo JSON (contrato) e código de teste que manipula versões
    antigas do schema de propósito, aplicada
    `migrations/tenant/000003_remove_regime_tributario_colunas` — testado
    contra 2 tenants reais de dev (`v2→v3`, sem erro).
  - **6 novos testes de integração contra Postgres real** (`SCN-001` a
    `SCN-006`, `internal/adapter/postgres/regime_repo_test.go`) + 1 de
    regressão explícita.
  - **Gap real do F1-01 corrigido**: `entity.RegimesValidos` não tinha
    `simples_excesso_sublimite` (CRT 2) apesar de BR-001 e do `CHECK` da
    migration já exigirem — adicionada a constante e incluída no mapa.
  - **Desvio de organização (não material)**: `RegimeRepo` foi pra
    `port/port.go` existente (não um `port/regime.go` separado) e o adapter
    chama-se `postgres/regime_repo.go` (não `regime.go`), mesmo padrão de
    nomenclatura de `empresa_repo.go`/`filial_repo.go` já vivo no pacote.
  - `make lint`/`build`/`test` (suíte completa, `-race`) verdes;
    `sdd-guard.sh pre-implement`/`pre-complete task_02` → `OK`; `compozy
    tasks validate`/`sync` ok.
  - 2 testes pré-existentes de `migracao_test.go` e o de
    `parametros_empresa_test.go` precisaram de ajuste adicional pra
    conviver com a 3ª migration de tenant (`Subir`/`Baixar` com contagem de
    passos certa) — nova função `migracao.Subir` (contrário de `Baixar`)
    adicionada ao pacote.

- **`task_03` EXECUTADA E FECHADA** (`b2a9f50`): numeração de documentos com
  reserva atômica (RF-005..008, BR-005..008, RNF-001, ADR-004). `entity.TipoDocumento`
  (`entity/numeracao.go`), `port.NumeracaoRepo` (`Criar`/`Listar`/`PorNome`/
  `Reservar`/`AjustarProximoNumero`, em `port/port.go`), `postgres.NumeracaoRepo`
  (`adapter/postgres/numeracao_repo.go` — `Reservar` é um único
  `UPDATE empresa_tipo_documento SET proximo_numero = proximo_numero + 1 ...
  RETURNING` sobre `(empresa_id, nome)`, sem `SELECT` prévio nem lock
  explícito), `usecase.{CriarTipoDocumento,ListarTiposDocumento,ReservarNumero,
  AjustarProximoNumero}` (`usecase/numeracao.go`), handlers `POST/GET
  .../tipos-documento`, `POST .../tipos-documento/{nome}/reservar`, `PATCH
  .../tipos-documento/{nome}` (`handler/numeracao.go`+`numeracao_dto.go`).
  **Teste de concorrência real obrigatório**: 50 goroutines chamando
  `Reservar` simultaneamente contra Postgres real (`-race`) — conjunto de
  números resultante exatamente contíguo, sem duplicidade nem lacuna
  (`TestSCN_010_ReservasConcorrentes_semColisaoNemLacuna`,
  `numeracao_repo_test.go`). Demais cenários `SCN-008`/`009`/`011`/`012`/`013`
  também cobertos (criar/listar tipo, motivo obrigatório no ajuste,
  retrocesso bloqueado sob sequencial estrito, nome duplicado recusado).
  **2 achados técnicos pontuais corrigidos na execução** (mesmo padrão das
  tasks anteriores — sem reabrir plano): (1) `RegimeRepo`/adapter seguiram o
  mesmo desvio de organização já aceito na `task_02` (`port.go` existente,
  `numeracao_repo.go` em vez dos nomes literais `port/numeracao.go`/
  `postgres/numeracao.go` da task); (2) a query `ReservarNumero`
  (`proximo_numero - 1 AS numero_reservado`) foi inferida pelo `sqlc` como
  `int32` em vez de `int64` — corrigido com `(proximo_numero - 1)::bigint`
  explícito. `make lint`/`build`/`vet`/`test` (suíte completa, `-race`)
  verdes; `sdd-guard.sh pre-implement`/`pre-complete task_03` → `OK`;
  `compozy tasks validate`/`sync` ok. **Nota de processo**: a primeira
  tentativa de `pre-complete` reprovou por eu ter colocado a nota de desvio
  de implementação sob `## Limites` em vez de `## Evidências produzidas` —
  só essa última seção é normalizada pelo guard contra o Evidence SHA
  auditado; movida a nota pra lá e o gate passou.

**FECHADO — ciclo SDD completo** (2026-09-14). `task_04` (identidade
visual) e `task_05` (calendário) executadas depois de `task_03`, PR #11
(`sdd/f1-02-parametros-identidade-empresa-plan`) aberto e mesclado em
`main` (`b71113f`) — mas o review remoto publicou **2 P1 + 6 P2**, exigindo
um segundo ciclo corretivo pós-merge: branch `sdd/f1-02-correcao-pos-merge`
(reauditoria própria, achados críticos corrigidos — `beee4f5` compensa
criação sem vigência, `88b8021` preserva auditoria de feriados removidos),
PR #12 mesclado (`8a83598`). Consolidação (`d59b920`, contrato de
`empresas` atualizado) e aprendizados (`386964d`) registrados. `main` do
`systeme-erp-backend` avançou até `386964d`. Zero P0/P1 abertos ao fechar.
**Aprendizado promovido**: review remoto (comentários do bot/CI no PR) deve
ser tratado como bloqueador de merge mesmo com checks verdes — vira
checklist operacional para os próximos incrementos. Lead time total do
incremento: 34h; 7 bugs registrados no total (task_01 3, task_03 2,
review remoto 2 críticos do corretivo). Ver [[f1-03-certificado-digital]]
para o incremento seguinte.
