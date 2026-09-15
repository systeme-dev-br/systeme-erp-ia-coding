---
name: f1-01-frontend-cadastro-empresas-filiais
description: "Estado do incremento SDD F1-01 frontend (cadastro de empresas e filiais) do systeme-erp — código em systeme-erp-frontend, fluxo SDD rodado a partir de systeme-erp-backend"
metadata:
  node_type: memory
  type: project
  originSessionId: 80cb4e67-356f-4613-abe5-a522474b4926
  modified: 2026-09-13T10:54:41.905Z
---

Incremento SDD **f1-01-frontend-cadastro-empresas-filiais** — a parte de
frontend da issue F1-01 do plano de fases (`docs/historias/plano-fases-issues.md`
linha 112, labels backend+frontend). Decidido na triagem do F1-01 backend
(2026-09-10) como incremento **separado, logo após o backend**: o backend fecha
primeiro (API + contrato vivo `empresas` consolidado — ver [[f1-01-cadastro-empresas-filiais]]),
o frontend consome por referência de SHA. Iniciado em 2026-09-11 a pedido do dono
("pode começar").

## Natureza cross-repo (PRM-001)

O fluxo SDD (guard, prompts, `incremento.yaml`, `.compozy/tasks`) roda inteiramente
em `systeme-erp-backend` — o `sdd-guard.sh` só audita esse worktree (mesma
restrição de `systeme-erp-docs` no F1-01 backend, PRM-004 de lá). O **código**
(React/TS) é escrito e commitado em `systeme-erp-frontend`. Cada task referencia
o SHA do commit em `systeme-erp-frontend`, não um diff local no backend.

## Decisões da triagem (2026-09-11)

- **Auth provisória (AskUserQuestion, dono escolheu "Tela mínima de bootstrap"):**
  sem login real ainda (F1-07 depende de F1-06, ambos depois do F1-01). O frontend
  ganha uma tela mínima que chama `POST /usuarios` (cria o usuário titular), guarda
  o id retornado em `localStorage` e usa como header `X-Usuario-ID` em toda chamada
  — mesmo caráter provisório do ADR-002 do F1-01 backend (bloqueia produção até
  F1-07). Dá um fluxo demonstrável ponta a ponta (criar usuário → criar empresa →
  ativar).
- **Fora de escopo desta entrega (PRM-003..007):**
  - Botão "buscar dados por CNPJ" (EMP-CNPJ-014): backend só tem a porta
    `port.ConsultaCNPJ` sem provedor concreto — formulário 100% manual.
  - Limite de empresas do plano (EMP-CAD-001 RN-01): F11 (billing), backend não
    enforce; sem UI de limite/upgrade.
  - RBAC fino ("Administrar empresas"): F1-08; qualquer usuário criado pode
    administrar empresas nesta entrega.
  - Troca de contexto / consolidado (EMP-CTX-006/007/008): F1-04; listagem de
    empresas é só navegação simples.
  - Compartilhamento de cadastros entre matriz/filial (EMP-CAD-002 RN-03): não
    existe módulo de cadastro operacional ainda; nota de UX para o futuro.

## Classificação

`rigor: medium` (várias telas sobre contrato de API já consolidado, sem
ambiguidade arquitetural grande) · `risco: medio` (sem migração/dado regulado, mas
toca criação de conta e introduz a tela de bootstrap de identidade provisória) ·
`alvo: branch` (sem ambiente de deploy do frontend ainda) · `autonomia:
autonomo_ate_pr` (mesmo padrão do F1-01 backend). Rota: 00→01→02→03→04→06→07→08→
10→11→13→14 (TechSpec+review+PR/merge obrigatórios; deploy dispensado).

## Escopo funcional (a partir da API já consolidada)

- Bootstrap: criar usuário titular mínimo (tela provisória, ver acima).
- Criar empresa (formulário completo: razão social, CNPJ, natureza jurídica, IE/
  isento, CNAE principal+secundários, endereço com IBGE).
- Listar empresas da conta; ver detalhe + checklist de ativação
  (`GET /empresas/{id}` com `checklist_ativacao`).
- Definir regime tributário (`PATCH /empresas/{id}`).
- Ativar empresa quando o checklist fechar (`POST /empresas/{id}/ativacao`);
  mostrar pendências quando incompleto.
