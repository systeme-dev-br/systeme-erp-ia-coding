---
name: f1-03-certificado-digital
description: "Estado do incremento SDD F1-03 (certificado digital A1) do systeme-erp-backend — cofre cifrado, e-CNPJ/e-CPF, validade, alertas e bloqueio de emissão fiscal"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1b4d8295-0b6b-4788-a445-a2b83382be68
  modified: 2026-09-15T02:32:44.708Z
---

Incremento SDD **f1-03-certificado-digital** — issue F1-03 do plano de
fases (`docs/historias/plano-fases-issues.md`, labels `backend`,
`security`, `mvp`, depende de F1-01), história `EMP-CERT-004`
(`docs/historias/gestao-empresas-multiempresa.md`). Depende só do contrato
vivo `empresas`. Segue direto após o fechamento de
[[f1-02-parametros-identidade-empresa]] (2026-09-14).

## Escopo

Cofre de certificado A1 por empresa: upload de PFX/P12+senha, validação
(senha/formato/cadeia/validade/vínculo de CNPJ), cifra em repouso,
histórico de substituições, remoção com confirmação e auditoria, alertas
de expiração por marco, e uma porta de elegibilidade que emissores fiscais
futuros vão consultar (sem emissor real nesta entrega). Por decisão do
dono em 2026-09-14, também aceita **e-CPF de responsável fiscal**
designado explicitamente pelo titular da empresa — CPF nunca persistido em
claro, comparado via HMAC de chave de runtime, designação auditada (não é
consulta externa de representação legal).

Restrições inegociáveis: nunca logar/persistir/retornar chave privada, PFX
ou senha em claro; cifra falha fechada se a chave de cifragem não estiver
disponível; sem A3, certificado em nuvem, emissor de NF-e real ou e-mail
real nesta entrega.

## Classificação (passo 00)

`rigor: large` / `risco: regulado` (segredo criptográfico + CPF
pseudonimizado + isolamento multiempresa) / `autonomia: autonomo_ate_pr` /
`alvo: branch` (sem ambiente de deploy do backend confirmado). Rota
completa: TechSpec, review e PR humanos obrigatórios; deploy dispensado.

## Progresso

- **Base**: `main` do `systeme-erp-backend` em `386964d` (pós-fechamento
  F1-02).
- **Especificação FEITA com reauditoria 1**: `auditoria-especificacao.md`
  com Evidence SHA `cb08b45` — **PRONTO**. `revisoes_plano: 1`,
  `reauditorias: 1`.
