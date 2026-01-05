# Comprehensive Platform Audit Canvas

## Who the Auditor Agent is (execution context)

**Auditor Agent** = an automated, stateless program that uses an LLM only as a **bounded execution engine**.

The Auditor Agent:

* Operates on a repository snapshot.
* Executes only deterministic detectors: **PATH / CFG / AST / TEXT / DIFF**.
* Treats the repository as the **only source of authority**.
* Retains **no memory** between runs. Historical state must live in-repo.
* MAY have limited write access; if so it MUST write only under `audit/**` and MUST NOT modify production code.

This document is written **for the Auditor Agent**. Humans may read it, but it is designed for reliable agent execution.

---

## What this audit is for

This is a deterministic audit framework intended for repos that assume:

* No stable teams
* No long-term ownership
* No reliable institutional memory

Therefore the system must be:

* **Structurally self-explanatory**
* **Mechanically verifiable**
* **Safe to change by unfamiliar engineers**
* **Resilient to long-term entropy**

Architecture, documentation, constraints, and runtime signals must replace tribal knowledge.

---

# Part A — Constitutional contracts (load-bearing)

## A1. Authority contract (normative)

1. **Authority derives only from declared repository artefacts and deterministic proof.**
2. The Auditor Agent MUST NOT introduce new policy.
3. The Auditor Agent MUST NOT infer authority.
4. Anything that cannot be checked mechanically is treated as **risk** (not as fact).

## A2. Non-interference contract (normative)

The following outputs are **non-authoritative**:

* Suggestions
* Action Plan
* Emerging primitives / fragmentation analysis
* Hygiene/advisory checks
* Patch sketches or diffs
* Bootstrap drafts

They MUST:

* NOT create requirements
* NOT affect scoring
* NOT affect deltas
* NOT affect pass/fail
* NOT be treated as proof

## A3. Evidence standard (mandatory)

Every Finding MUST include:

* **Artefact** (path, symbol, config key, dashboard ref)
* **Proof** (quoted value, AST/CFG match, diff, rule violation)
* **Impact** (security, reliability, cost, safety, integrity, operability)
* **Scope** (local or systemic)
* **Confidence** (High | Medium | Low)

Recommendations are advisory only and MUST be traceable to the finding.

## A4. Determinism contract (normative)

### A4.1 Path normalisation

All paths used in findings/artefacts/fingerprints MUST:

* Be repo-relative
* Use `/`
* Be case-sensitive
* Remove any leading `./`
* Resolve `..` safely without escaping the repo root

### A4.2 Proof normalisation

Normalise before comparison:

* TEXT: trim trailing whitespace, convert CRLF→LF, preserve order
* AST/CFG: structural representation only; ignore formatting/comments; ordering ignored unless semantically relevant
* DIFF: compare normalised representations only

A “material proof change” is one that remains after normalisation.

### A4.3 Fingerprint rules

Fingerprint derivation by detector:

* PATH: hash(normalised path)
* CFG: hash(canonical JSON with sorted keys)
* AST: hash(tuple(path, fully-qualified symbol, semantic signature))
* TEXT: hash(normalised excerpt)

If fingerprint cannot be produced:

* `fingerprint = "UNAVAILABLE"`
* `confidence = Low`
* Emit an explicit limitation

### A4.4 content_hash integrity

`content_hash = sha256(normalised findings[])` where findings[] are:

* Sorted by `finding_id`
* Stripped of volatile fields (timestamps, priority_score)
* Serialised with stable key ordering

---

# Part B — Audit mode, tiers, and execution state

## B1. Modes (mutually exclusive)

The audit runs in exactly one mode:

* **Bootstrap (Drafting)** — non-authoritative discovery, no enforcement
* **Enforcement (Normal)** — deterministic enforcement and scoring

### Bootstrap precedence rule (normative)

If bootstrap entry conditions are met (B2), the audit MUST run in Bootstrap mode regardless of any declared tier. In Bootstrap mode:

* Enforcement checks MUST NOT run
* Hard-fail conditions MUST NOT be evaluated
* Scoring and deltas MUST NOT be computed

## B2. Bootstrap entry conditions (mandatory)

Bootstrap MUST activate iff any are true:

* `audit.manifest.yaml` is missing, OR
* `audit.manifest.yaml` fails schema validation (missing required top-level keys), OR
* `audit.manifest.yaml` exists but one or more declared `docs_contract` files are missing

## B3. Statelessness and repo-persisted state (Enforcement mode)

The Auditor Agent retains no memory. The repository stores state.

The Auditor Agent MUST read:

* `audit/latest.json` when present
* `audit/open_findings.yaml` when present (but MUST NOT assume it exists)

---

# Part C — Bootstrap mode (drafting only)

## C1. Default bootstrap scope

If `audit.manifest.yaml` is missing, bootstrap discovery MUST use:

* `include_paths`: ["**/*"]
* `exclude_paths` (minimum):

  * `.git/**`
  * `node_modules/**`
  * `vendor/**`
  * `third_party/**`
  * `dist/**`
  * `build/**`
  * `coverage/**`
  * `.next/**`
  * `target/**`
  * `bin/**`
  * `obj/**`

If `.gitignore` exists, ignored paths SHOULD be treated as excluded.

## C2. Allowed inference (bootstrap only; non-authoritative)

The auditor MAY infer **for drafting only**:

* Components from directory boundaries / build artefacts / Dockerfiles / IaC modules
* Entry points from routes/handlers/cron/queue consumers
* Data stores from ORM configs/migrations/connection usage
* External dependencies from SDK usage/outbound calls
* Trust boundaries from ingress/network exposure
* Capabilities (e.g. `http`, `auth`, `async`, `ui`)

All inferred fields MUST include `{value, source, evidence, confidence}` and MUST set `source: inferred` when not directly evidenced.

## C3. Required bootstrap outputs

The auditor MUST generate:

1. `audit/bootstrap.manifest.draft.yaml`
2. `audit/bootstrap.components.draft/COMPONENT.<id>.yaml` (one per inferred component)
3. `audit/bootstrap.report.json`

If write access exists: write to these paths.

If not: emit full content inline with explicit path labels and stable delimiters.

## C4. Bootstrap gate finding

In bootstrap mode, the auditor MUST emit exactly one gating finding:

* **High** severity `BOOTSTRAP_REQUIRED`

And MUST:

* Mark all bootstrap outputs **PROVISIONAL**
* Refuse to run enforcement checks
* Refuse to compute scoring or deltas
* Set audit pass/fail status to `N/A`

## C5. Promotion rule

Bootstrap ends only when:

* `audit/bootstrap.manifest.draft.yaml` is promoted to `audit.manifest.yaml`, AND
* Each drafted component contract is moved to its declared `docs_contract` path

Renaming/moving constitutes human acceptance.

---

# Part D — Enforcement mode (authoritative within bounds)

## D1. Audit objective (mandatory)

Record:

* Primary risk classes (safety, integrity, availability, cost)
* Decisions this audit informs
* Validity horizon of results
* Audit tier

## D2. Tier declaration (normative)

Tier MUST be declared in `audit.manifest.yaml`:

```yaml
system:
  audit:
    tier: 0 | 1 | 2 | 3
```

If missing:

* Default to Tier 0
* Emit a Medium finding

Only checks at or below the effective tier may produce hard failures.

---

# Part E — Authoritative repository artefacts

## E1. `audit.manifest.yaml` (authoritative)

### Required top-level keys

* `manifest_version` (string)
* `system` (object)
* `scope` (object)
* `components` (array)
* `journeys` (array)
* `policies` (object)
* `checks` (object)

### System intent (authoritative rule)

System intent MUST be declared only in `audit.manifest.yaml`:

* `system.name`
* `system.description`
* `system.primary_user_goals` (required array)

The auditor MUST NOT infer system intent.