- Criar/listar filial vinculada à matriz (mesmo radical de CNPJ; herança de
  regime via `COALESCE`, mostrar o que é herdado vs. próprio — nota de UX da
  história EMP-CAD-002).
- Tratamento de erro: mapear os slugs documentados no contrato `empresas`
  (`cnpj_invalido`, `cnae_invalido`, `campo_obrigatorio`, `ibge_invalido`,
  `ie_indefinida`, `regime_invalido`, `radical_cnpj_divergente`) para mensagem
  específica por campo, não só erro genérico.

## Progresso

- **Branch**: `sdd/f1-01-frontend-cadastro-empresas-filiais-plan` (de `main`
  `f7fdc7d` do `systeme-erp-backend`), empurrada.
- **Passo 00 (triagem) FEITO** (`ee93f96`): `incremento.yaml`
  (`rigor: medium`/`risco: medio`/`autonomia: autonomo_ate_pr`/`alvo: branch`,
  PRM-001..007, `base_sha: f7fdc7d`), `brief.md`,
  `impacto-contratual/empresas/contrato.md` (ALTERADO).
- **Passo 01 (PRD) FEITO** (`4fe060f`): RF-001..008, BR-001..003,
  RNF-001..003. Trata os dois casos especiais de erro do backend
  (`cnpj_ja_cadastrado` 409, `checklist_incompleto` 422), não só os slugs de
  campo simples.
- **Passo 02 (TechSpec) FEITO** (`488db2c`): arquitetura RTK Query
  (`injectEndpoints` por feature), `identidadeSlice`+`localStorage`,
  normalização de erro `{erro,mensagem,detalhes}` → mensagem por campo,
  tipos espelhando `dto.go`. **ADR-001** (bootstrap via `localStorage` +
  slice Redux, sem lib de persistência) e **ADR-002** (validação de CNPJ no
  cliente é espelho, nunca bloqueio definitivo).
- **Passo 03 (plano+tasks) FEITO** (`2636236` + fix `6a322af`): `execucao.md`
  (RSK-001 drift `msw`↔API real sem OpenAPI, RSK-002 bootstrap confundido
  com login), `INDEX.md`, `feature/001` com **10 SCN/TST**, **6 tasks**
  (task_01 infra, task_02 bootstrap, task_03 cadastrar empresa+erro, task_04
  detalhe+checklist+regime, task_05 filiais, task_06 consolidação+verificação
  manual contra backend real). `compozy tasks validate` → all valid (6).
  **2 achados corrigidos no próprio passo 03** (não esperar auditoria):
  `complexity: small` não é enum válido do schema v2 (usar `low`) — o
  exemplo do prompt 03 está errado, registrar no passo 14; `dependencies:
  ["task_NN"]` (com aspas) passa no `compozy tasks validate` mas reprova o
  `sdd-guard.sh pre-implement` (mesmo gotcha do ciclo 05-fiscal, forma
  canônica é `[task_NN]` sem aspas).
- **Passo 04 (auditoria) FEITO** (`c2ff6b9 aprova especificacao`): achados
  AUD-001/AUD-002 corrigidos em `de2c133`; gate humano aprovado pelo dono
  (2026-09-11).
- **`task_01` FEITA e reauditada** (`d84356b47`/`090d0bc`): infra RTK
  Query/store/identidade/rota protegida.
- **`task_02` FEITA e reauditada** (`20d703e7`/`b0e5a6a1`): bootstrap
  provisório de identidade.
- **`task_03` FEITA e reauditada** (2026-09-12): cadastrar empresa e
  validação de erro. Código frontend commit `8c65c6e88cb4def17f54d521a377d86e5fe3a547`
  (branch `sdd/f1-01-frontend-cadastro-empresas-filiais`) — reescreveu por
  completo um rascunho não commitado que estava em qualidade abaixo do
  padrão do repo (linhas únicas densas, sem Tailwind, formulário incompleto:
  faltavam isento de IE, CNAEs secundários, regime tributário no formulário
  de criação). Cobertura final: 27 testes na task (37 na suíte completa do
  frontend), lint/typecheck/build verdes. **Desvio técnico aceito (AUD-006,
  P3):** a TechSpec previa `msw`; a implementação seguiu o padrão real já
  estabelecido pela task_02 (`vi.mock` do hook RTK Query, sem dependência
  nova) — documentado em `task_03.md` e na auditoria, `_techspec.md` não foi
  editado para não reabrir revisão de plano por um detalhe de técnica de
  teste. Registro de execução `39beca2eddc9fc1e60b4c751fe5ab084b4a8b2c6`,
  reaudit `2416e1ee894af5a29195ac9c33c4777f2688858c` (AUD-005 task_03,
  AUD-006 TechSpec desatualizada) — ambos no backend, branch
  `sdd/f1-01-frontend-cadastro-empresas-filiais-plan`. `reauditorias: 3`.
