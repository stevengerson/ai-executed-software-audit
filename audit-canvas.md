# Comprehensive Platform Audit Canvas

**Zero-Team / Rotating-Maintainer Edition — AI-Executed, Periodic Audit (British English)**

This canvas defines a **gold-standard, end-to-end audit instruction set** designed to be **executed periodically by an AI against a codebase and its associated artefacts**.

It assumes:

* No stable teams
* No long-term ownership
* No reliable institutional memory

The system must therefore be:

* **Structurally self-explanatory**
* **Mechanically verifiable**
* **Safe to change by unfamiliar engineers**
* **Resilient to long-term entropy**

Architecture, documentation, constraints, and runtime signals must replace people.

---

## Guiding Principles (AI-Enforced)

1. **What cannot be checked mechanically is treated as risk**.
2. **Explicit structure beats inferred intent**.
3. **Documentation is executable safety infrastructure**.
4. **Boundaries must be enforceable, not advisory**.
5. **Drift over time is more dangerous than one-off defects**.
6. **Audit strictness must be explicit and tiered to avoid adoption failure**.

---

## 0) Audit Mode, Scope, and Guarantees

### 0.1 Audit mode

This audit operates under the following model:

> **Stateless execution; state persisted in-repo.**
>
> The auditor retains no memory between runs. All lifecycle, drift, and prioritisation are computed exclusively from repository artefacts.

Accordingly:

* The AI executes periodically (e.g. monthly)
* The repository is the sole source of historical state
* Absence of required state reduces usefulness and is reported explicitly

**AI rule:** The audit MUST read `audit/latest.json` when present. It MUST read `audit/open_findings.yaml` when present, but MUST NOT assume it exists.

### 0.2 Audit objective

Explicitly record:

* Primary risk classes (safety, integrity, availability, cost)
* Decisions this audit informs
* Validity horizon of results
* **Audit tier** (see Appendix G)

**AI rule:** Only checks at or below the declared audit tier may produce hard failures.

### 0.3 Scope and exclusions

Machine-readable declaration of:

* In-scope components
* Out-of-scope components
* Assumed external guarantees

### 0.4 Evidence standard (mandatory)

Every finding MUST include:

* **Artefact** (file path, symbol, configuration key, dashboard reference)
* **Proof** (quoted value, AST match, diff, or rule violation)
* **Impact** (security, reliability, cost, safety)
* **Scope** (local or systemic)
* **Confidence** (High / Medium / Low)

### 0.5 Structural accountability (replaces ownership)

For every component, the AI must be able to identify:

* Responsibility boundary
* Forbidden responsibilities
* Declared change surface
* Declared failure responsibility

Absence of any of the above is a **High-severity finding**.

---

## 1) System Intent and Critical Journeys

### 1.1 System intent

The AI must locate a canonical declaration of:

* System purpose
* Primary user goals

### 1.2 Critical journeys

For each declared journey:

* Entry point
* Data stores touched
* External dependencies
* Expected failure impact

### 1.3 Non-functional expectations

Explicit, machine-readable declarations for:

* Latency expectations
* Availability targets
* Durability guarantees
* Privacy constraints
* Cost ceilings

---

## 2) System Map and Trust Boundaries

### 2.1 Component inventory

The AI must enumerate all deployable units and classify them by:

* Type
* Runtime
* Deployment mechanism

### 2.2 Dependency topology

The AI must extract and diff:

* Internal dependencies
* External integrations
* Newly introduced dependencies since last audit

### 2.3 Trust and tenant boundaries

The AI must identify:

* External trust boundaries
* Internal trust boundaries
* Cross-tenant data paths

---

## 3) Entry Points and Execution Paths

### 3.1 Entry-point inventory

The AI must enumerate:

* HTTP routes
* Event consumers
* Scheduled jobs
* Administrative or privileged entry points

### 3.2 Execution-path tracing (static, constrained)

Static tracing MUST be constrained and deterministic.

The AI MUST only trace within **declared trace anchors**, such as:

* Entry-point handlers declared in the manifest
* Explicit annotations (language-specific)
* Declared service/orchestrator modules

**AI rule:**

* The auditor MUST NOT attempt whole-program tracing.
* Tracing is limited to a bounded depth and declared boundaries.
* Unsupported languages/runtimes MUST emit a limitation notice, not speculative findings.

---

## 4) Architecture and Separation of Concerns

### 4.1 Boundary explicitness and orchestration locality

The audit enforces **where orchestration may occur**, not whether it exists.

The manifest MAY declare:

* Orchestrator module locations (allowed to touch DB + network)
* Domain module locations (forbidden from DB + network)

**AI rule:**

* DB + external network access in forbidden locations is High severity.
* Access in declared orchestrator locations is allowed.

### 4.2 Forbidden dependencies and layer graph

The audit MUST use an explicit dependency graph model.

The manifest MAY declare:

* `layers` (ordered list)
* `component_layers` (component_id → layer)
* `allowed_edges` (rules, e.g. same or lower layer)
* `edge_exceptions` (time-bounded, explicit)

**AI rule:**

* Only declared graph rules may be enforced.
* Undeclared graph policies MUST NOT be inferred.

### 4.3 Pattern uniformity (anchor-based)

Pattern uniformity (anchor-based)

Uniformity checks MUST reference **declared primitives**, not inferred similarity.

The manifest MAY declare:

* `policies.logging.wrapper_symbol`
* `policies.http_client.wrapper_module`
* `policies.validation.framework`

**AI rule:**

* The auditor checks for usage of the declared primitive.
* Absence of a declared primitive disables the uniformity check (SKIPPED).
* The auditor MUST NOT flag stylistic differences without an explicit anchor.

---

## 5) Data Architecture and Lifecycle Safety

### 5.1 Data ownership and access

The AI must verify:

* Single write owner per data store
* Declared read access
* Absence of undocumented cross-access

### 5.2 Integrity and consistency

The AI must detect:

* Transaction boundaries
* Idempotency mechanisms
* Safe event emission patterns

### 5.3 Data lifecycle

For each data store, the AI must locate declarations for:

* Classification
* Retention
* Deletion semantics
* Propagation behaviour

### 5.4 Backup, restore, and disaster recovery

The AI must flag absence of:

* Declared RPO/RTO
* Backup configuration artefacts

---

## 6) Observability and Operability

### 6.1 Logging

The AI must verify:

