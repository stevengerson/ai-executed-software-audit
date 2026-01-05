# Comprehensive Platform Audit

## Part A0 — Canonical structure and rule ownership (normative)

This document follows a **single-source-of-truth rule** to prevent semantic duplication.

Canonical ownership:

* **Execution semantics & global guarantees** → Part A
* **Check outcome semantics (PASS / FAIL / SKIPPED)** → §A6
* **Applicability rules (capability-based)** → Part H
* **Mode selection (entry conditions)** → Part B
* **Bootstrap gating semantics** → §C4
* **Enforcement gating semantics (tiers, hard-fail)** → Part D / Part G
* **Check definitions & catalogue** → Part I

### A0.1 Canonical reference rule (mandatory)

Any section that needs to rely on a rule defined elsewhere MUST reference the **canonical section ID** and MUST NOT restate the rule text.

To preserve readability, sections MAY include **non-normative summaries**, provided they are explicitly labeled as such and do not introduce new requirements, conditions, or exceptions.

Restating normative behavior outside its canonical section is forbidden.

**Example:**

* ❌ “Bootstrap mode MUST emit exactly one `BOOTSTRAP_REQUIRED` finding.”
* ✅ “Bootstrap behavior is defined canonically in §C4.”

## Document intent (normative)

This document is designed for **deterministic machine execution**.

* Normative language uses **MUST / MUST NOT / SHOULD / MAY**.
* Underspecification handling (including when outcomes MUST be **SKIPPED**) is defined canonically in §A6.

---

## Who the Auditor Agent is (execution context)

**Auditor Agent** = an automated, stateless program that uses an LLM only as a **bounded execution engine**.

The Auditor Agent:

* Operates on a repository snapshot.
* Executes only deterministic detectors: **PATH / CFG / AST / TEXT / DIFF**.
* Treats the repository as the **only source of authority**.
* Retains **no memory** between runs. Historical state must live in-repo.
* MAY have limited write access; if so it MUST write only under `audit/**` and MUST NOT modify production code.

This document is written **for the Auditor Agent**. Humans may read it, but it is designed for reliable agent execution.

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

### A1.0 Named invariants (normative)

The following invariants apply globally and MUST NOT be violated:

* **Invariant A1 — No inferred authority**: Authority derives only from declared repository artefacts and deterministic proof.
* **Invariant D1 — Determinism**: Identical inputs MUST produce identical outputs.
* **Invariant S1 — Stateless execution**: No memory may persist outside the repository.

Subsequent sections MUST reference these invariants by name rather than restating them.

## A1. Authority contract (normative)

1. **Authority derives only from declared repository artefacts and deterministic proof.**
2. The Auditor Agent MUST NOT introduce new policy.
3. The Auditor Agent MUST NOT infer authority.
4. Anything that cannot be checked mechanically is treated as **risk** (not as fact).

## A2. Non-interference contract (normative)

The following outputs are **non-authoritative**:

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

* **Artefact** (path, symbol, config key, dashboard ref)
* **Proof** (quoted value, AST/CFG match, diff, rule violation)
* **Impact** (security, reliability, cost, safety, integrity, operability)
* **Scope** (local or systemic)
* **Confidence** (High | Medium | Low)

Recommendations are advisory only and MUST be traceable to the finding.

## A4. Artefact authority boundary (mandatory)

### A4.1 Normative policy inputs

The following repository artefacts define policy and constraints and are **normative**:

* `audit.manifest.yaml`
* each component `docs_contract`
* allowlists/deny-lists under `policies.*`
* accepted-risk registries (if present)

### A4.2 Authoritative execution inputs (non-normative)

These artefacts are authoritative **inputs** to execution but MUST NOT be treated as policy:

* Repository source code and configuration
* `audit/latest.json`
* `audit/open_findings.yaml`
* Repo-stored metrics under `audit/**`
* Schemas under `audit/schemas/**`

### A4.3 Auditor outputs (non-normative)

All auditor-generated outputs (e.g. `audit/**`) are non-normative and advisory unless explicitly promoted by an external process.

The Auditor Agent MUST NOT modify normative policy inputs.

## A5. Determinism contract (normative)

### A5.1 Path normalisation

All paths used in findings/artefacts/fingerprints MUST:

* Be repo-relative
* Use `/`
* Be case-sensitive
* Remove any leading `./`
* Resolve `..` safely without escaping the repo root

### A5.2 Proof normalisation

Normalise before comparison:

* TEXT: trim trailing whitespace, convert CRLF→LF, preserve order
* AST/CFG: structural representation only; ignore formatting/comments; ordering ignored unless semantically relevant
* DIFF: compare normalised representations only

A “material proof change” is one that remains after normalisation.

### A5.2.1 TEXT proof excerpt rule (mandatory)

When a TEXT detector emits proof excerpts, excerpt selection MUST be deterministic:

* Excerpt MUST be anchored to a deterministic match boundary (regex span, fixed line range, or fixed marker boundaries).
* Excerpt MUST include the anchor plus a fixed context window (default: 2 lines before, 2 after) unless overridden by manifest `checks.proof_excerpt`.
* If multiple matches exist, order by (path, start_offset) and include the first `K` (default: 3).

### A5.3 Fingerprint rules

Fingerprint derivation by detector:

* PATH: hash(normalised path)
* CFG: hash(canonical JSON with sorted keys)
* AST: hash(tuple(path, fully-qualified symbol, semantic signature))
* TEXT: hash(normalised excerpt)

### A5.3.1 AST semantic signature minimum (mandatory)

For AST fingerprints, the semantic signature MUST be computed deterministically and MUST include at minimum:

* symbol kind (function/class/method/etc.)
* fully-qualified name
* arity (when applicable)

If the auditor cannot compute a semantic signature for a language, it MUST set:

* `semantic_signature = "UNAVAILABLE"` and `fingerprint = "UNAVAILABLE"` and `confidence = Low`

If fingerprint cannot be produced:

* `fingerprint = "UNAVAILABLE"`
* `confidence = Low`
* Emit an explicit limitation

### A5.4 content_hash integrity

`content_hash = sha256(normalised findings[])` where findings[] are:

* Sorted by `finding_id`
* Stripped of volatile fields (timestamps, priority_score)
* Serialised with stable key ordering

### A5.5 Check execution determinism (normative)

* A check MUST NOT produce a Finding unless it has:

  * (a) an applicable scope (Part H), AND
  * (b) the required authoritative anchors/inputs (Part E / Part I), AND
  * (c) deterministic proof (PATH/CFG/AST/TEXT/DIFF).

Outcome semantics for missing inputs/anchors (including when the outcome is **SKIPPED**) are defined canonically in §A6.

## A6. Check outcome semantics (normative; canonical)

This section is the **canonical** definition of PASS/FAIL/SKIPPED behavior.
**Non-normative note:** Other sections should reference §A6 rather than repeating outcome rules.

### A6.1 Outcomes

* **PASS**: the check executed and detected no violation.
* **FAIL**: the check executed, detected a violation, and therefore produces a Finding.
* **SKIPPED**: the check did not execute due to inapplicability or missing required inputs. SKIPPED produces no Findings.

### A6.2 Preconditions to execute

A check is eligible to execute only when all of the following hold:

1. **Mode allows execution** (Parts B–D)
2. **Tier allows execution** (Part D)
3. **Applicability holds** (Part H)
4. **Required authoritative inputs exist** (Part E)
5. **Required anchors exist** where the check declares anchor requirements (Part I)
6. **Proof-shapes are available** for the repo language/framework when required (Part I6)
7. **Repo-stored baseline metrics exist** for DIFF checks when required (Part I8)

If any precondition is not satisfied, the outcome MUST be **SKIPPED** with a deterministic `skip_reason`.

### A6.3 Canonical skip reasons

The Auditor Agent MUST use a deterministic skip reason from the following closed set:

* `bootstrap`
* `tier_excluded`
* `not_applicable`
* `missing_authoritative_input`
* `anchor_not_declared`
* `proof_shapes_unavailable`
* `metric_unavailable`

### A6.3.1 Skip-reason precedence (mandatory)

If multiple preconditions fail, the Auditor Agent MUST select the skip reason corresponding to the **earliest failing precondition** in the ordered list defined in A6.2.

### A6.4 SKIPPED listing requirement

All SKIPPED checks MUST be listed in the report with:

* `check_id`
* `skip_reason`
* evidence refs (paths/keys) when the reason implies missing artefacts/inputs

---

# Part B — Audit mode, tiers, and execution state