- **`task_04` FEITA e reauditada** (2026-09-12): detalhe da empresa,
  checklist de ativação, definir regime tributário. Código frontend commit
  `dac4a4e61a505655e6d0689dac3c66b4d5628a15` (branch
  `sdd/f1-01-frontend-cadastro-empresas-filiais`): `empresasApi.obterEmpresa`/
  `definirRegime`/`ativarEmpresa` (tag `Empresa`), `ChecklistAtivacao`
  (pendentes vs. botão "Ativar"; `422 checklist_incompleto` atualiza a lista
  a partir de `detalhes.pendentes` sem novo GET), `EmpresaDetalhePage`
  (regime via `PATCH`, refetch automático por `invalidatesTags`), rota
  `/empresas/:id`. Regimes válidos confirmados no backend (`empresa.go`):
  `simples_nacional`/`lucro_presumido`/`lucro_real`. Cobertura: 6 testes
  novos citando SCN-006/SCN-007 (43 na suíte completa), lint/typecheck/build
  verdes. Mesmo desvio técnico do `msw` (AUD-006) documentado de novo em
  `task_04.md`. Registro de execução `b655c474043adbcae1854e53f550ef13fcb83c4a`,
  reaudit `d54ed9911aaf1bf2f165df8ff1d6079b7c40fd81` (AUD-007) — backend, branch
  `sdd/f1-01-frontend-cadastro-empresas-filiais-plan`. `reauditorias: 4`.
- **`task_05` FEITA e reauditada** (2026-09-12): cadastro e listagem de
  filiais. Código frontend commit `887b2c329a5e5f5727b191ba80862fd67a2bc49f`
  (branch `sdd/f1-01-frontend-cadastro-empresas-filiais`):
  `empresasApi.criarFilial`/`listarFiliais` (tag `Filial`),
  `FormularioFilial` (CNPJ, IE/isento, inscrição municipal, endereço, regime
  opcional com opção "Herdar da matriz"), `FilialNovaPage` (mapeia `422
  radical_cnpj_divergente` para o campo CNPJ da filial), lista de filiais em
  `EmpresaDetalhePage` (CNPJ, status, `regime_efetivo`) + link para
  `/empresas/:id/filiais/nova`. Campos de `criarFilialReq`/`FilialResp` em
  `types/api.ts` já existiam prontos desde task_01 (só reusados). Cobertura:
  6 testes novos citando SCN-008/SCN-009/SCN-010 (46 na suíte completa),
  lint/typecheck/build verdes. Mesmo desvio técnico do `msw` (AUD-006)
  documentado de novo em `task_05.md`. Registro de execução
  `4d2873449ca24aa7d5176e77429f11cf5919ed0b`, reaudit
  `fb3c516a5752a8b5345cbf980cf94f188c3fc5b2` (AUD-008) — backend, branch
  `sdd/f1-01-frontend-cadastro-empresas-filiais-plan`. `reauditorias: 5`.