* Structured logging
* Required fields
* Redaction rules

### 6.2 Metrics

The AI must detect:

* Golden signal metrics
* Dependency health metrics
* Cost and usage metrics

### 6.3 Tracing

The AI must verify:

* Trace ID propagation
* Cross-service correlation

### 6.4 Operability guarantees

For each critical journey, the AI must find:

* Canonical dashboard reference
* Canonical trace example
* Documented recovery steps

Absence is **High severity**.

---

## 7) Reliability, Scale, and Failure Containment

### 7.1 SLO-driven design

The AI must verify presence of:

* SLI/SLO declarations OR explicit expectations
* Alerting configuration references

### 7.2 Resilience patterns

The AI must statically verify:

* Time-outs on external calls
* Bounded retries
* Circuit breaker usage where applicable

### 7.3 Load management and back-pressure

The AI must detect:

* Rate limits
* Queues
* Concurrency caps

### 7.4 Failure-mode table

The AI must locate a declared failure-mode table for critical dependencies.

---

## 8) Security, Privacy, and Abuse Resistance

### 8.1 Threat modelling

The AI must locate threat-model artefacts for critical flows.

### 8.2 Identity and isolation

The AI must verify:

* Auth enforcement at boundaries
* Centralised authorisation logic
* Tenant isolation guarantees

### 8.3 Secrets and key management

The AI must detect:

* Secret manager usage
* Absence of hard-coded secrets

Hard-coded secrets are **Critical severity**.

### 8.4 Platform security

The AI must verify:

* Dependency locking and scanning
* SSRF and egress constraints
* Web security headers and policies
* Rate limiting on exposed surfaces

### 8.5 Auditability

The AI must detect:

* Immutable audit logs for sensitive actions

---

## 9) Infrastructure and Deployment Safety

### 9.1 Infrastructure as code

The AI must verify:

* IaC coverage
* Absence of unmanaged infrastructure

### 9.2 CI/CD guarantees (deterministic)

The manifest MAY declare:

* `checks.required_commands` (e.g. `make test`, `npm test`, `go test ./...`)
* Expected exit codes
* Optional coverage thresholds

**AI rule:**

* Required commands must exist and be invokable.
* Exit-code-only checks are allowed; coverage is optional.

### 9.3 Environment parity

The AI must verify:

* Config validation
* Safe defaults
* Promotion controls

## 10) Front-End and Client Systems

### 10.1 Component architecture

The AI must verify:

* Component taxonomy
* Clear responsibility boundaries

### 10.2 State and data flow

The AI must detect:

* Centralised API clients
* Consistent loading and error handling

### 10.3 UX, accessibility, and performance

The AI must flag:

* Obvious accessibility violations
* Excessive bundle size growth

### 10.4 Front-end security

The AI must verify:

* Token storage strategy
* CSP presence
* Third-party script controls

---

## 11) GenAI and Machine-Learning Systems (If Applicable)

### 11.1 Execution architecture

The AI must verify:

* Centralised execution gateways
* Explicit request and response schemas

### 11.2 Governance and change safety

The AI must detect:

* Allow-listed models
* Versioned prompts and tools
* Hard constraints on cost and tokens

Violations are **Critical severity**.

### 11.3 Safety and evaluation

The AI must verify:

* Schema validation
* Prompt/tool isolation
* Evaluation artefacts

### 11.4 Cost containment

The AI must detect:

* Budget caps
* Usage attribution per feature

---

## 12) Cold-Start Operability (Critical)

### 12.1 Survivability signals

The AI must detect:

* Single-command local run documentation
* Environment bootstrap scripts
* Happy-path runbook

Absence is **High severity**.

### 12.2 Documentation contracts

Each component MUST include a machine-parseable documentation file containing:

* Purpose
* Responsibilities
* Forbidden responsibilities
* Entry points
* Data stores
* External dependencies
* Failure modes
* Safe changes
* Unsafe changes
* Roll-back procedure

Missing sections are **High severity**.

---

## 12.3 Change-risk analysis (delta-driven)

For each audit run, the AI MUST classify changes since the previous run:

* Dependency changes
* Schema/migration changes
* Auth/authz-related changes
* Tenancy-related changes
* Infrastructure changes

The AI MUST map changed files to affected components and journeys and emit a **risk delta summary**.

## 13) Scoring, State, and Fail Conditions

Scoring, State, and Fail Conditions

### 13.1 Maturity scoring (1–5)

Maturity scoring MUST be deterministic.

**Default rule (unless overridden in `audit.manifest.yaml`):**

* Start each domain at score 5
* For each **open** finding in the domain:

  * subtract:

    * 2 points for Critical
    * 1 point for High
    * 0.5 points for Medium
    * 0.2 points for Low
* Cap minimum score at 1

Accepted risks do not reduce score unless expired.

**Applicability rule:** Findings from SKIPPED checks do not exist and therefore cannot affect scoring. If a domain has no applicable checks, the AI MUST report the domain score as `N/A` and exclude it from hard-fail evaluation for “domain scores 1” (unless overridden by manifest scoring rules).

This makes the rule “any domain scores 1” mechanically enforceable.

### 13.2 Repository audit state (mandatory for periodic usefulness)

Because the auditor is stateless, the repository is responsible for preserving audit state.

**Required for full effectiveness**

* `audit/latest.json` — machine output from the most recent successful audit run

**Strongly recommended**

* `audit/open_findings.yaml` — curated lifecycle state for findings (open / accepted / resolved)

**Optional**

* `audit/history/YYYY-MM.json` — archived monthly outputs
* `audit/suppressions.yaml` — time-bounded suppressions (see Appendix D)

**AI rules:**

* If `audit/latest.json` is missing, the AI MUST treat the run as a baseline and emit `AUDIT_STATE_MISSING` (High severity).
* If `audit/open_findings.yaml` is missing, the AI MUST infer lifecycle state from `audit/latest.json` and emit `OPEN_FINDINGS_MISSING` (Medium severity).

### 13.3 Delta without memory (repo-based drift)

The AI MUST compute deltas by comparing the current run to `audit/latest.json`.

Findings MUST be categorised as:

* **New** — not present in the previous run
* **Still open** — present previously and not resolved
* **Worsened** — severity increased, scope expanded, or additional artefacts affected (deterministic rules below)
* **Resolved** — previously present and no longer detected

**Deterministic “worsened” rules**