## B1. Modes (mutually exclusive)

The audit runs in exactly one mode:

* **Bootstrap (Drafting)** — non-authoritative discovery, no enforcement
* **Enforcement (Normal)** — deterministic enforcement and scoring

### Bootstrap precedence rule (normative)

Bootstrap mode selection and its effects are defined canonically in §C4.

**Non-normative summary:** When bootstrap conditions hold, enforcement checks do not run and scoring/deltas are not computed.

## B2. Bootstrap entry conditions (mandatory)

Bootstrap MUST activate iff any are true:

* `audit.manifest.yaml` is missing, OR
* `audit.manifest.yaml` fails validation (see E1: required keys; optional JSON Schema), OR
* `audit.manifest.yaml` exists but one or more declared `docs_contract` files are missing

## B3. Statelessness and repo-persisted state (Enforcement mode)

The Auditor Agent retains no memory. The repository stores state.

The Auditor Agent MUST read:

* `audit/latest.json` when present
* `audit/open_findings.yaml` when present (but MUST NOT assume it exists)

---

# Part C — Bootstrap mode (drafting only)

## C1. Default bootstrap scope

If `audit.manifest.yaml` is missing, bootstrap discovery MUST use:

* `include_paths`: ["**/*"]
* `exclude_paths` (minimum):

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

If `.gitignore` exists, ignored paths SHOULD be treated as excluded.

## C2. Allowed inference (bootstrap only; non-authoritative)

The auditor MAY infer **for drafting only**:

* Components from directory boundaries / build artefacts / Dockerfiles / IaC modules
* Entry points from routes/handlers/cron/queue consumers
* Data stores from ORM configs/migrations/connection usage
* External dependencies from SDK usage/outbound calls
* Trust boundaries from ingress/network exposure
* Capabilities (e.g. `http`, `auth`, `async`, `ui`)

All drafted fields MUST include `{value, source, evidence, confidence}`.

* Use `source: evidence` when directly supported by repo artefacts.
* Use `source: inferred` when not directly supported.
* Fields without direct evidence MUST set `source: inferred`

## C3. Required bootstrap outputs

The auditor MUST generate:

1. `audit/bootstrap.manifest.draft.yaml`
2. `audit/bootstrap.components.draft/COMPONENT.<id>.yaml` (one per inferred component)
3. `audit/bootstrap.report.json`

If write access exists: write to these paths.

If not: emit full content inline with explicit path labels and stable delimiters.

## C4. Bootstrap gate finding (canonical)

This section is the **complete and canonical definition** of bootstrap gating behavior.

### C4.1 Bootstrap mode effects (normative)

When the audit is running in **Bootstrap mode**:

* The Auditor Agent **MUST NOT run enforcement checks**.
* The Auditor Agent **MUST NOT evaluate hard-fail conditions**.
* The Auditor Agent **MUST NOT compute scoring**.
* The Auditor Agent **MUST NOT compute deltas**.
* The audit **pass/fail status MUST be `N/A`**.

These effects apply regardless of declared tier or configured checks.

### C4.2 Bootstrap gating finding (normative)

In Bootstrap mode, the Auditor Agent:

* **MUST emit exactly one Finding**.
* That Finding **MUST** have `finding_id = BOOTSTRAP_REQUIRED`.
* Severity **MUST** be **High**.
* The Finding **MUST NOT** be treated as an enforcement failure.
* **No other Findings MAY be emitted** while in Bootstrap mode.

### C4.3 Bootstrap outputs and authority (normative)

All outputs produced during Bootstrap mode:

* **MUST be marked `PROVISIONAL`**.
* **MUST be non-authoritative** and **MUST NOT be treated as policy**.
* **MUST NOT be promoted** to authoritative artefacts by the Auditor Agent.

The Auditor Agent **MUST refuse promotion** of bootstrap outputs. Promotion may occur only via an external (human or AI) process, as defined elsewhere.

Required bootstrap outputs and their file locations are defined in §C3; this section defines their authority and handling.

### C4.4 Bootstrap blockers section (mandatory; non-finding)

Bootstrap reports **MUST include** a dedicated **“bootstrap blockers”** section.

This section:

* **MUST list** the evidence references (paths, keys) that caused or sustain Bootstrap mode.
* **MUST NOT** assign severities.
* **MUST NOT** affect scoring, deltas, or pass/fail status.
* **MUST NOT** be treated as Findings.