- **2 ADRs**: `adr-001-cifra-envelope-e-validacao-pkcs12.md` (cifra por
  envelope + validação PKCS#12), `adr-002-responsavel-fiscal-e-cpf-
  pseudonimizado.md` (HMAC de CPF, sem persistir em claro).
- **Gate humano de especificação APROVADO** (dono, 2026-09-14): escopo
  cobre certificado A1 e-CNPJ/e-CPF, responsável fiscal designado com CPF
  pseudonimizado, testes, commits, push e abertura de PR; merge continua
  sujeito a gate próprio.
- **Plano**: 5 tasks (`task_01` schema/portas/persistência tenant,
  `task_02` criptografia/PKCS#12/trust store, `task_03` domínio/casos de
  uso/HTTP, `task_04` job/alertas, `task_05` regressão/segurança/runbook),
  14 SCN/TST, 4 FEAT, contrato `empresas` (ALTERADO).
- **`task_01` EXECUTADA E FECHADA** (`845c5c2`, "cria cofre tenant de
  certificado"): migration tenant `000006` (versões/alertas/auditoria/
  responsáveis fiscais HMAC, sem coluna de senha/PFX/CPF em claro), sqlc,
  entidade/porta/mapper/repo PostgreSQL via `database.EmTenant`. SCN-011/
  SCN-013 automatizados.
- **`task_02` EXECUTADA E FECHADA** (`6ca2c9d`, "valida PKCS#12 e cifra
  certificado por envelope"): pacote novo `internal/shared/certificado/`
  (`validador.go`, `cifra.go`, `documento.go`, `erros.go`) — `Validador`
  decodifica PKCS#12 (`software.sslmate.com/src/go-pkcs12`, ADR-001),
  confere senha/correspondência chave-certificado/validade/cadeia contra
  trust store PEM/documento do titular via SAN ICP-Brasil (e-CNPJ/e-CPF)
  com fallback por `SerialNumber`/assinatura sintética em memória;
  `Cifrador` implementa envelope AES-256-GCM (nonce aleatório por chamada,
  AAD empresa+versão+key_id); `Pseudonimizador` gera HMAC-SHA256 do CPF do
  responsável fiscal (ADR-002) sem persistir em claro. `config.go` ganhou
  4 campos de certificado **sem default** — falha fechada
  (`ErrConfiguracaoCriptografica`/`ErrTrustStore`) se ausentes/inválidos.
  7 testes novos (`SCN-001/002/003/004/012/014`) com fixtures 100%
  sintéticas geradas em runtime (PRM-001) — nenhum PFX/senha real em
  código ou log. `go build`/`go vet`/`make lint`/`go test ./... -race`
  (suíte completa) verdes, sem regressão. `sdd-guard.sh pre-implement` e
  `pre-complete task_02` → OK; `scan-secrets` → OK; `compozy tasks
  validate` → all valid (5).
- **Branch ainda 100% local**: `sdd/f1-03-certificado-digital` não tem
  upstream no GitHub (`git ls-remote` não mostra a branch) — diferente do
  padrão de F1-01/F1-02 de empurrar cedo. Push ficará para quando o
  incremento chegar ao passo de abrir PR, a menos que o dono peça antes.

- **`task_03` EXECUTADA E FECHADA** (`f01dca3`, "casos de uso e API segura
  do ciclo de certificado"): a task_01 tinha deixado só schema/portas —
  faltava o `CertificadoRepo` Postgres de verdade (`Criar`/`Utilizavel`/
  `Historico`/`Remover`/`RegistrarAlerta`/`ProximaVersao`) e a auditoria
  segura (`empresa_certificado_auditoria`, query nova
  `RegistrarAuditoriaCertificado`); completados nesta task, junto com
  `usecase/certificado.go` (upload→valida→autoriza titular e-CNPJ/e-CPF→
  cifra com a próxima versão→persiste; consulta/histórico/remoção
  confirmada; elegibilidade derivada de `validade_fim` em tempo de leitura,
  BR-010) e os handlers/rotas HTTP. `port.MembershipRepo` ganhou
  `EhTitular` (BR-003). **Desvio técnico registrado**: designação de
  responsável fiscal virou `POST .../responsaveis-fiscais` (não o `PUT
  .../responsaveis-fiscais/{id}` da TechSpec) porque o id é gerado pelo
  servidor. 7 testes de usecase (fakes) + 6 de integração Postgres real
  (isolamento entre tenants, idempotência de alerta) + 2 E2E HTTP com A1
  sintético de verdade via multipart real (fixture extraída para
  `internal/testsupport/certfixture`, compartilhada com a task_02).
  `go build`/`vet`/`make lint`/`go test ./... -race` (suíte completa)
  verdes; `scan-secrets` OK; `sdd-guard.sh pre-implement`/`pre-complete
  task_03` → OK; `compozy tasks validate` → all valid (5). Branch ainda só
  local, sem push.

- **`task_04` EXECUTADA E FECHADA** (`57e5c74`, "alertas idempotentes e
  comando diário de expiração"): nova porta `port.NotificadorCertificado`
  (no-op instrumentado em `internal/adapter/notificacao/`, PRM-002 — só
  loga, a garantia é o evento persistido, sem provedor externo real).
  `usecase.VerificarExpiracoesCertificado` varre uma lista de tenants já
  resolvida (o `cmd` descobre os tenants, não o usecase) comparando datas
  civis UTC truncadas contra os marcos 30/15/7/1 dia (relógio injetável,
  BR-010), com timeout de 30s por tenant e falha isolada agregada sem
  interromper os demais. `cmd/certificado-alertas/main.go` é uma varredura
  única (sem ticker interno — cron é da infraestrutura). 5 testes de
  usecase novos (SCN-009/010 + fora do marco + sem certificado + falha
  isolada). Smoke test manual contra Postgres real de dev: `go run
  ./cmd/migrate up` (3 tenants pré-existentes migrados pra 000006) +
  `go run ./cmd/certificado-alertas` → 3 processados, 0 alertas (correto,
  nenhum tem certificado real). Suíte completa/lint/scan-secrets/gates
  verdes.

- **`task_05` EXECUTADA E FECHADA** (`749e5a9`, "regressão transversal,
  segurança e runbook"): conferida a matriz SCN-001..014 (todos com prova
  automática entre task_02/03/04), `make sqlc`/`test`/`lint`/`build` e
  scan de segredo verdes. Novo `docs/runbook-certificado-digital.md`
  (variáveis sem default, rotação de chave, execução do job, resposta a
  expiração, dev local). `README.md` ganhou a tabela de rotas F1-03 +
  `cmd/certificado-alertas` + aviso de pré-requisito. `.env.example` ganhou
  os 4 placeholders vazios. **Achado de governança registrado, não
  corrigido** (fora de escopo, escopo reduzido honesto): `.env.example` já
  tinha `DB_PASSWORD=postgres` antes da F1-03 — bate a política
  `secrets.forbidden_patterns` (só aceita placeholders tipo
  `changeme/example/dummy/test`), mas é a senha fixa do
  `docker-compose.yml` do `systeme-erp-infra` (repo separado); não
  alterado para não quebrar `make compose-up`.

**AS 5 TASKS DE IMPLEMENTAÇÃO DO F1-03 ESTÃO FECHADAS** (`749e5a9`).

- **Review de implementação** (agente `cz-revisor-implementacao`, contexto
  isolado): **APROVADO**, Evidence SHA `749e5a9`. 3 achados P3 (sem P0/P1):
  doc desatualizada quanto ao verbo HTTP de responsável fiscal, ausência de
  slug específico para A3, lacuna estrutural (não funcional) de cobertura.
  Rodou `go build`/`vet`/`make lint`/`go test ./... -race` ele mesmo.
- **QA** (agente `cz-qa`, contexto isolado): **APROVADO**, mesmo Evidence
  SHA, 0 bugs. Validou como consumidor real: suíte completa contra
  Postgres/Redis reais, API real via `curl` com A1 sintético ponta a
  ponta, `cmd/certificado-alertas` contra banco de dev, inspeção direta via
  `psql` das tabelas de auditoria/responsáveis fiscais confirmando ausência
  de CPF/segredo em claro. 14 SCN cobertos, nenhum `NAO_VERIFICADO`.
- `sdd-fluxo.sh --run` aplicou a transição `em_execucao → validado`
  (`fase: pr`). Pacote de PR preparado (`pr/pr-package.md`+`pr-body.md`,
  commit `1a20bdd`).
- **PR #13 aberto** (`systeme-erp-backend`, branch
  `sdd/f1-03-certificado-digital` → `main`, head SHA `ad863de`):
  https://github.com/systeme-dev-br/systeme-erp-backend/pull/13.
  `gates.pr.gate_humano` registrado como aprovado (autonomia
  `autonomo_ate_pr`, escopo já coberto pelo gate de especificação + "sim"
  do dono nesta sessão). `pr.status: aberto`. CI (`build`/`guard`/`lint`/
  `test`) disparou, ainda rodando na abertura.

**CI do PR #13 concluído**: `build`/`lint`/`test` verdes; `guard` vermelho
— **EVAL-066 esperado** (mesmo gap de governança já documentado em
[[sdd-guard-authority-gap]], `ci.yml` da base sem actions fixadas por SHA),
não uma falha introduzida por este incremento.

**PR #13 MESCLADO PELO DONO** (2026-09-14, merge SHA `98c1a2c`, `main` do
`systeme-erp-backend`). Gate humano de merge registrado como aprovado no
`incremento.yaml` (`status: validado`/`fase: merge`); `sdd-guard.sh
pre-merge` → OK. `merge-report.md` escrito (Head SHA validado = Evidence
SHA de review/QA `749e5a9`, Merge SHA `98c1a2c`) e empurrado para a branch
`sdd/f1-03-certificado-digital` (`2d34d8d`) — branch já mesclada, sem
efeito em `main`.

**Padrão observado divergente do F1-01**: o `git log` de `main` mostra que
a consolidação de contrato vivo e os aprendizados do F1-02
(`d59b920`/`386964d`) foram commitados **direto em `main`, sem PR** — linha
reta no `git log --graph`, não merge commit. Diferente do padrão de PRs
separados de consolidação/aprendizados usado nas séries F0-06/F1-01.

**FECHADO — ciclo SDD 14/14** (2026-09-14/15). Passos 13 (consolidar
contrato vivo) e 14 (aprendizados) feitos numa branch dedicada
`sdd/f1-03-certificado-digital-fechamento`, criada a partir do **Merge SHA**
`98c1a2c` (não da ponta de `main`, que já tinha avançado com o commit
`AGENTS.md`/repo de memória — gotcha do prompt 13 confirmado na prática;
precisou de `cherry-pick` do `merge-report.md` que eu tinha empurrado por
engano na branch já mesclada). `sdd/contratos/empresas/contrato.md` ganhou
5 comportamentos novos (certificado A1, responsável fiscal e-CPF,
renovação/remoção, elegibilidade, alertas). Incremento e tasks movidos
para `sdd/historico/2026-09-14-f1-03-certificado-digital/`.

**2 achados de harness promovidos** (`sdd/aprendizados/2026-09-14-f1-03-
certificado-digital.md`): (1) `sdd-metricas.sh` `expected_plan_paths`
tratava célula com vários caminhos separados por vírgula como 1 path
malformado — zerava a métrica de aderência ao plano mesmo sem desvio real
(mesmo formato de tabela já usado no F1-02, provavelmente com o mesmo bug
lá); corrigido para separar por caminho. (2) prompt 03 não instruía
caminho completo na tabela "Arquivos e superfícies esperadas" — nomes
curtos de pacote (`entity`, `usecase`) nunca batem por comparação de
prefixo; nota preventiva adicionada ao prompt.

**PR #14 MESCLADO PELO DONO** (2026-09-15, merge SHA `8ad424d`, `main` do
`systeme-erp-backend`). **INCREMENTO 100% FECHADO.** Faxina feita:
`sdd/f1-03-certificado-digital` e `sdd/f1-03-certificado-digital-
fechamento` apagadas (local+remoto); zero PRs abertas no repo. `main` em
`8ad424d`.