A finding is “worsened” if, compared to the previous run:

* `severity` increases (critical > high > medium > low), OR
* `evidence.artefacts` set grows (set expansion), OR
* `primary_artefact` is unchanged AND `fingerprint` is unchanged AND `evidence.proof` changes materially (normalised diff), OR
* the finding affects additional components (new `component_id` values for the same `finding_id` family, where applicable).

A drop in `confidence` alone does NOT count as worsened unless `severity` also increases.

**Visibility rule:**

* Open findings MUST continue to be reported with stable identifiers.
* Accepted risks MUST NOT be re-argued, but MUST remain visible with reduced priority until expired.

### 13.4 Hard fail conditions

The audit automatically fails if:

* Any Critical finding is present
* Any **applicable** domain scores 1 (domains scored `N/A` are excluded)
* Documentation contracts are missing for any `criticality: critical` component

---

## 14) Final Output (AI-Generated)

### 14.1 Executive summary

* System intent
* Audit pass/fail status
* **Top 10 Now** (ranked)
* New and unresolved risks
* Drift highlights
* **Skipped checks summary** (what was not applicable and why)

### 14.2 Evidence-backed findings

For each finding:

* Artefact
* Proof
* Impact
* Recommendation
* Confidence

### 14.3 Remediation guidance

* Required fixes
* Optional improvements
* Verification signals

### 14.4 Explicit limitations

* What could not be verified statically
* Assumptions made by the AI

---

## Closing Criterion

**“Can a competent engineer, unfamiliar with this system, safely understand, modify, and operate it — as verified mechanically by this audit?”**

If not, the system is **not survivable** under zero-team conditions.

---

## Appendix A — Machine-Checkable Requirements Index

For each section, the AI must maintain a checklist of required artefacts, rules, and signals. Absence or violation must map deterministically to severity levels.

This appendix is normative: **if it cannot be checked, it cannot be trusted**.

---

## Appendix B — `audit.manifest.yaml` (Machine-Readable Audit Manifest)

To minimise inference and maximise repeatability, the repository MUST include a top-level manifest file:

* **Path:** `./audit.manifest.yaml`
* **Purpose:** Declaratively defines components, critical journeys, trust boundaries, and enforceable policy expectations.
* **AI rule:** The audit MUST treat the manifest as the source of truth for scope, criticality, and checks.

### B.1 Required structure

The manifest MUST include the following top-level keys:

* `manifest_version` (string)
* `system` (object)
* `scope` (object)
* `components` (array)
* `journeys` (array)
* `policies` (object)
* `checks` (object)

**Scope selector (required)**

`scope` MUST declare:

* `include_paths` (array of globs)
* `exclude_paths` (array of globs)
* `third_party_paths` (array of globs; vendored/generated)

The AI MUST NOT analyse files outside `include_paths` or inside `exclude_paths`/`third_party_paths`.

### B.2 Component schema (minimum fields)

Each `components[]` item MUST include:

* `id` (stable identifier)
* `name`
* `type` (api|worker|ui|job|function|library)
* `paths` (array of directories)
* `entry_points` (array; routes/handlers/commands/triggers)
* `data_stores` (array; named stores it touches)
* `external_dependencies` (array; named integrations)
* `trust_boundary` (public|internal|privileged)
* `docs_contract` (path to machine-parseable component doc)
* `criticality` (low|medium|high|critical)
* `capabilities` (array of strings; see Appendix F)

**AI rule:** If `capabilities` is missing for any component, emit `MANIFEST_CAPABILITIES_MISSING` (Medium severity) and proceed using capability inference (non-authoritative) with reduced confidence.

### B.3 Journey schema (minimum fields)

Each `journeys[]` item MUST include:

* `id`
* `name`
* `entry_points` (references to component entry points)
* `data_stores`
* `external_dependencies`
* `failure_impact` (low|medium|high|critical)
* `slos` (optional but recommended)

### B.4 Policy expectations

`policies` MUST declare expectations the AI can validate, for example:

* `forbidden_dependencies` (by pattern)
* `required_security_headers`
* `secrets_detection` (tooling / patterns)
* `genai` (gateway paths, allow-listed models, token/cost caps)
* `logging_schema` (required fields)

### B.5 Deterministic fail rules

`checks.fail_conditions` MUST declare hard gates, e.g.:

* any Critical finding
* any domain score of 1
* missing docs contracts for critical components

### B.6 Applicability rules (to avoid non-applicable noise)

To prevent false positives in repos where certain concerns do not apply (e.g. no HTTP, no UI, no multi-tenancy), the manifest MAY define deterministic applicability rules.

* **Location:** `checks.applicability`
* **Form:** per-check rules using required capabilities

Example:

```yaml
checks:
  applicability:
    RATE_LIMITING:
      requires: ["http"]
    FRONTEND_CSP:
      requires: ["ui"]
    TENANT_ISOLATION:
      requires: ["multi_tenant"]
    IDEMPOTENCY_ASYNC:
      requires: ["async"]
    GENAI_GATEWAY_ONLY:
      requires: ["genai"]
```

**AI rule:** If a check’s required capabilities are absent across all declared components, the check MUST be marked **SKIPPED** (not PASS), MUST be listed in output, and MUST NOT affect scoring.

### B.7 Example `audit.manifest.yaml`