## C5. Promotion rule

Bootstrap ends only when:

* `audit/bootstrap.manifest.draft.yaml` is promoted to `audit.manifest.yaml`, AND
* Each drafted component contract is moved to its declared `docs_contract` path

Renaming/moving constitutes acceptance by an external process (human or AI). The Auditor Agent MUST NOT perform promotion.

---

# Part D — Enforcement mode (authoritative within bounds)

## D1. Audit objective (mandatory)

Record:

* Primary risk classes (safety, integrity, availability, cost)
* Decisions this audit informs
* Validity horizon of results
* Audit tier

These MUST appear in the Executive Summary

## D2. Tier declaration (normative)

Tier MUST be declared in `audit.manifest.yaml`:

```
system:
  audit:
    tier: 0 | 1 | 2 | 3

```

If missing:

* Default to Tier 0
* Emit a Medium finding

Only checks at or below the effective tier may produce hard failures.

### Tier execution rule (normative)

Checks above the effective tier MUST be **SKIPPED** and MUST NOT emit Findings.

---

# Part E — Authoritative repository artefacts

## E1. `audit.manifest.yaml` (authoritative)

### Required top-level keys

* `manifest_version` (string)
* `system` (object)
* `scope` (object)
* `components` (array)
* `journeys` (array)
* `policies` (object)
* `checks` (object)

### Validation rule (normative)

Manifest validation is defined as follows:

* **Required-keys validation (mandatory):** `MANIFEST_SCHEMA` is FAIL if any required top-level key is missing.
* **Optional JSON Schema validation (recommended):**

  * If `audit/schemas/audit.manifest.schema.json` exists, the auditor MUST additionally validate the manifest against it.
  * If schema validation fails, `MANIFEST_SCHEMA` is FAIL and the report MUST include the failing schema paths.

### System intent (authoritative rule)

System intent MUST be declared only in `audit.manifest.yaml`:

* `system.name`
* `system.description`
* `system.primary_user_goals` (required array)

The auditor MUST NOT infer system intent.

If `system.primary_user_goals` missing: High finding.

### Critical journeys (authoritative rule)

Journeys are defined exclusively in `audit.manifest.yaml` under `journeys[]`.

If missing or empty: High finding.

### Non-functional expectations (authoritative rule)

Must be declared in `audit.manifest.yaml` under `system.non_functional`.

If missing: High finding.

## E2. Component documentation contract (authoritative)

Each component MUST have a machine-parseable doc at `docs_contract`.

Missing or non-conformant `COMPONENT.yaml` is High severity.

Missing for any `criticality: critical` component is an automatic audit fail (per manifest fail conditions).

### E3. Enforcement anchors (authoritative; recommended)

Anchors are **never policy requirements**. They exist solely as deterministic proof targets for checks.

Absence of required anchors MUST NOT produce Findings and results only in **SKIPPED** checks per Part A6.

To keep AST/CFG checks deterministic across languages/frameworks, the manifest SHOULD declare enforcement anchors.

```

policies:
  anchors:
    logging:
      wrapper_symbol: []      # e.g. [{kind: "symbol", value: "log"}]
      required_fields_source: []  # e.g. [{kind: "config", value: "policies.logging_schema.required_fields"}]
    tracing:
      propagation_points: []  # middleware/wrappers
    outbound:
      http_client_wrappers: [] # wrapper modules/symbols
      timeout_config_sources: []
      retry_config_sources: []
    authn:
      enforcement_points: []   # middleware/guards at boundary
    authz:
      decision_points: []      # central policy entrypoints
    tenancy:
      enforcement_points: []
    rate_limit:
      enforcement_points: []

```

---

# Part F — Output channels

## F1. Findings (authoritative within bounds)

Findings are evidence-backed violations produced by deterministic checks.

Findings MAY affect scoring, deltas, and pass/fail (subject to mode + tier gating).

## F2. Suggestions (non-authoritative)

Suggestions:

* MUST be traceable to one of: `finding_id`, `check_id`, `skipped_check_id`, `drift_event`
* MUST reference evidence already collected
* MUST NOT affect scoring/pass-fail/deltas

## F3. Patch safety

If the auditor emits diffs:

* MUST be marked as proposed
* MUST be minimal and scoped to referenced artefacts
* MUST NOT add dependencies unless allowed by manifest (or suggestion is to add+declare)
* If write access exists, may write only to `audit/patches/*.diff`
* MUST NOT directly modify production code

---

# Part G — Scoring, deltas, and hard fail conditions (Enforcement only)

## G0. Domains (authoritative)

* **Domains are a closed vocabulary defined by the Machine Check Catalogue (Part I).**
* Each check declares exactly one `domain`.
* Domains are used only for aggregation, scoring, and reporting.
* The auditor MUST NOT invent new domains at runtime.

## G1. Maturity scoring (1–5)

### Domain identifier rule (normative)

Canonical domain identifiers are defined in Part I (§I3.1). Any human-readable domain labels appearing elsewhere in this document or tables MUST be mapped deterministically by the auditor to the canonical identifiers.

Applies only in Enforcement mode.

****Default rule (unless overridden in `audit.manifest.yaml` under `checks.scoring`):****

* Start each domain at 5
* For each **open** finding in the domain subtract:

  * 2.0 for Critical
  * 1.0 for High
  * 0.5 for Medium
  * 0.2 for Low
* Cap minimum at 1

Accepted risks do not reduce score unless expired.

### G1.1 Accepted-risk registry (optional, deterministic if present)

The repo MAY provide `audit/risk_acceptance.yaml`.

If present, entries MUST be matched deterministically using stable keys:

* `finding_id`, OR
* `delta_key`, OR
* `(primary_artefact, fingerprint)`

Each entry MUST include:

* `accepted_by` (string)
* `accepted_at` (date)
* `expires_at` (date or null)
* `reason` (string)

If absent, the Auditor Agent MUST NOT infer acceptance and MUST treat all emitted findings as `status: open`.

**Applicability rule:** Findings from SKIPPED checks do not exist and cannot affect scoring. If a domain has no applicable checks, report `N/A` and exclude from “domain score 1” fail condition unless overridden.

## G2. Repo audit state

Required:

* `audit/latest.json` (auditor-authored)

Recommended:

* `audit/open_findings.yaml` (human-authored)

### G2.1 Status vs acceptance precedence (normative)

If both `audit/open_findings.yaml` and `audit/risk_acceptance.yaml` exist:

* `audit/open_findings.yaml` is authoritative for **finding lifecycle status** (open/resolved).
* `audit/risk_acceptance.yaml` affects **scoring and penalty application only**.
* The auditor MUST NOT infer status changes from acceptance entries alone.

Optional:

* `audit/history/YYYY-MM.json`
* `audit/suppressions.yaml`

Rules:

* If `audit/latest.json` missing: treat run as baseline; emit High `AUDIT_STATE_MISSING`.
* If `audit/open_findings.yaml` missing: infer lifecycle state from `audit/latest.json`; emit Medium `OPEN_FINDINGS_MISSING`.

### Lifecycle default rule (normative)

If `audit/open_findings.yaml` is absent, then for the current run:

* All emitted Findings default to `status: open`.
* Accepted-risk penalties MUST NOT be applied (because no authoritative accepted-risk file exists).

## G3. Delta semantics (no memory)

Compare current run to `audit/latest.json`.

Classify findings as:

* New
* Still open
* Worsened
* Resolved

Worsened iff:

* Severity increases, OR
* `evidence.artefacts` set grows, OR
* `primary_artefact` unchanged AND `fingerprint` unchanged AND proof changes materially, OR
* Finding affects additional components

Drop in confidence alone does not count unless severity also increases.

## G4. Hard fail conditions (Enforcement only)

Audit fails if:

* Any applicable Critical finding exists from checks at/below effective tier
* Any applicable domain score is 1 (domains scored N/A excluded)
* Docs contracts missing for any `criticality: critical` component

---

# Part H — Deterministic applicability (noise control)

## H1. Capabilities vocabulary (recommended)

* `http`
* `http_client`
* `ui`
* `async`
* `auth`
* `authz`
* `multi_tenant`
* `external_calls`
* `data_store`
* `observability`
* `genai`
* `operability`

## H2. Applicability rule (normative)

If a check declares required capabilities and none are present across in-scope components, the check MUST be treated as **not applicable**. Outcome semantics, canonical `skip_reason` selection, and SKIPPED listing/scoring effects are defined canonically in §A6.

