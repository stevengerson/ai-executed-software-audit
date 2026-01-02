# Platform Audit Canvas

**A deterministic audit framework with an LLM acting as a bounded execution engine—not an authority.**

A tiered audit specification for making software platforms **mechanically understandable, enforceably constrained, and resilient to change**—even when ownership, context, and institutional memory do not persist.

This repository contains the **canonical Platform Audit Canvas**: an audit instruction set designed to be executed **periodically by an automated auditor** against a codebase and its declared artefacts.

It is **not a tool**.
It is an **audit contract**.

---

## Why this exists

Modern software rarely fails all at once.
It becomes unsafe to change long before anything visibly breaks.

Assumptions remain implicit. Boundaries erode. Temporary decisions harden into production paths. Documentation drifts. These risks are amplified in fast-moving, AI-accelerated development environments.

Over time, systems become hard to understand, risky to modify, and difficult to assess against security, reliability, or cost expectations.

The Platform Audit Canvas exists to **surface these risks early**, before they turn into incidents or irreversible loss of system understanding.

---

## What this is (and is not)

This canvas defines a **structured, deterministic way for a repository to explain itself**.

> **Deterministic**, in this context, means that audit outcomes are driven exclusively by declared artefacts and rule-based detectors.
> The auditor may analyse and report evidence, but it does not invent policy, infer intent authoritatively, or make autonomous decisions.

A system declares:

* system intent and critical journeys
* component boundaries and responsibilities
* enforceable rules and constraints

An automated auditor then:

* executes the audit in a stateless manner
* gathers concrete, verifiable evidence
* compares results across runs
* highlights drift, violations, and increasing change-risk

**Example:** detecting new public entry points not declared in the manifest, or provider SDK imports outside approved gateway paths.

### What this is not

* Not autonomous decision-making
* Not policy-generating
* Not code-rewriting
* Not a replacement for human judgement

All authority lives in the repository.
The auditor executes declared rules; it does not invent them.

---

## Core principles

* What cannot be checked mechanically is treated as risk
* Explicit structure beats inferred intent
* Documentation is executable safety infrastructure
* Boundaries must be enforceable, not advisory
* Drift over time is more dangerous than one-off defects
* Audit strictness must be tiered to enable adoption

---

## How it works (at a glance)

1. Humans declare system intent, structure, and constraints in machine-readable artefacts.
2. An automated auditor executes the audit periodically and statelessly.
3. Findings and audit state are written back into the repository.
4. Each run compares against prior results to detect drift and change-risk.
5. Humans review outputs and decide on remediation.

The repository is the memory.
The auditor is the executor.
Humans remain the authority.

---

## Tiered enforcement

Audit enforcement is intentionally **tiered**.

The effective tier determines which checks may enforce and which are only observed or skipped. Higher tiers enable stricter guarantees as system criticality increases.

Checks above the declared tier are explicitly **skipped**, not silently passed.

---

## Status and feedback

This repository contains **Version 1** of the Platform Audit Canvas.

It is published to invite critique, experimentation, and iteration.

Feedback is welcome—please open an issue or discussion so intent, trade-offs, and edge cases can be reviewed explicitly.

---

## Licence

Licensed under the **Creative Commons Attribution 4.0 International Licence (CC BY 4.0)**.