```yaml
manifest_version: "1.0"

system:
  name: "Example Platform"
  description: "A multi-component platform with API, workers, and web UI."
  repository:
    default_branch: "main"
  audit:
    cadence: "weekly"
    mode: "ai_periodic"

components:
  - id: "api"
    name: "Public API"
    type: "api"
    paths:
      - "services/api"
    entry_points:
      - kind: "http"
        pattern: "services/api/src/routes/**"
      - kind: "http"
        pattern: "services/api/src/controllers/**"
    data_stores:
      - "postgres_primary"
      - "redis_cache"
    external_dependencies:
      - "stripe"
      - "sendgrid"
    trust_boundary: "public"
    docs_contract: "services/api/COMPONENT.yaml"
    criticality: "critical"
    capabilities: ["http", "auth", "authz", "external_calls", "multi_tenant", "observability"]

  - id: "worker"
    name: "Background Worker"
    type: "worker"
    paths:
      - "services/worker"
    entry_points:
      - kind: "queue"
        pattern: "services/worker/src/consumers/**"
      - kind: "cron"
        pattern: "services/worker/src/jobs/**"
    data_stores:
      - "postgres_primary"
    external_dependencies:
      - "s3"
    trust_boundary: "internal"
    docs_contract: "services/worker/COMPONENT.yaml"
    criticality: "high"
    capabilities: ["async", "external_calls", "observability"]

  - id: "web"
    name: "Web UI"
    type: "ui"
    paths:
      - "apps/web"
    entry_points:
      - kind: "route"
        pattern: "apps/web/src/pages/**"
    data_stores: []
    external_dependencies:
      - "public_api"
    trust_boundary: "public"
    docs_contract: "apps/web/COMPONENT.yaml"
    criticality: "high"
    capabilities: ["ui", "http_client", "observability"]

journeys:
  - id: "signup"
    name: "User Sign-up"
    entry_points:
      - component: "web"
        ref: "apps/web/src/pages/signup*"
      - component: "api"
        ref: "POST /v1/users"
    data_stores:
      - "postgres_primary"
    external_dependencies:
      - "sendgrid"
    failure_impact: "high"
    slos:
      availability: "99.9%"
      latency_p95_ms: 400

policies:
  forbidden_dependencies:
    - rule_id: "no_provider_sdks_outside_gateways"
      description: "Provider SDKs may only be imported within approved gateway modules."
      patterns:
        - forbidden_import: "stripe"
          allowed_paths:
            - "services/api/src/integrations/stripe/**"
        - forbidden_import: "openai"
          allowed_paths:
            - "services/api/src/genai/gateway/**"

  logging_schema:
    required_fields:
      - "timestamp"
      - "level"
      - "message"
      - "service"
      - "environment"
      - "requestId"
      - "traceId"

  required_security_headers:
    - "content-security-policy"
    - "strict-transport-security"
    - "x-content-type-options"

  secrets_detection:
    tools:
      - "gitleaks"
    fail_on_findings: true

  genai:
    enabled: true
    gateway_paths:
      - "services/api/src/genai/gateway/**"
    allow_listed_models:
      - "gpt-4.1"
      - "gpt-4o-mini"
    caps:
      max_tokens: 2048
      max_cost_usd_per_day: 50

checks:
  applicability:
    RATE_LIMITING:
      requires: ["http"]
    FRONTEND_CSP:
      requires: ["ui"]
    TENANT_ISOLATION:
      requires: ["multi_tenant"]
    IDEMPOTENCY_ASYNC:
      requires: ["async"]
    GENAI_GATEWAY_ONLY:
      requires: ["genai"]

  fail_conditions:
    - id: "critical_finding_present"
      description: "Any Critical finding fails the audit."
    - id: "domain_score_one"
      description: "Any domain scored as 1 fails the audit."
    - id: "missing_docs_contract_for_critical_component"
      description: "Missing docs contract for any critical component fails the audit."

  severity_mapping:
    hard_coded_secrets: "critical"
    genai_outside_gateway: "critical"
    forbidden_dependency_edge: "high"
    missing_operability_guarantees: "high"
    missing_docs_section: "high"
```

### B.7 Notes for implementers

* Keep `id` values stable across time to enable drift detection.
* Prefer glob patterns the AI can match deterministically.
* Treat `docs_contract` as mandatory for all production components.
* Encode policy in the manifest whenever possible; avoid prose-only rules.

---

## Appendix C — `COMPONENT.yaml` (Machine-Parseable Component Documentation Contract)

To make the platform survivable without hand-over, each production component MUST include a **machine-parseable component contract**.

* **Purpose:** Replace tribal knowledge with a deterministic, auditable source of truth.
* **AI rule:** Missing or non-conformant `COMPONENT.yaml` is **High severity**; missing for any `criticality: critical` component is an **automatic audit fail** (per manifest fail conditions).

### C.1 File location

* Each component MUST include the file at the path referenced by `docs_contract` in `audit.manifest.yaml`.
* Recommended filename: `COMPONENT.yaml`.

### C.2 Required schema (minimum fields)

Each `COMPONENT.yaml` MUST include:

* `contract_version` (string)
* `component` (object)

  * `id` (must match `audit.manifest.yaml` component id)
  * `name`
  * `type` (api|worker|ui|job|function|library)
  * `responsibilities` (array of statements)
  * `forbidden_responsibilities` (array of statements)
* `interfaces` (object)

  * `entry_points` (array)
  * `public_api` (optional; paths, OpenAPI/GraphQL refs)
  * `admin_entry_points` (optional; privileged surfaces)
* `data` (object)

  * `stores` (array; must be subset of manifest `data_stores`)
  * `mutations` (array; primary write operations)
  * `consistency` (transactional|eventual|mixed)
  * `idempotency` (required if any async processing)
* `dependencies` (object)

  * `internal` (array)
  * `external` (array; must be subset of manifest `external_dependencies`)
  * `timeouts_retries` (required for outbound calls)
* `security` (object)

  * `authn` (required; how authentication is enforced)
  * `authz` (required; how authorisation decisions are made)
  * `tenant_isolation` (required if multi-tenant)
  * `secrets` (required; where secrets come from)
  * `data_classification` (required; what sensitive data exists)
* `operability` (object)

  * `dashboards` (array; named references/links)
  * `alerts` (array; named references)
  * `runbooks` (array; paths)
  * `logging` (required fields and redaction notes)
  * `tracing` (correlation/propagation notes)
* `failure_modes` (array)

  * Each item: `dependency`, `failure`, `user_impact`, `detection`, `mitigation`
* `change_safety` (object)

  * `safe_changes` (array)
  * `unsafe_changes` (array)
  * `roll_back` (step-by-step)
  * `verification` (how to confirm success in production)
* `local_dev` (object)

  * `prerequisites`
  * `run` (single command)
  * `smoke_test` (single command)

### C.3 Deterministic validation rules

The AI MUST enforce:

* `component.id` matches the manifest component `id`.
* All referenced data stores and external dependencies are declared in the manifest.
* If any `interfaces.admin_entry_points` exist, `security.authz` MUST include an explicit admin policy.
* If `type` is `worker` or any queue/cron entry points exist, `data.idempotency` MUST be present.
* If any `dependencies.external` exist, `dependencies.timeouts_retries` MUST specify time-outs and retry bounds.
* `operability.dashboards` and `operability.runbooks` MUST be non-empty for `criticality: high|critical`.

### C.4 Example `COMPONENT.yaml`