Capability inference is allowed only to:

* Emit `MANIFEST_CAPABILITY_MISMATCH`, or
* Provide hints

Inference MUST NOT silently apply or skip checks.

---

# Part I — Machine Check Catalogue (deterministic ABI)

## I0. Catalogue ABI guarantee (normative)

* The auditor MUST implement **at least** the checks listed in I3.
* Additional checks MAY be added, but MUST NOT weaken, bypass, replace, or redefine the minimum checks.
* `check_id` values are stable across versions.
* Removing a check or changing detector semantics is a breaking change.
* Default severities may be overridden only by manifest `checks.severity_mapping`.

### I0.1 New-check introduction rule (normative)

New checks (added in future auditor versions) MUST default to **SKIPPED** per Part A6 unless made eligible to execute via:

* Applicability (Part H), AND
* Required anchors/inputs (Parts E & I), where applicable.

This prevents auditor upgrades from creating unexpected Findings without explicit enablement.

## I1. Detector types

* PATH: file/directory existence, glob match
* TEXT: deterministic regex/text match
* AST: language-aware parse and rule match
* CFG: config parse (YAML/JSON/TOML/etc.)
* DIFF: comparison to prior audit state

## I2. Outcomes

Outcomes are defined canonically in Part A6.

## I3. Core checks (minimum set)

### I3.0 Catalogue declaration (normative)

The I3 catalogue table is **purely declarative**. It defines identifiers and required metadata only.

Canonical table columns:

* `check_id`
* `domain`
* `detector_type`
* `required_anchors`
* `default_severity`
* `confidence`
* `delta_key`

| check_id                      | domain        | detector_type | required_anchors              | default_severity | confidence | delta_key                  |
| ----------------------------- | ------------- | ------------- | ----------------------------- | ---------------- | ---------- | -------------------------- |
| MANIFEST_PRESENT              | audit_mode    | PATH          | —                             | High             | High       | repo:manifest              |
| MANIFEST_SCHEMA               | audit_mode    | CFG           | —                             | High             | High       | repo:manifest_schema       |
| MANIFEST_CAPABILITIES_MISSING | audit_mode    | CFG           | —                             | Medium           | High       | repo:manifest_caps         |
| MANIFEST_CAPABILITY_MISMATCH  | audit_mode    | AST/TEXT      | —                             | Medium           | Medium     | repo:cap_mismatch          |
| AUDIT_STATE_MISSING           | audit_mode    | PATH          | —                             | High             | High       | repo:audit_state           |
| AUDIT_STATE_CORRUPT           | audit_mode    | CFG/DIFF      | —                             | High             | High       | repo:audit_state_integrity |
| AUDIT_STATE_WRITE_FAILED      | audit_mode    | TEXT          | —                             | High             | High       | repo:audit_state_write     |
| COMPONENTS_ENUMERABLE         | system_map    | CFG           | —                             | High             | High       | comp:{id}:schema           |
| DOCS_CONTRACT_PRESENT         | cold_start    | PATH          | —                             | High             | High       | comp:{id}:docs_path        |
| DOCS_CONTRACT_SCHEMA          | cold_start    | CFG           | —                             | High             | High       | comp:{id}:docs_schema      |
| DOCS_CONTRACT_CROSSREF        | cold_start    | CFG           | —                             | High             | High       | comp:{id}:docs_refs        |
| FORBIDDEN_IMPORTS             | boundaries    | AST           | —                             | High             | High       | rule:{rule_id}:violations  |
| LAYER_VIOLATIONS              | boundaries    | AST/CFG       | —                             | High             | High       | graph:forbidden_edges      |
| MULTI_RESP_MODULE             | boundaries    | AST           | —                             | High             | Medium     | comp:{id}:multi_resp       |
| SECRETS_IN_REPO               | security      | TEXT/AST      | —                             | Critical         | High       | sec:secrets                |
| AUTHN_ENFORCEMENT             | security      | AST/TEXT      | authn.enforcement_points      | High             | Medium     | comp:{id}:authn            |
| AUTHZ_CENTRALISED             | security      | AST           | authz.decision_points         | High             | Medium     | comp:{id}:authz            |
| TENANT_ISOLATION              | security      | AST/TEXT      | tenancy.enforcement_points    | High             | Medium     | sec:tenant                 |
| OUTBOUND_TIMEOUTS             | reliability   | AST/CFG       | outbound.http_client_wrappers | High             | Medium     | rel:timeouts               |
| BOUNDED_RETRIES               | reliability   | AST/CFG       | outbound.retry_config_sources | High             | Medium     | rel:retries                |
| RATE_LIMITING                 | reliability   | AST/CFG       | rate_limit.enforcement_points | High             | Medium     | rel:ratelimit              |
| IDEMPOTENCY_ASYNC             | data          | AST/CFG       | —                             | High             | Medium     | data:idempotency           |
| LOG_SCHEMA_FIELDS             | observability | AST/CFG       | logging.wrapper_symbol        | High             | Medium     | obs:log_schema             |
| PII_REDACTION                 | observability | CFG/TEXT      | —                             | High             | Medium     | obs:redaction              |
| TRACE_PROPAGATION             | observability | AST/CFG       | tracing.propagation_points    | High             | Medium     | obs:tracing                |
| RUNBOOK_PRESENT               | operability   | PATH/TEXT     | —                             | High             | Medium     | ops:runbooks               |
| GENAI_GATEWAY_ONLY            | genai         | AST           | —                             | Critical         | High       | ai:gateway                 |
| GENAI_MODEL_ALLOWLIST         | genai         | AST/CFG       | —                             | Critical         | High       | ai:models                  |
| GENAI_COST_CAPS               | genai         | CFG/TEXT      | —                             | High             | Medium     | ai:caps                    |
| FRONTEND_CSP                  | frontend      | CFG/TEXT      | —                             | High             | Medium     | fe:csp                     |
| BUNDLE_SIZE_DRIFT             | frontend      | DIFF          | —                             | Medium           | High       | fe:bundle                  |
| DEPENDENCY_DRIFT              | system_map    | DIFF          | —                             | Medium           | High       | dep:drift                  |
| HIGH_FINDINGS_DRIFT           | delta         | DIFF          | —                             | High             | High       | delta:high                 |