If `system.primary_user_goals` missing: High finding.

### Critical journeys (authoritative rule)

Journeys are defined exclusively in `audit.manifest.yaml` under `journeys[]`.

If missing or empty: High finding.

### Non-functional expectations (authoritative rule)

Must be declared in `audit.manifest.yaml` under `system.non_functional`.

If missing: High finding.

## E2. Component documentation contract (authoritative)

Each component MUST have a machine-parseable doc at `docs_contract`.

Missing or non-conformant `COMPONENT.yaml` is High severity.

Missing for any `criticality: critical` component is an automatic audit fail (per manifest fail conditions).

---

# Part F — Output channels

## F1. Findings (authoritative within bounds)

Findings are evidence-backed violations produced by deterministic checks.

Findings MAY affect scoring, deltas, and pass/fail (subject to mode + tier gating).

## F2. Suggestions (non-authoritative)

Suggestions:

* MUST be traceable to one of: `finding_id`, `check_id`, `skipped_check_id`, `drift_event`
* MUST reference evidence already collected
* MUST NOT affect scoring/pass-fail/deltas

## F3. Patch safety

If the auditor emits diffs:

* MUST be marked as proposed
* MUST be minimal and scoped to referenced artefacts
* MUST NOT add dependencies unless allowed by manifest (or suggestion is to add+declare)
* If write access exists, may write only to `audit/patches/*.diff`
* MUST NOT directly modify production code

---

# Part G — Scoring, deltas, and hard fail conditions (Enforcement only)

## G0. Domains (authoritative)

* **Domains are a closed vocabulary defined by the Machine Check Catalogue (Part I).**
* Each check declares exactly one `domain`.
* Domains are used only for aggregation, scoring, and reporting.
* The auditor MUST NOT invent new domains at runtime.

## G1. Maturity scoring (1–5)

Applies only in Enforcement mode.

**Default rule (unless overridden in `audit.manifest.yaml` under `checks.scoring`):**

* Start each domain at 5
* For each **open** finding in the domain subtract:

  * 2.0 for Critical
  * 1.0 for High
  * 0.5 for Medium
  * 0.2 for Low
* Cap minimum at 1

Accepted risks do not reduce score unless expired.

**Applicability rule:** Findings from SKIPPED checks do not exist and cannot affect scoring. If a domain has no applicable checks, report `N/A` and exclude from “domain score 1” fail condition unless overridden.

## G2. Repo audit state

Required:

* `audit/latest.json` (auditor-authored)

Recommended:

* `audit/open_findings.yaml` (human-authored)

Optional:

* `audit/history/YYYY-MM.json`
* `audit/suppressions.yaml`

Rules:

* If `audit/latest.json` missing: treat run as baseline; emit High `AUDIT_STATE_MISSING`.
* If `audit/open_findings.yaml` missing: infer lifecycle state from `audit/latest.json`; emit Medium `OPEN_FINDINGS_MISSING`.

## G3. Delta semantics (no memory)

Compare current run to `audit/latest.json`.

Classify findings as:

* New
* Still open
* Worsened
* Resolved

Worsened iff:

* Severity increases, OR
* `evidence.artefacts` set grows, OR
* `primary_artefact` unchanged AND `fingerprint` unchanged AND proof changes materially, OR
* Finding affects additional components

Drop in confidence alone does not count unless severity also increases.

## G4. Hard fail conditions (Enforcement only)

Audit fails if:

* Any applicable Critical finding exists from checks at/below effective tier
* Any applicable domain score is 1 (domains scored N/A excluded)
* Docs contracts missing for any `criticality: critical` component

---

# Part H — Deterministic applicability (noise control)

## H1. Capabilities vocabulary (recommended)

* `http`, `http_client`, `ui`, `async`, `auth`, `authz`, `multi_tenant`, `external_calls`, `data_store`, `observability`, `genai`, `admin_surface`

## H2. Applicability rule (normative)