- **`task_06` FEITA e reauditada (2026-09-12) — última task de implementação.**
  Código frontend commit `be1c0d5` (branch
  `sdd/f1-01-frontend-cadastro-empresas-filiais`): só `README.md` (seção
  F1-01 com estado real + instrução de como subir a stack completa;
  correção da referência obsoleta a `docs/openapi.yml`, fechando AUD-002).
  Suíte completa: 46 testes, 10/10 SCN (`SCN-001..010`) citados em pelo
  menos um teste; lint/typecheck/build verdes.
  **Verificação manual contra o backend real**: subiu a stack de verdade
  (Postgres/Redis via `systeme-erp-infra`, `make migrate-up`, `make run` em
  `http://localhost:8080`, frontend via `npm run dev` em `:5173`). A
  verificação via navegador (`chrome-devtools`) **não funcionou neste
  ambiente** (`Target closed`, sem Chrome/display utilizável no sandbox) —
  substituída por verificação via `curl` direto contra a API real,
  reproduzindo os 10 SCN ponta a ponta (bootstrap → empresa → regime →
  checklist 422 → ativação → filial com herança → radical divergente 422 →
  CNPJ duplicado 409 → listagens) e comparando cada resposta com
  `src/types/api.ts`. **Nenhuma divergência de contrato encontrada — RSK-001
  não se materializou.** Única observação sem impacto: `endereco.complemento`
  é `omitempty` no backend (ausente quando vazio) mas tipado como `string`
  obrigatória no frontend; nenhum componente lê esse campo de uma resposta
  da API, então sem efeito — registrado em `task_06.md`, sem correção
  necessária. Registro de execução `b78436f66e36aa2bd09aa0ca6790d5d8ca8cf536`,
  reaudit `875abc7` (AUD-009) — backend, branch
  `sdd/f1-01-frontend-cadastro-empresas-filiais-plan`. `reauditorias: 6`.
  `incremento.yaml` avançou `fase: implementacao` → `fase: review`
  (`status` continua `em_execucao`).
  **Todas as 6 tasks de implementação estão concluídas.**
- **Review (passo 07) — 2 rodadas (2026-09-12/13).** Rodada 1 `REPROVADO`
  (commit `4a99289`, backend): REVIEW-001 (P1, aviso de CNPJ duplicado nunca
  exibido — `EmpresaDetalhePage` não lia `location.state`), REVIEW-002 (P2,
  CNPJ ausente na listagem de empresas), REVIEW-003 (P2, `/bootstrap`
  reenviável mesmo com usuário já guardado). Corrigidos no passo 09 (bugfix):
  frontend commit `58c1366` (7 arquivos, +4 testes de regressão, suíte
  46→50), backend `f4321bb` (fecha as 3 issues + `bugfix-report.md`). Rodada
  2 `APROVADO` (commit `1cafc97`, Evidence SHA `58c1366`) — revalidou lendo o
  código real (não aceitou o relato do bugfix), reexecutou
  lint/typecheck/test/build; 2 P3 aceitos sem correção (promise não tratada
  em `BootstrapPage`; `EmpresaDetalhePage` preso em "carregando" p/ id
  inexistente).
- **QA (passo 08) — 1 rodada, `APROVADO`** (commit `fed56dd`, Evidence SHA
  `58c1366`). 10/10 SCN passaram, nenhum `NAO_VERIFICADO`: subiu a stack real
  de novo (Postgres/Redis/API/frontend), validou via `curl` real (incluindo
  calcular um CNPJ com radical divergente pra isolar essa regra) + suíte de
  componente lida linha a linha. Nenhum bug novo. Serviços derrubados ao
  final. `incremento.yaml` → `status: validado`, `fase: validacao`.
- **PR (passo 10) — abertas 2026-09-13, 2 PRs pareadas (incremento
  cross-repo, PRM-001), aguardando merge do dono (nunca mesclar sozinho):**
  - **Código**: `systeme-erp-frontend#1` — https://github.com/systeme-dev-br/systeme-erp-frontend/pull/1
    (branch `sdd/f1-01-frontend-cadastro-empresas-filiais` → `main`, base
    `79ed731`, HEAD `58c1366`).
  - **Rastro SDD**: `systeme-erp-backend#8` — https://github.com/systeme-dev-br/systeme-erp-backend/pull/8
    (branch `sdd/f1-01-frontend-cadastro-empresas-filiais-plan` → `main`,
    base `f7fdc7d`). Traz `pr/pr-package.md`+`pr-body.md` (commit `118f743`).
  As duas PRs estão cruzadas (corpo de cada uma linka a outra).
  `incremento.yaml`: `fase: pr`, `gates.pr.gate_humano: aprovado`
  (messiasneto74, "sim, pode mandar"), `pr.numero/url` = PR do backend,
  `pr.frontend_numero/frontend_url` = PR do frontend (campo novo, não
  padrão dos incrementos anteriores — só este por ser cross-repo).
  **Próximo pendente: merge das duas PRs (decisão do dono) → passo 11
  (validar/merge) → passo 13 (consolidação do contrato vivo `empresas`) →
  passo 14 (aprendizados).**

