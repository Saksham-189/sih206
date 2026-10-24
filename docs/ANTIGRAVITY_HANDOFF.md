# Antigravity takeover brief — Pramaan FDR Portal

## Paste this prompt into Antigravity

You are taking over **Pramaan — Trust Backed by Evidence**, an AICTE Security Deposit/FDR portal for SIH 2026 problem statement 26265. Work carefully and incrementally. The repository is `https://github.com/Saksham-189/sih206`.

You are continuing Saksham's work. The active review branch is `s/s1-s6-release-integration`; its current head is `3fa2ee5` (`feat: add advisory AI release summaries`) and it is the head branch for PR #18. Do not merge the PR or push to `main` unless the human explicitly asks. Begin by checking `git status`, fetching remote branches, and reading `AGENTS.md` plus this document.

Do not edit `.env`, reveal or commit secrets, credentials, uploaded files, `data/`, or a virtual environment. Do not rewrite applied migrations. Do not implement Kushagra's platform/DevOps tasks or Yash's records/document/AI-provider tasks. Preserve normal role scope, CSRF/session authentication, optimistic locking (`version`), audit events, human decisions, and synthetic-data labels.

The immediate objective is to review and stabilize the existing implementation, not to invent a larger product. First establish a reproducible local baseline, report actual results, and then fix only verified defects. Treat the root README's statement that “No S7 work is added” as stale: S7 is now implemented on the active PR. This document and `docs/owned/S7_AI_RELEASE_SUMMARY.md` describe it accurately.

When changing code, explain the finding and intended fix first. Run focused tests after each meaningful change, then relevant regression tests. Do not claim an external service, browser check, deployment, or full regression passed unless you actually ran it. Keep the project functional with AI disabled.

## What the product does

The portal is a **synthetic demo MVP**, not a production AICTE or bank system. It supports an institution depositing FDR/RTGS/RPGF security evidence; officers reviewing records and obligations; maturity/reporting; release cases with clearance/penalty/authority controls; and protected/public letter verification. The React frontend uses a Django REST API. The legacy Django interface remains under `/legacy/`.

Core safety rules:

- Acceptance is tracking/review, never bank verification or legal compliance.
- Every consequential release action is a recorded human action with a reason and current version.
- Uploaded documents and AI output cannot independently set verification, penalties, release status, approval, or completion.
- Use synthetic files only for external AI until an approved data-governance decision exists.
- Scope every read before looking up IDs; do not trust an institution ID supplied by a client.

## Current branches and collaboration

| Item | State |
|---|---|
| Repository | `https://github.com/Saksham-189/sih206` |
| Saksham review branch | `s/s1-s6-release-integration` |
| Latest S7 commit | `3fa2ee5` |
| Review PR | `https://github.com/Saksham-189/sih206/pull/18` |
| Main branch policy | Do not push or merge directly; merge only after review and CI. |
| Ownership | Saksham: obligations, institutions, release casework, reports/KPIs/import, release UI, AI release summary. Yash: records/documents/AI shared module. Kushagra: platform, auth, shell, roles/audit/notifications/DevOps. |

`AGENTS.md` is the team contract. The private planning PDFs are in ignored `planning/` only on machines where they were copied. If they are unavailable, use the committed handoff and implementation documents rather than guessing policy.

## Authoritative code map

| Area | Start here |
|---|---|
| Django settings/root routing | `config/settings.py`, `config/urls.py`, `portal/api/urls.py` |
| Roles and scope | `portal/permissions.py` |
| Core records/documents | `portal/models.py`, `portal/services.py`, `portal/uploads.py` |
| Obligations/rules/API | `obligations/models.py`, `obligations/services.py`, `obligations/api/` |
| Release case workflow | `casework/services.py`, `casework/facts.py`, `casework/api.py`, `casework/api_urls.py` |
| AI provider shared by Yash | `ai/models.py`, `ai/services.py`, `ai/provider_worker.py` — consume, do not rewrite without Yash/team agreement |
| Saksham React screens | `frontend/src/features/saksham/` |
| Shared React API/auth/router | `frontend/src/api/`, `frontend/src/App.tsx`, `frontend/src/features/saksham/routes.tsx` |
| Tests | `obligations/tests/`, `portal/tests/`, `ai/tests/`, `frontend/src/features/saksham/*.test.tsx` |
| Saksham implementation record | `docs/owned/README.md`, `docs/owned/MERGED_CHECKS_2026-10-25.md`, `docs/owned/S1_S6_IMPLEMENTATION_PLAN.md` |