If a check declares required capabilities and none are present across in-scope components, the check outcome MUST be **SKIPPED** (not PASS), MUST be listed, and MUST NOT affect scoring.

Capability inference is allowed only to:

* Emit `MANIFEST_CAPABILITY_MISMATCH`, or
* Provide hints

Inference MUST NOT silently apply or skip checks.

---

# Part I — Machine Check Catalogue (deterministic ABI)

## I0. Catalogue ABI guarantee (normative)

* The auditor MUST implement **at least** the checks listed in I3.

* Additional checks MAY be added, but MUST NOT weaken, bypass, replace, or redefine the minimum checks.

* `check_id` values are stable across versions.

* Removing a check or changing detector semantics is a breaking change.

* Default severities may be overridden only by manifest `checks.severity_mapping`.

## I1. Detector types

* PATH: file/directory existence, glob match
* TEXT: deterministic regex/text match
* AST: language-aware parse and rule match
* CFG: config parse (YAML/JSON/TOML/etc.)
* DIFF: comparison to prior audit state

## I2. Outcomes

* PASS: executed, no violation
* FAIL: violation detected → produces a finding
* SKIPPED: not applicable → listed, produces no finding

## I3. Core checks (minimum set)

Bootstrap gate rule: if `audit.manifest.yaml` is missing, the auditor MUST enter Bootstrap mode and MUST emit only `BOOTSTRAP_REQUIRED` as the gating finding. In this case `MANIFEST_PRESENT` is reported as **SKIPPED (bootstrap)** and MUST NOT produce severity.