## I3.1 Domain vocabulary (mandatory)

Domains are a closed vocabulary used for aggregation/scoring. Each check MUST declare exactly one domain from:

* `audit_mode`
* `system_map`
* `cold_start`
* `entry_points`
* `boundaries`
* `security`
* `reliability`
* `data`
* `observability`
* `operability`
* `genai`
* `frontend`
* `delta`

The mapping from catalogue labels to canonical identifiers is defined exclusively by this section and MUST be implemented as a fixed lookup table in the auditor.

## I3.2 Catalogue normalisation rule (mandatory)

If the catalogue table uses composite domains (e.g. "Reliability/Security"), the auditor MUST map them to a single domain + tags during execution:

* Example: `domain = reliability`, `tags = [security]`.

## I6. Proof-shape publication rule (mandatory)

This section is the canonical definition of proof-shape availability. Checks that require proof-shapes MUST reference this section and MUST NOT restate availability rules elsewhere.

For checks with non-trivial semantics (e.g. authn/authz/tenancy/idempotency), deterministic proof-shapes MUST be defined in the auditor implementation. If a check’s proof-shapes are not available for the repo’s language/framework, the check MUST be SKIPPED with reason `proof_shapes_unavailable`.

### I7. Anchor requirements for selected checks (normative)

Anchor absence handling is defined canonically in §A6 and §E3.

**Non-normative summary:** Checks listed below require specific anchors to execute; when anchors are absent, the check is SKIPPED.

* AUTHN_ENFORCEMENT → `policies.anchors.authn.enforcement_points`
* AUTHZ_CENTRALISED → `policies.anchors.authz.decision_points`
* TENANT_ISOLATION → `policies.anchors.tenancy.enforcement_points`
* OUTBOUND_TIMEOUTS → `policies.anchors.outbound.http_client_wrappers` OR `policies.anchors.outbound.timeout_config_sources`
* BOUNDED_RETRIES → `policies.anchors.outbound.retry_config_sources`
* RATE_LIMITING → `policies.anchors.rate_limit.enforcement_points`
* LOG_SCHEMA_FIELDS → `policies.anchors.logging.wrapper_symbol` AND `policies.logging_schema.required_fields`
* TRACE_PROPAGATION → `policies.anchors.tracing.propagation_points`