## Completed Saksham scope

### S1 — institution/obligation foundation

`InstitutionProfile`, `ProgrammeApproval`, `RuleSet`, and `DepositObligation` are in `obligations/`. They retain rule/version provenance and do not invent data for old records.

### S2 — rules, calculations, findings and APIs

Draft demo rule payloads, deterministic required amount/coverage calculations, evidence-linked findings, calculator and institution/obligation APIs are implemented. Regulatory meaning remains draft/demo until the team validates it against current AICTE policy.

### S3 — institution screens

Institution list/detail and requirement calculator components are in `frontend/src/features/saksham/`, using the shared React shell and client.

### S4 — reports, KPI and import preparation/integration

CSV import validation/preview/commit, reports and KPI calculations/components exist. Some reports retain explicit source-adapter limits where Yash-owned verification/document facts are unavailable. Do not turn an unavailable source into a zero/clear result.

### S5 — release-case backend

`ReleaseCase`, clearances, penalty assessment, decision gates, unsigned synthetic NOC/sanction PDFs with QR lookup, revocation, and evidence-backed completion are implemented in `casework/` plus obligations models/migrations. The design fails closed when trusted release-facts, canonical questionnaire, completion parsing/evidence, or permission integrations are unavailable.

### S6 — release UI

The release wizard, release case, clearance queue and public verification UI are mounted through `frontend/src/features/saksham/routes.tsx`. The wizard intentionally refuses to invent a canonical Affidavit-6 questionnaire or source document rules.

### S7 — advisory AI release summary (latest work)

Implemented in commit `3fa2ee5`:

- Endpoint: `POST /api/releases/{id}/ai-summary` with `{ "version": <saved case version> }`.
- Users: an assigned clearance-unit user or authority, after normal case scoping.
- Source sent to the provider: structured case JSON only — path, boolean declarations, document kinds/count, clearance statuses, penalty facts, letter state and last 25 audit actions. No PDF bytes, document text, filenames, evidence references, person names, raw reasons, uploads, or tools.
- Output: a short `summary` and at most five `{field, severity, message}` risks. UI label: **AI-generated: verify before use**.
- Cache: success is keyed by a SHA-256 hash of the structured case payload, which includes case version. Calls, disabled requests and failures create `AiRun` rows with operation `RELEASE_SUMMARY`.
- Safety: it cannot call a release workflow; it does not mutate status, clearances, penalties, decisions, letters, monitoring, or completion.
- Availability: when `AI_ENABLED=false`, missing key, provider failure/timeout, or malformed model JSON occurs, it returns an unavailable/manual-review result and the release screen remains usable.
- Frontend: the button is only shown to eligible roles and is hidden when `AI_ENABLED=false`.

Read `docs/owned/S7_AI_RELEASE_SUMMARY.md` before changing this feature. S7 uses Yash's existing `ai.services.call_model()` transport but does not modify `ai/`.

## Local setup — Windows PowerShell

Use a fresh clone or a clean branch. Do not copy an existing `.venv`, database, browser profile, or `.env` from another teammate.

```powershell
git clone https://github.com/Saksham-189/sih206.git pramaan
Set-Location .\pramaan
git fetch origin --prune
git switch s/s1-s6-release-integration
git pull --ff-only

py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt

Set-Location .\frontend
npm ci
Set-Location ..
```

Node.js **22.12+** and Python **3.12** are expected. For a normal synthetic demo, no provider API key or cloud database is required.

Create `.env` from `.env.example` only when required by the current scripts. Keep it ignored. For a completely local demo, retain the safe defaults:

```env
DJANGO_DEBUG=true
AI_ENABLED=false
```

Do not put an API key, database password, audit secret, or any real applicant data in a prompt, source file, git commit, screenshot, or documentation.

## Run the application

Preferred Windows command:

```powershell
.\run.ps1
```

Or run the backend manually after migrations/seeding:

```powershell
.\.venv\Scripts\python.exe manage.py migrate
.\.venv\Scripts\python.exe manage.py seed_review
.\.venv\Scripts\python.exe manage.py runserver 127.0.0.1:8010 --noreload
```

Open `http://127.0.0.1:8010/`. API documentation is at `/api/docs/`; legacy screens are at `/legacy/`.

The review/demo seed commands create synthetic accounts and private local credentials. Read those files only locally; they are git-ignored. Do not publish account passwords.

## Tests and checks

Start with these exact commands:

```powershell
# Backend static/migration checks. Use local SQLite if a private .env contains an unreachable remote DATABASE_URL.
$env:DATABASE_BACKEND='sqlite'
.\.venv\Scripts\python.exe manage.py check
.\.venv\Scripts\python.exe manage.py makemigrations --check --dry-run

# S7-focused backend checks
.\.venv\Scripts\python.exe manage.py test obligations.tests.test_ai_release_summary --verbosity 1

# Frontend checks
Set-Location .\frontend
npm run typecheck
npm run test -- --run src/features/saksham/components.test.tsx
npm run build
Set-Location ..
```

Recorded actual result at commit `3fa2ee5`:

| Check | Result |
|---|---|
| S7 route/contract test | 4 passed |
| Saksham React component suite | 12 passed |
| Frontend TypeScript | passed |
| Frontend production build | passed |
| Python compile/diff check | passed |

### Known test/environment issue

The wider `obligations.tests.test_casework` / `ai.tests.test_extraction` execution was blocked **before S7 tests ran** in `PortalTestCase.draft()`: it expects HTTP `302` from the legacy `instrument_create` view but received `200`. It also encountered a Windows sandbox temporary-media cleanup `PermissionError`. Do not attribute this to S7 without reproducing on a normal local clone. Investigate the legacy route/test setup separately, make a minimal verified fix only if it is reproducible, then run the full regression suite.

The migration dry run may warn if `.env` points at a Supabase host that cannot be resolved from the machine. That is an environment/network condition, not confirmation that a migration passes. Use a local SQLite test setting for the structural check.

## Enabling real AI later

This is optional and must be approved for **synthetic data only**. The implementation does not provide a free/local model automatically.

1. Obtain an approved Anthropic API key through the team/owner.
2. In the private deployment or local `.env`, set:

   ```env
   AI_ENABLED=true
   AI_MODEL=claude-opus-5-5
   ANTHROPIC_API_KEY=<private value>
   ```

3. Restart Django.
4. Sign in as a clearance user assigned to the case or an authority user, open a release case and select **Generate AI summary**.
5. Verify the output is labelled advisory, field-linked, stored in `AiRun`, cache-reused for the same case version, and does not change workflow state.

Do not claim a model call worked until a real approved synthetic-data test succeeds. `AI_ENABLED=false` is the intended default and must remain a fully usable manual workflow.

## First tasks for Antigravity

1. Verify PR #18 against current `origin/main`; resolve merge conflicts only after reviewing both sides and never discard teammate changes.
2. Reproduce the known backend test failure on a clean Windows/local SQLite environment. Identify whether it is a stale test expectation, routing order, or legacy template behavior. Keep the patch separate from S7 if unrelated.
3. Run the full Django and frontend suites after that reproduction/fix. Report exact pass/fail counts and commands.
4. Manually test a synthetic release flow in the browser: institution request → assigned clearance decision → officer penalty review → independent authority decision → letter → completion. Test an unauthorized direct URL too.
5. Test S7 in both modes: AI disabled (no button/manual path), and provider-enabled only if an approved synthetic-data key is supplied. Confirm cached same-version output and stale-version conflict handling.
6. Reconcile stale docs, especially README S7 statements, only after verifying the merged behavior. Avoid deleting historical handoff material; label it dated instead.
7. Before any deployment, finish the release-readiness manual checks: Chromium/Firefox target-browser behavior, protected PDF viewing/download, offline demo rehearsal, secrets review, migrations on a fresh database, and recovery drill.

## Definition of done for any further change

- The relevant user flow works end-to-end using synthetic data.
- Server-side authorization and scope are tested, including direct endpoint access.
- A stale `version` write returns `409` without partial state changes.
- Consequential decisions remain human-recorded, auditable and evidence-backed.
- AI is advisory and the app works with AI disabled or unavailable.
- Focused tests and the applicable regression suite have real recorded results.
- No secrets, private uploads, databases, build artifacts, credentials, or `.env` files are committed.
- Documentation states what is real, simulated, unavailable, and still pending.

## References

- `AGENTS.md` — ownership, contracts and security rules.
- `docs/api-conventions.md` — API/error/pagination conventions.
- `frontend/README.md` — React setup and test guidance.
- `docs/review-build.md` — synthetic seed/review workflow.
- `docs/release-readiness.md` — prior release audit and manual checklist.
- `docs/handover/README.md` — earlier handoff index.
- `docs/owned/S7_AI_RELEASE_SUMMARY.md` — exact S7 behavior.