| Check ID                      | Domain               | What is checked                                       | Detector  | Evidence (proof)                   | Default severity        | Confidence | Delta key                  |
| ----------------------------- | -------------------- | ----------------------------------------------------- | --------- | ---------------------------------- | ----------------------- | ---------- | -------------------------- |
| MANIFEST_PRESENT              | Audit mode           | `audit.manifest.yaml` exists at repo root             | PATH      | Presence/absence                   | n/a (bootstrap gate)    | High       | repo:manifest              |
| MANIFEST_SCHEMA               | Audit mode           | Manifest contains required top-level keys             | CFG       | Parsed keys + missing list         | High                    | High       | repo:manifest_schema       |
| MANIFEST_CAPABILITIES_MISSING | Audit mode           | Any component missing `capabilities`                  | CFG       | Component ids missing capabilities | Medium                  | High       | repo:manifest_caps         |
| MANIFEST_CAPABILITY_MISMATCH  | Audit mode           | Inferred capability present but missing from manifest | AST/TEXT  | Evidence + component id            | Medium                  | Medium     | repo:cap_mismatch          |
| AUDIT_STATE_MISSING           | Audit mode           | `audit/latest.json` missing                           | PATH      | Missing path                       | High                    | High       | repo:audit_state           |
| AUDIT_STATE_CORRUPT           | Audit mode           | `audit/latest.json` hash mismatch                     | CFG/DIFF  | Previous vs recomputed hash        | High                    | High       | repo:audit_state_integrity |
| AUDIT_STATE_WRITE_FAILED      | Audit mode           | Auditor cannot write `audit/latest.json`              | TEXT      | Explicit write failure             | High                    | High       | repo:audit_state_write     |
| COMPONENTS_ENUMERABLE         | System map           | Components declared with required fields              | CFG       | Missing fields list                | High                    | High       | comp:{id}:schema           |
| DOCS_CONTRACT_PRESENT         | Cold-start           | `docs_contract` exists per component                  | PATH      | Missing paths list                 | High (critical => fail) | High       | comp:{id}:docs_path        |
| DOCS_CONTRACT_SCHEMA          | Cold-start           | Contract conforms to schema                           | CFG       | Missing keys list                  | High                    | High       | comp:{id}:docs_schema      |
| DOCS_CONTRACT_CROSSREF        | Cold-start           | Contract references only declared stores/deps         | CFG       | Invalid refs list                  | High                    | High       | comp:{id}:docs_refs        |
| ENTRYPOINTS_MATCH             | Entry points         | Manifest entry points match code locations            | PATH/TEXT | Unmatched patterns list            | High                    | Medium     | comp:{id}:entrypoints      |
| FORBIDDEN_IMPORTS             | Boundaries           | Disallowed imports outside allowed paths              | AST       | Offending files + symbols          | High                    | High       | rule:{rule_id}:violations  |
| LAYER_VIOLATIONS              | Boundaries           | Dependency graph forbidden edges                      | AST/CFG   | Edge list (from→to)                | High                    | High       | graph:forbidden_edges      |
| MULTI_RESP_MODULE             | Boundaries           | DB+network in forbidden locations                     | AST       | Call/import evidence               | High                    | Medium     | comp:{id}:multi_resp       |
| SECRETS_IN_REPO               | Security             | Hard-coded secrets                                    | TEXT/AST  | Snippet hash + file                | Critical                | High       | sec:secrets                |
| AUTHN_ENFORCEMENT             | Security             | Auth guard on public boundaries                       | AST/TEXT  | Wiring evidence                    | High                    | Medium     | comp:{id}:authn            |
| AUTHZ_CENTRALISED             | Security             | Authz via central policy                              | AST       | Call evidence                      | High                    | Medium     | comp:{id}:authz            |
| TENANT_ISOLATION              | Security             | Tenant enforced and propagated                        | AST/TEXT  | Enforcement points                 | High                    | Medium     | sec:tenant                 |
| OUTBOUND_TIMEOUTS             | Reliability          | Outbound calls have timeouts                          | AST/CFG   | Wrapper/args evidence              | High                    | Medium     | rel:timeouts               |
| BOUNDED_RETRIES               | Reliability          | Retries bounded & idempotency-aware                   | AST/CFG   | Policy/config evidence             | High                    | Medium     | rel:retries                |
| RATE_LIMITING                 | Reliability/Security | Rate limiting on exposed entry points                 | AST/CFG   | Middleware/config evidence         | High                    | Medium     | rel:ratelimit              |
| IDEMPOTENCY_ASYNC             | Data/Reliability     | Async consumers idempotent/deduped                    | AST/CFG   | Strategy evidence                  | High                    | Medium     | data:idempotency           |
| LOG_SCHEMA_FIELDS             | Observability        | Logs include required fields                          | AST/CFG   | Logger wrapper evidence            | High                    | Medium     | obs:log_schema             |
| PII_REDACTION                 | Obs/Security         | Redaction/scrubbing present                           | CFG/TEXT  | Redaction config evidence          | High                    | Medium     | obs:redaction              |
| TRACE_PROPAGATION             | Observability        | Trace IDs propagated and logged                       | AST/CFG   | Middleware evidence                | High                    | Medium     | obs:tracing                |
| RUNBOOK_PRESENT               | Operability          | Runbooks exist w/ recovery steps                      | PATH/TEXT | Missing files/heading evidence     | High                    | Medium     | ops:runbooks               |
| GENAI_GATEWAY_ONLY            | GenAI                | Provider SDK only inside gateway                      | AST       | Offending imports                  | Critical                | High       | ai:gateway                 |
| GENAI_MODEL_ALLOWLIST         | GenAI                | Models restricted to allow-list                       | AST/CFG   | Offending model strings            | Critical                | High       | ai:models                  |
| GENAI_COST_CAPS               | GenAI                | Token/cost caps exist                                 | CFG/TEXT  | Cap values                         | High                    | Medium     | ai:caps                    |
| FRONTEND_CSP                  | Front-end            | CSP exists and is non-trivial                         | CFG/TEXT  | Header/meta evidence               | High                    | Medium     | fe:csp                     |
| BUNDLE_SIZE_DRIFT             | Front-end            | Bundle size drift beyond threshold                    | DIFF      | Size delta + threshold             | Medium                  | High       | fe:bundle                  |
| DEPENDENCY_DRIFT              | System map           | New external deps since last run                      | DIFF      | New deps list                      | Medium (escalate)       | High       | dep:drift                  |
| HIGH_FINDINGS_DRIFT           | Delta                | New High/Critical findings since last run             | DIFF      | Delta list                         | High                    | High       | delta:high                 |