```yaml
contract_version: "1.0"

component:
  id: "api"
  name: "Public API"
  type: "api"
  responsibilities:
    - "Serve authenticated HTTP APIs for end-user and partner clients."
    - "Validate inputs, enforce tenancy, and orchestrate domain operations."
  forbidden_responsibilities:
    - "Direct access to third-party provider SDKs outside integration gateways."
    - "Running long-lived batch jobs or scheduled tasks."

interfaces:
  entry_points:
    - kind: "http"
      locations:
        - "services/api/src/routes/**"
  public_api:
    - kind: "openapi"
      ref: "services/api/openapi.yaml"
  admin_entry_points:
    - kind: "http"
      pattern: "POST /admin/*"

data:
  stores:
    - "postgres_primary"
    - "redis_cache"
  mutations:
    - "CreateUser"
    - "UpdateSubscription"
  consistency: "mixed"
  idempotency:
    strategy: "idempotency-key"
    where:
      - "POST /v1/subscriptions"

dependencies:
  internal:
    - "worker"
  external:
    - "stripe"
    - "sendgrid"
  timeouts_retries:
    outbound:
      default_timeout_ms: 3000
      retries:
        max_attempts: 3
        backoff: "exponential_jitter"
        retry_on:
          - "timeout"
          - "5xx"
        require_idempotency_key_for_retries: true

security:
  authn:
    mechanism: "bearer_jwt"
    enforcement_points:
      - "services/api/src/middleware/auth/**"
  authz:
    model: "central_policy"
    policy_locations:
      - "services/api/src/authz/policy/**"
    admin_policy_required: true
  tenant_isolation:
    guarantee: "orgId enforced on every request and propagated to DB and cache keys."
    enforcement_points:
      - "services/api/src/middleware/tenant/**"
  secrets:
    sources:
      - "environment"
      - "secret_manager"
    never_in:
      - "repo"
      - "logs"
  data_classification:
    pii_fields:
      - "email"
      - "name"
    retention_days: 365
    deletion:
      strategy: "hard_delete_with_propagation"

operability:
  dashboards:
    - name: "API Golden Signals"
      ref: "observability/dashboards/api-golden-signals"
  alerts:
    - name: "API 5xx rate high"
      ref: "observability/alerts/api-5xx"
  runbooks:
    - "runbooks/api.md"
  logging:
    required_fields:
      - "timestamp"
      - "level"
      - "message"
      - "service"
      - "environment"
      - "requestId"
      - "traceId"
    redaction:
      - "email"
      - "auth_token"
  tracing:
    propagation: "traceId passed via standard headers and included in logs"

failure_modes:
  - dependency: "stripe"
    failure: "timeout"
    user_impact: "Subscription creation may fail or be delayed."
    detection: "Increase in payment latency and error-rate metrics; traces show Stripe span time-outs."
    mitigation: "Use idempotency keys; retry bounded; degrade to pending state; support replay."

change_safety:
  safe_changes:
    - "Add a new endpoint following the standard route/controller pattern."
    - "Add a new field using expand/contract migration steps."
  unsafe_changes:
    - "Changing tenancy enforcement logic."
    - "Modifying payment retry semantics."
  roll_back:
    - "Revert deployment to previous release tag."
    - "Disable feature flag 'new_subscription_flow'."
    - "Verify error rates return to baseline within 10 minutes."
  verification:
    - "Trace a sign-up journey end-to-end and confirm orgId propagation."
    - "Confirm API p95 latency is within SLO for 30 minutes."

local_dev:
  prerequisites:
    - "Docker"
    - "Make"
  run: "make dev"
  smoke_test: "make smoke"
```

### C.5 Notes for implementers

* Keep statements short and testable; avoid vague prose.
* Use stable names for dependencies and data stores to support drift detection.
* Prefer referencing concrete file paths for enforcement points.
* Treat `COMPONENT.yaml` as the authoritative contract; keep it current via CI checks.

---

## Appendix D — Machine Check Catalogue (Deterministic Rules)

This catalogue defines **what the AI checks**, **how it detects it**, and **how results are scored and compared over time**.

### D.1 Catalogue conventions

* **Detector types**

  * `PATH`: file/directory existence, glob match
  * `TEXT`: deterministic text/regex match
  * `AST`: language-aware parse and rule match (imports, calls, annotations)
  * `CFG`: configuration file parse (YAML/JSON/TOML/etc.)
  * `DIFF`: comparison to prior audit state stored in-repo

* **Check outcomes**

  * `FAIL`: violation detected (produces a finding)
  * `PASS`: check executed and no violation detected
  * `SKIPPED`: check not applicable (required capabilities absent)

**AI rule:** SKIPPED checks MUST be listed in output, MUST NOT create findings, and MUST NOT affect scoring.

* **Confidence rules**

  * High: direct match (AST/CFG/PATH) with unambiguous proof
  * Medium: strong heuristic (TEXT) with limited ambiguity
  * Low: inference required; MUST be reported under “Explicit limitations”

* **Severity rules**

  * If `checks.severity_mapping` exists in `audit.manifest.yaml`, it overrides defaults below.

### D.2 Core checks (minimum set)

> The AI MUST implement at least these checks. Additional checks may be added, but may not weaken or bypass these.