## Nota de processo (2026-09-12)

Os forks despachados para task_03/task_04 foram instruídos a nunca
despachar sub-agentes por conta própria; o fork da task_03 violou isso uma
vez (despachou um segundo agente de reaudição não pedido, sem efeito
colateral). A partir da task_05 a implementação passou a ser feita
diretamente pela sessão principal (sem fork), inclusive o papel de
reaudição (sem invocar `cz-auditor-especificacao` como ferramenta — o
parecer é escrito diretamente no mesmo formato AUD-00N). A task_06 seguiu o
mesmo padrão (execução direta, sem sub-agente) e ainda documentou uma
segunda substituição de ferramenta aceita: verificação manual planejada via
navegador trocada por `curl` direto na API, por indisponibilidade de
Chrome/display no ambiente de execução — mesmo objetivo (mitigar RSK-001),
outro meio.

**Domínio de contrato vivo:** `empresas` (ALTERADO, não NOVO — é o mesmo domínio
de negócio do backend, só ganhando comportamento observável de UI).

## FECHAMENTO — ciclo SDD 14/14 (2026-09-13)

- **Merge das 2 PRs do passo 10** (2026-09-13, dono): `systeme-erp-frontend#1`
  → merge `6636b073603d0b5f746e955a6ce72a390a9233fe`; `systeme-erp-backend#8`
  → merge `c3913ec000a965bb11b12148bff2a3518f4dc4a2`. Sem drift em `main`.
- **Passo 11 (validar merge) + passo 13 (consolidar contrato vivo)**, feitos
  juntos numa PR: `systeme-erp-backend#9`, commit `4d05e97`, mesclada pelo
  dono (`559e8f2`). Bug real encontrado e corrigido no processo: o Evidence
  SHA de review/QA da rodada 2 apontava direto pro commit do
  `systeme-erp-frontend` (`58c1366`), inválido no worktree do backend onde o
  `sdd-guard.sh` roda — corrigido para o commit do backend (`f4321bb`, mesmo
  padrão das auditorias AUD-001..009). `pre-merge`/`pre-consolidate` OK após
  a correção. `sdd/contratos/empresas/contrato.md` ganhou a seção de
  comportamentos de UI; `status: consolidado`, incremento arquivado em
  `sdd/historico/2026-09-11-f1-01-frontend-cadastro-empresas-filiais/`.
- **Passo 14 (aprendizados)**: `systeme-erp-backend#10`, commit `10057bf`,
  aberta 2026-09-13, **aguardando merge do dono** (esperado ficar vermelha no
  CI, `sdd/metricas.csv` fora das exclusões — mesmo padrão dos outros ciclos).
  `sdd/aprendizados/2026-09-13-f1-01-frontend-cadastro-empresas-filiais.md`.
  Promovido: nota em `sdd/prompts/07-revisar-implementacao.md` e
  `08-executar-qa.md` (Evidence SHA cross-repo tem de ser commit deste
  worktree, nunca de outro repositório); nota em `_comum.md` (faixa agregada
  tipo `TST-001..TST-010` na rastreabilidade vira id composto único pro guard
  — 3ª ocorrência da família, não promovido a mudança de regex); nota em
  `02-criar-techspec.md` (checar convenção real de mock do repo antes de
  propor `msw`); nota em `08-executar-qa.md` (API real + suíte de componente
  é evidência válida quando não há navegador no sandbox). Não promovido: o
  sub-agente despachado sem autorização na task_03 (ocorrência única, sem
  recorrência).
- **Métricas finais** (`sdd/metricas.csv`): lead total 40h, `reauditorias: 6`,
  `revisoes_plano: 0`, `issues_review: 3` (1 P1 + 2 P2, todos corrigidos),
  `bugs: 0`, `p0/p1_abertos: 0`, `fora_do_plano=16/16`.
- **INCREMENTO 100% FECHADO.** Zero branches `sdd/` remanescentes esperadas
  após o merge da PR #10 (frontend e backend). Próxima frente: continuar o
  plano de fases F1 (ver `f1-01-cadastro-empresas-filiais` para o que vem
  depois de F1-01).