---

# Part J — Final output contract

## J1. Executive summary

* System intent (from manifest)
* Mode (Bootstrap/Enforcement)
* Pass/fail (`N/A` in bootstrap)
* Top 10 Now (ranked)
* New and unresolved risks
* Drift highlights
* Skipped checks summary (what and why)

## J2. Findings

For each finding:

* Artefact
* Proof
* Impact
* Recommendation (advisory)
* Confidence

## J3. Remediation guidance

* Required fixes
* Optional improvements
* Verification signals

## J4. Explicit limitations

* What could not be verified statically
* Any assumptions (bootstrap inference)

## J5. Suggestions (non-authoritative)

Each suggestion MUST include:

* `suggestion_id` (stable)
* `title`
* `trigger` (finding_id | check_id | skipped_check_id | drift_event)
* `evidence_refs`
* `change_type` (code|config|docs|infra|observability|tests)
* `risk_class` (security|reliability|cost|safety|integrity|operability)
* `expected_effect`
* `confidence`

Optional:

* `effort_hint` (S|M|L)
* `patch_sketch` (non-authoritative)
* `verifies_finding_ids`
* `expected_verification_change`

## J6. Action Plan (derived; non-authoritative)

Construction (default):

1. Take all `summary.top_findings`.
2. Attach any Suggestions whose trigger references those findings.
3. Rank using:

   * finding priority (implementation-defined unless overridden in `audit.manifest.yaml` under `checks.prioritisation`)
   * suggestion prioritisation weights (implementation-defined unless overridden in `audit.manifest.yaml` under `checks.prioritisation`)
   * component criticality
4. Emit top N (default 10)

Each action MUST include:

* `action_id`
* `source` (finding_id | suggestion_id | combined)
* `recommended_change`
* `evidence_refs`
* `verification`

---

# Part K — Advisory analyses (non-authoritative)

This part defines optional analyses that may produce **Suggestions** only. They MUST obey Part A2 (non-interference).

## K1. Emerging primitives / fragmentation (advisory)

Purpose: surface early convergence/fragmentation in bounded pattern categories **without** implying correctness or policy.

Rules:

* MUST NOT create findings, severities, scores, deltas, or fail conditions.
* MUST rely only on PATH/AST/CFG (TEXT allowed only with reduced confidence).
* MUST NOT perform whole-program inference or cross-category similarity.

Applicability (mandatory):

* If mode is Bootstrap → MUST be **SKIPPED**.
* If `MANIFEST_SCHEMA` is FAIL → MUST be **SKIPPED**.
* If `COMPONENTS_ENUMERABLE` is FAIL → MUST be **SKIPPED**.

Eligible categories (bounded):

* Logging init/usage
* Outbound HTTP/RPC clients
* Input validation frameworks
* Error construction/propagation
* Auth/authz middleware
* Database access patterns
* GenAI invocation patterns (only if `genai` capability is declared)

Output (if any): category, pattern identifiers, counts, components affected, trend (if baseline exists), evidence refs, confidence.

## K2. Hygiene catalogue (advisory)

Advisory checks MAY exist to generate Suggestions (e.g. missing lockfiles, missing test commands, duplicated config sprawl).

Rules:

* MUST have stable `check_id`s.
* MUST be listed in output.
* MUST NOT create findings or affect scoring/deltas/pass-fail.

---

# Closing criterion

**Can a competent engineer, unfamiliar with this system, safely understand, modify, and operate it — as verified mechanically by this audit?**

If not, the system is **not survivable** under zero-team conditions.

**Can a competent engineer, unfamiliar with this system, safely understand, modify, and operate it — as verified mechanically by this audit?**

If not, the system is **not survivable** under zero-team conditions.