| Check ID                      | Domain                 | What is checked                                                       | Detector  | Evidence (proof)                         | Default severity                        | Confidence | Delta key                  |
| ----------------------------- | ---------------------- | --------------------------------------------------------------------- | --------- | ---------------------------------------- | --------------------------------------- | ---------- | -------------------------- |
| MANIFEST_PRESENT              | Audit mode             | `audit.manifest.yaml` exists at repo root                             | PATH      | Path exists                              | critical                                | High       | repo:manifest              |
| MANIFEST_SCHEMA               | Audit mode             | Manifest contains required top-level keys                             | CFG       | Parsed keys + missing list               | high                                    | High       | repo:manifest_schema       |
| MANIFEST_CAPABILITIES_MISSING | Audit mode             | Any component missing `capabilities`                                  | CFG       | Component ids missing capabilities       | medium                                  | High       | repo:manifest_caps         |
| MANIFEST_CAPABILITY_MISMATCH  | Audit mode             | Inferred capability present in code but missing from manifest         | AST/TEXT  | Evidence of capability + component id    | medium                                  | Medium     | repo:cap_mismatch          |
| AUDIT_STATE_MISSING           | Audit mode             | `audit/latest.json` missing (baseline run)                            | PATH      | Missing path                             | high                                    | High       | repo:audit_state           |
| AUDIT_STATE_CORRUPT           | Audit mode             | `audit/latest.json` fails integrity check (`content_hash` mismatch)   | CFG/TEXT  | Previous hash + recomputed hash mismatch | high                                    | High       | repo:audit_state_integrity |
| AUDIT_STATE_WRITE_FAILED      | Audit mode             | Auditor cannot write `audit/latest.json`                              | TEXT      | Explicit write failure signal            | high                                    | High       | repo:audit_state_write     |
| COMPONENTS_ENUMERABLE         | System map             | All components declared with required fields                          | CFG       | Component list + missing fields          | high                                    | High       | comp:{id}:schema           |
| DOCS_CONTRACT_PRESENT         | Cold-start             | `docs_contract` file exists per component                             | PATH      | Missing paths list                       | high (critical comps => fail condition) | High       | comp:{id}:docs_path        |
| DOCS_CONTRACT_SCHEMA          | Cold-start             | `COMPONENT.yaml` conforms to required schema                          | CFG       | Missing keys list                        | high                                    | High       | comp:{id}:docs_schema      |
| DOCS_CONTRACT_CROSSREF        | Cold-start             | Docs contract references only manifest stores/deps                    | CFG       | Invalid refs list                        | high                                    | High       | comp:{id}:docs_refs        |
| ENTRYPOINTS_MATCH             | Entry points           | Entry points in manifest map to actual code locations                 | PATH/TEXT | Unmatched patterns list                  | high                                    | Medium     | comp:{id}:entrypoints      |
| FORBIDDEN_IMPORTS             | Boundaries             | Disallowed imports absent outside allowed paths                       | AST       | Offending file paths + import symbols    | high                                    | High       | rule:{rule_id}:violations  |
| LAYER_VIOLATIONS              | Boundaries             | Dependency graph contains forbidden edges (layer rules)               | AST/CFG   | Edge list (from → to)                    | high                                    | High       | graph:forbidden_edges      |
| MULTI_RESP_MODULE             | Boundaries             | Module touches DB and external network without explicit orchestration | AST       | Call/import evidence                     | high                                    | Medium     | comp:{id}:multi_resp       |
| SECRETS_IN_REPO               | Security               | Hard-coded secrets detected                                           | TEXT/AST  | Snippet hash + file path                 | critical                                | High       | sec:secrets                |
| AUTHN_ENFORCEMENT             | Security               | Auth middleware/guard exists on public boundaries                     | AST/TEXT  | Wiring evidence                          | high                                    | Medium     | comp:{id}:authn            |
| AUTHZ_CENTRALISED             | Security               | Authorisation flows through central policy                            | AST       | Call graph evidence                      | high                                    | Medium     | comp:{id}:authz            |
| TENANT_ISOLATION              | Security               | Tenant identifier enforced and propagated                             | AST/TEXT  | Enforcement points                       | high                                    | Medium     | sec:tenant                 |
| OUTBOUND_TIMEOUTS             | Reliability            | Outbound calls have explicit time-outs                                | AST/CFG   | Wrapper usage or args                    | high                                    | Medium     | rel:timeouts               |
| BOUNDED_RETRIES               | Reliability            | Retries bounded and idempotency-aware                                 | AST/CFG   | Policy config                            | high                                    | Medium     | rel:retries                |
| RATE_LIMITING                 | Reliability/Security   | Rate limiting exists on public entry points                           | AST/CFG   | Middleware/config evidence               | high                                    | Medium     | rel:ratelimit              |
| IDEMPOTENCY_ASYNC             | Data/Reliability       | Consumers implement idempotency/deduplication                         | AST/CFG   | Strategy evidence                        | high                                    | Medium     | data:idempotency           |
| LOG_SCHEMA_FIELDS             | Observability          | Logging includes required fields                                      | AST/CFG   | Logger wrapper evidence                  | high                                    | Medium     | obs:log_schema             |
| PII_REDACTION                 | Observability/Security | Redaction/scrubbing rules present                                     | CFG/TEXT  | Redaction config                         | high                                    | Medium     | obs:redaction              |
| TRACE_PROPAGATION             | Observability          | Trace IDs propagated and included in logs                             | AST/CFG   | Middleware evidence                      | high                                    | Medium     | obs:tracing                |
| RUNBOOK_PRESENT               | Operability            | Runbook files exist and include recovery steps                        | PATH/TEXT | Missing files; recovery heading          | high                                    | Medium     | ops:runbooks               |
| GENAI_GATEWAY_ONLY            | GenAI                  | Provider SDK imports only inside declared gateway                     | AST       | Offending imports                        | critical                                | High       | ai:gateway                 |
| GENAI_MODEL_ALLOWLIST         | GenAI                  | Models restricted to allow-list                                       | AST/CFG   | Offending model strings                  | critical                                | High       | ai:models                  |
| GENAI_COST_CAPS               | GenAI                  | Hard caps on tokens/cost exist                                        | CFG/TEXT  | Cap values                               | high                                    | Medium     | ai:caps                    |
| FRONTEND_CSP                  | Front-end              | CSP exists and is non-trivial                                         | CFG/TEXT  | Header/meta evidence                     | high                                    | Medium     | fe:csp                     |
| BUNDLE_SIZE_DRIFT             | Front-end              | Bundle size change beyond threshold                                   | DIFF      | Size delta + threshold                   | medium                                  | High       | fe:bundle                  |
| DEPENDENCY_DRIFT              | System map             | New external dependencies since last audit                            | DIFF      | New dependency list                      | medium (escalate by criticality)        | High       | dep:drift                  |
| HIGH_FINDINGS_DRIFT           | Delta                  | New High/Critical findings since last audit                           | DIFF      | Delta list                               | high                                    | High       | delta:high                 |

### D.3 Applicability and delta rules

**Applicability**

* A check with `checks.applicability.<CHECK_ID>.requires` MUST be executed only if at least one in-scope component declares all required capabilities.
* Otherwise, the check outcome is **SKIPPED**.

**Capability inference (allowed, non-authoritative)**

The auditor MAY infer capabilities from code signals (e.g. route definitions → `http`, React entrypoints → `ui`, queue consumers → `async`).

