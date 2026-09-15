---
name: sdd-guard-authority-gap
description: "Saga do check \"SDD guard\" vermelho no systeme-erp (AG-001) e como está sendo resolvido"
metadata:
  node_type: memory
  type: project
---

O job `SDD guard` falhava em todo PR do systeme-erp porque o template SDD entrega `authority.check_command` vazio e o `sdd-guard.sh` falha fechado em todo gate de autoridade (audit/review/qa/quality/merge/consolidate). Doc: `docs/governanca/AG-001-authority-checker-ausente.md`.

**Tentativa 1 — opção B (#162, MESCLADA, foi erro):** `continue-on-error: true` na etapa "Pre-merge" do `sdd-guard.yml`. Quebrou **EVAL-026** (tier1, roda da base, não é advisory, tem `! grep continue-on-error: true`). Depois do #162 no `main`, todo PR passou a falhar nas evals.

**Correção — PR #163 (`chore/sdd-authority-checker`, aberto, aguardando merge manual):**
- `sdd/governanca/authority-check.sh` — checker POSIX sh determinístico. Concede gates do fluxo de incremento (audit/review/qa/merge/quality_*/contract_change/commit/push/pull_request/merge); nega governance_change/protected_path/staging/production.
- `policies.yaml` — `check_command: /usr/local/bin/sdd-authority-check` + `check_sha256` (253c0b58…09178).
- `sdd-guard.yml` — (1) etapa que instala o checker no CI da base confiável com verificação de SHA; (2) **reverte** o continue-on-error do #162; (3) "Pre-merge" só roda `pre-merge-ci` em incremento `status: validado`; (4) `docs/governanca/*` nas exclusões.
- Local: `sudo install -D -m 0755 sdd/governanca/authority-check.sh /usr/local/bin/sdd-authority-check` — **já feito** nesta máquina.
- Tier1 completo passa local: 91/0/0.

**#163 fica vermelho no CI (esperado, 2x):** o PR (evals leem base contaminada pelo #162 + edita governança) e o push-de-merge dele (`event.before` ainda tem continue-on-error). Merge manual. **Depois disso todo PR fica verde** e `pre-consolidate` / mudança de `sdd/contratos/` passam.

**How to apply:** se #163 já está no `main`, `guard` verde é o esperado (exceto PR de governança, vermelho 1x por design). Se não, o vermelho continua esperado. Próximo passo pós-merge: consolidação do 00 (PR separado, branch off main limpa). Ver [[serie-modelo-dados-f0-06]].
