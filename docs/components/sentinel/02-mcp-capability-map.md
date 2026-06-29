# Sentinel — MCP Capability Map (L1)

The capabilities Sentinel orchestrates. All are **external MCP servers** — Sentinel runs no
policy logic of its own, hosts no audit database, and stores no consent records itself. It
**calls** these. Only **L2** reaches them (`03`); above L2 everything works on normalized
domain objects.

## Capability catalog

| MCP server | Provides | Inputs | Outputs | Feeds |
|---|---|---|---|---|
| **Policy engine** | evaluate access/scope rules | principal, action, target scope, attributes | allow / deny + obligations (e.g. de-identify) | `AccessDecision` (L5) |
| **Audit-log store** | append-only, immutable event log | `AuditEvent` | write ack + sequence/hash | audit trail (L6/`08`) |
| **Consent / PHI registry** | resolve consent & PHI boundaries | patient, action, scope | `ConsentRecord` (status, purpose, restrictions) | consent check (L5) |
| **Identity / access provider** | authenticate & resolve principal | identity token | principal (subject, roles, org) | principal (L4) |
| **Model-eval / drift monitor** | eval scores & drift signals per model version | model id, version, eval suite | eval results, drift status, approval state | `ModelGovernanceRecord` (L5) |

> The **identity** and **terminology**-adjacent grounding may be **shared** with other
> components' L1 catalogs — same server, reused. The policy engine, consent registry and
> eval/drift monitor are Sentinel's distinctive capabilities.

## Per-capability notes

### Policy engine — the rulebook for access
Given a principal, an action and a target scope, the policy engine returns an allow/deny
verdict **plus obligations** — e.g. "allow, but de-identify" for cohort retrieval, or "allow
only if role ∈ {attending, MDT}" for a promotion. Sentinel does not encode these rules in
code; they live in the engine and are evaluated there. L5 consumes the verdict; it does not
re-implement the logic.

### Audit-log store — the immutable trail
Serves an **append-only** log. Every decision Sentinel makes and every event a component
reports via `record` is written here, hash-chained for tamper-evidence (`08`). The store is
write-once: Sentinel never updates or deletes an entry, only appends.

### Consent / PHI registry — the privacy boundary
Resolves, for a patient + action + scope, the applicable **`ConsentRecord`**: is there a
lawful basis / consent for this use, with what purpose limitation and restrictions. This is
what gates **Recall cohort discovery** — cross-patient memory is only served on
de-identified partitions for which consent/lawful-basis is recorded
(`docs/components/recall/08-granularity-and-spaces.md`).

### Identity / access provider — who is asking
Authenticates the incoming identity token and resolves the **principal**: subject, roles
(clinician, attending, MDT member, service account), organization. This `who` is the first
input to every governance context (`05`).

### Model-eval / drift monitor — model governance
Holds eval results and drift signals per **model version**. When Perception registers a new
segmentation model, Recall an embedding model, or Reasoner/Pathways an LLM version, Sentinel
records a **`ModelGovernanceRecord`** and checks the version is within eval thresholds and
not flagged for drift. The components **call** the models; Sentinel **governs** which
versions are approved to be called.

## Contract stability

Higher layers code against `AccessDecision` / `ConsentRecord` / `ModelGovernanceRecord` /
`AuditEvent` (`04`), never these payloads. Swap the policy-engine vendor → only its L2
adapter changes; the enforcement logic (L5) is unaffected because it works on the normalized
`AccessDecision`, not the engine's native verdict shape.

> Model governance — approval, eval, drift, safety — lives here in Sentinel (C8). Scheduling
> the eval/backfill runs is **Conductor** (C6). The model-calling components just *call*;
> they register their versions and honor the approval state Sentinel returns.