* If inferred capability exists but is not declared in the relevant component’s `capabilities`, emit `MANIFEST_CAPABILITY_MISMATCH` (Medium severity).
* Inference MUST NOT be used to mark checks as applicable unless the manifest is missing `capabilities` entirely; in that case, proceed with reduced confidence and emit `MANIFEST_CAPABILITIES_MISSING`.

**Delta**

* The AI MUST produce a **delta summary** using `audit/latest.json` as the baseline.
* The AI MAY support suppression, but only via explicit configuration:

  * `audit/suppressions.yaml`
  * Each suppression MUST include:

    * `check_id`
    * `artefact` (file/path/rule)
    * `reason`
    * `expiry` (date)

Expired suppressions MUST be treated as violations.

### D.4 Output determinism requirements

To keep periodic audits comparable, the AI MUST:

* Emit stable IDs for findings using semantic targets where possible.

**Finding identity rules:**

* Prefer semantic identifiers: `{check_id}:{component_id}:{rule_id|symbol}`
* Fall back to file paths only when no semantic target exists
* Each finding MUST include a separate `fingerprint` field derived from a stable AST/config signature to survive refactors

### D.5 Extending the catalogue

When adding checks:

* Prefer AST/CFG/PATH detectors over TEXT
* Define an explicit delta key
* Declare default severity and confidence rules
* Ensure checks cannot be satisfied by documentation alone when runtime behaviour is required

---

## Appendix E — Audit State, Finding Lifecycle, and Output Contract

Because the AI has no memory between runs, **state must live in the repository**. This appendix defines the required state files and the output format the AI must emit.

### E.1 `audit/latest.json` (required)

* **Purpose:** Baseline for comparison and prioritisation.
* **Update rule:** Each successful run SHOULD overwrite `audit/latest.json`.

**Authority rule**

* `audit/latest.json` is **AI-authored only**.
* Humans and automation MUST NOT edit this file manually.
* Any manual modification invalidates drift semantics and MUST be treated as corruption.

**Write-failure fallback**

If the auditor cannot write to the repository, it MUST:

* Emit `AUDIT_STATE_WRITE_FAILED` (High severity)
* Output a complete baseline report
* Include the full contents of the would-be `audit/latest.json` as part of the audit output, suitable for manual or CI-assisted commit

**Minimum schema (required keys)**

`audit/latest.json` MUST be valid JSON and MUST include:

* `schema_version` (string)
* `generated_by` (string; e.g. auditor name)
* `tool_version` (string)
* `repo_revision` (string; commit SHA or equivalent)
* `audit_run` (object)

  * `run_id` (string)
  * `timestamp_utc` (string)
  * `manifest_version` (string)
* `scores` (object)

  * `domains` (object; domain → score 1–5 or `N/A`)
* `summary` (object)

  * `pass` (boolean)
  * `top_findings` (array of `finding_id`)
  * `new_findings` (array of `finding_id`)
  * `worsened_findings` (array of `finding_id`)
  * `resolved_findings` (array of `finding_id`)
  * `skipped_checks` (array of objects)

    * each item: `check_id`, `reason`, `requires` (capabilities)
* `findings` (array of objects)

  * each finding MUST include:

    * `finding_id` (string; stable)
    * `check_id` (string)
    * `domain` (string)
    * `severity` (critical|high|medium|low)
    * `confidence` (high|medium|low)
    * `component_id` (string; if applicable)
    * `primary_artefact` (string)
    * `fingerprint` (string; stable-ish AST/CFG signature)
    * `evidence` (object)

      * `artefacts` (array of strings; related paths/symbols/config keys)
      * `proof` (object or string; deterministic proof payload)
    * `impact` (string)
    * `recommendation` (string)
    * `priority_score` (number)
    * `status` (open|in_progress|accepted_risk|resolved)

**Status authority rule**

* `findings[].status` represents the **effective status** after applying `audit/open_findings.yaml` when present.

* If `audit/open_findings.yaml` is absent, all findings default to `open`.

  * `status` (open|in_progress|accepted_risk|resolved)

* `content_hash` (string)

  * Deterministic hash over the normalised `findings` array (excluding volatile fields such as timestamps).

**State integrity rule (deterministic)**

On each run, the auditor MUST:

* Recompute `content_hash` for the previous `audit/latest.json`.
* If the hash does not match, emit `AUDIT_STATE_CORRUPT` (High severity) and treat deltas as unreliable (baseline run).

### E.2 `audit/open_findings.yaml` (recommended)

* **Purpose:** Human-curated lifecycle state so the AI does not repeatedly re-argue known items.
* **Authority rule:** This file is **human-authored**.
* **AI rule:** This file is authoritative for finding status when present.

**Minimum schema**

```yaml
version: "1.0"
findings:
  - finding_id: "SECRETS_IN_REPO:services/api/.env:abcd1234"
    status: "open"           # open|in_progress|accepted_risk|resolved
    reason: ""               # required when status is accepted_risk
    expires: ""              # required when status is accepted_risk (YYYY-MM-DD)
    links:                    # optional
      prs: []
      issues: []
```

**Rules**

* `accepted_risk` MUST include `reason` and `expires`.
* Expired `accepted_risk` entries MUST be treated as `open`.
* `resolved` entries MAY be retained for history; the AI should not prioritise them unless regression occurs.

### E.3 Output contract for “Top things to address now”

The AI MUST emit a prioritised summary optimised for human action:

1. **Top 10 Now** — ranked list
2. **New since last run** — subset
3. **Still open (de-prioritised)**
4. **Resolved**
5. **Skipped checks** — what was not applicable and why

**Default prioritisation weights (unless overridden in `audit.manifest.yaml`):**

* severity_weight:

  * critical: 100
  * high: 30
  * medium: 10
  * low: 3
* confidence_weight:

  * high: 1.0
  * medium: 0.7
  * low: 0.4
* criticality_weight:

  * critical: 2.0
  * high: 1.5
  * medium: 1.2
  * low: 1.0
* boost_new: +20%
* boost_worsened: +30%
* accepted_risk_penalty: −80% (ignored if expired)

**Accepted-risk absence rule**

If `audit/open_findings.yaml` is absent, treat all previously present findings as `open` and apply no accepted-risk penalty.

### E.4 Minimal human workflow (no AI memory assumed)

* Run audit → AI writes `audit/latest.json`.
* Human reviews “Top 10 Now”.
* Human updates `audit/open_findings.yaml` statuses as work starts/completes.
* Next month, AI reads both files and produces deltas and re-prioritisation.

---

## Appendix F — Capabilities and Applicability (Noise Control)

Applicability prevents the auditor from producing misleading “missing X” findings when X does not apply.

### F.1 Capability vocabulary (recommended)

Components MAY declare any string capabilities, but the following vocabulary is recommended for portability:

* `http` — HTTP server/routes
* `http_client` — makes outbound HTTP requests
* `ui` — browser/client UI
* `async` — queue consumers, schedulers, background processing
* `auth` — authentication boundary enforcement
* `authz` — authorisation decisions
* `multi_tenant` — tenant isolation required
* `external_calls` — calls to third-party services/providers
* `data_store` — persistent database usage
* `observability` — emits logs/metrics/traces
* `genai` — calls LLM/GenAI providers
* `admin_surface` — privileged/admin entry points

### F.2 Deterministic applicability rule

* If a check declares required capabilities and none are present across in-scope components, the check outcome MUST be **SKIPPED**.
* SKIPPED checks MUST be reported (with reason) and MUST NOT affect scoring.

### F.3 Capability inference (non-authoritative)

The auditor MAY infer capabilities from code. Inference is used only to:

* Detect `MANIFEST_CAPABILITY_MISMATCH` (Medium), or
* Provide hints for improving the manifest.

Inference MUST NOT be used to silently skip or silently apply checks.

---

## Appendix G — Audit Tiers (Adoption Control)

To avoid adoption failure and enable incremental rollout, audits are tiered.

### G.1 Tier definitions

* **Tier 0 (Bootstrap)**

  * Manifest present
  * Component docs present
  * Secrets scanning
  * Forbidden imports
  * Dependency locking
  * Entrypoints match manifest
  * Audit state files

* **Tier 1 (Boundaries & Security)**

  * Authn/authz anchors
  * Tenancy enforcement
  * CSP / security headers
  * Timeout/retry wrappers

* **Tier 2 (Operability)**

  * Runbooks
  * Dashboards
  * Alerts
  * Golden signals
  * Trace propagation

* **Tier 3 (Resilience & Drift)**

  * SLOs
  * Failure-mode tables
  * Load/back-pressure
  * GenAI governance
  * Bundle and dependency drift

### G.2 Tier enforcement

* The manifest MAY declare:

```yaml
audit:
  tier: 1
```

* Checks above the declared tier MUST be marked **SKIPPED**.
* Hard-fail conditions apply only to checks at or below the declared tier.

---

## Appendix H — Emerging Primitives and Fragmentation (Advisory)

This appendix defines **non-enforcing detection of emerging architectural primitives and pattern fragmentation**.

It exists to surface **early architectural drift** without inferring intent or modifying policy.

**Applicability rule (mandatory):**

* If `MANIFEST_PRESENT` is FAIL, Appendix H MUST be **SKIPPED**.
* If `MANIFEST_SCHEMA` is FAIL, Appendix H MUST be **SKIPPED**.
* If `COMPONENTS_ENUMERABLE` is FAIL, Appendix H MUST be **SKIPPED**.

---

### H.1 Scope and guarantees

This mechanism is:

* Advisory only
* Non-enforcing
* Non-scoring
* Non-blocking

It MUST NOT:

* Create findings
* Assign severity
* Affect domain scores
* Trigger fail conditions
* Modify applicability rules
* Introduce or enforce policy

Declared primitives in `audit.manifest.yaml` remain the **only enforceable primitives**.

---

### H.2 Eligible pattern categories

The auditor MAY analyse only the following bounded categories:

* Logging initialisation and usage
* Outbound HTTP / RPC clients
* Input validation frameworks
* Error construction and propagation
* Authentication / authorisation middleware
* Database access patterns
* GenAI invocation patterns (only if `genai` capability is declared)

The auditor MUST NOT perform unconstrained similarity analysis or cross-category inference.

---

### H.3 Detection model (deterministic, bounded)

For each eligible category, the auditor MAY:

1. Identify distinct patterns using:

   * Imported module or package
   * Wrapper or factory symbol
   * Call signature shape
   * Configuration object shape

2. Group callsites by stable pattern identifier.

3. Record:

   * Total callsites per pattern
   * Components affected
   * Change since the previous run (if baseline exists)

**Rules**

* Detection MUST rely on PATH, AST, or CFG signals.
* TEXT heuristics MAY be used only where structural signals are unavailable and MUST lower confidence.
* Whole-program inference is forbidden.
* Pattern identifiers MUST be derived from a deterministic tuple, e.g. {category}:{import_or_symbol}:{normalised_signature}.

---

### H.4 Candidate surfacing thresholds

A pattern MAY be surfaced as an **emerging primitive candidate** only if all of the following are true:

* Appears in ≥ *N* callsites (default: 10)
* Appears in ≥ *M* components (default: 2)
* AND at least one of:

  * Usage increased since the previous run
  * Multiple distinct patterns exist in the same category

Thresholds MAY be overridden in `audit.manifest.yaml`:

```yaml
checks:
  emerging_primitives:
    min_callsites: 10
    min_components: 2
```

---

### H.5 Output requirements

If any candidates are surfaced, the auditor MUST emit a dedicated section:

#### Emerging Primitives and Fragmentation (Advisory)

For each eligible category, the output MUST include:

* Category
* Detected patterns (stable identifiers)
* Usage counts (current and previous run, if available)
* Components affected
* Trend (increasing | decreasing | stable)
* Evidence references (file paths or symbols)
* Confidence (High | Medium | Low)

No interpretation or prescriptive language is permitted.

---

### H.6 State persistence (optional)

To support trend analysis, the auditor MAY write:

```
audit/emerging_primitives.json
```

**Rules**

* AI-authored only
* Read on subsequent runs if present
* Absence MUST NOT produce findings or warnings

---

### H.7 Non-interference rules (mandatory)

The auditor MUST ensure:

* No enforcement occurs
* No implicit primitive declaration occurs
* No policy changes occur automatically
* Promotion to an enforceable primitive requires explicit declaration in `audit.manifest.yaml`

---

### H.8 Interpretation boundary

Surfaced candidates indicate **pattern convergence or fragmentation**, not correctness.

Architectural intent remains human-declared.

---

### Closing rule

**The auditor may observe structure. Only humans may declare it.**
