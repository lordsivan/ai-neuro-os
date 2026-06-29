# Pathways — MCP Capability Map (L1)

The capabilities Pathways orchestrates. All are **external MCP servers** — Pathways runs no
matching engine and hosts no knowledge base. Only **L2** calls these. (Its other inputs —
Connectome and Recall — are **sibling components**, not L1; see `03`.)

## Capability catalog

| MCP server | Provides | Inputs | Outputs | Feeds |
|---|---|---|---|---|
| **Guideline KB** | treatment guideline definitions | diagnosis / stage / patient factors | recommended regimens + conditions + evidence grade | treatment options (L4) |
| **Clinical-trial registry / matching** | trials + eligibility evaluation | patient profile + diagnosis | matching trials + eligibility verdicts | trial matches (L4) |
| **Response-criteria KB** | response/progression criteria (RANO, McDonald, …) | scheme id | criterion sets (items, thresholds) | response assessment (L5) |
| **Terminology / ontology** | code & relate clinical concepts | term / code | SNOMED CT / ICD-11 / regimen codes | grounding & coding |

> **Response-criteria KB is the same KB Reasoner uses** (`docs/components/reasoner/02-mcp-capability-map.md`)
> — RANO/McDonald live once; Reasoner applies them for diagnosis, Pathways for monitoring.
> Terminology is shared across the stack.

## Per-capability notes

### Guideline KB — the rulebook for treatment
Serves machine-readable treatment guidelines: given a coded diagnosis (+ stage/grade,
patient factors, molecular markers), return recommended regimens with their conditions and
evidence grade. Pathways matches the case against these to generate grounded
`TreatmentOption`s — it does not invent therapy.

### Clinical-trial registry / matching — eligibility
Holds open trials and evaluates **eligibility** for a patient profile (diagnosis, prior
treatment, biomarkers, performance status). Returns matching trials with per-criterion
eligibility verdicts, so Pathways can surface a trial as an option **with** why the patient
qualifies (or doesn't).

### Response-criteria KB — the rulebook for monitoring
Serves response/progression criteria (RANO for neuro-onc, McDonald for MS activity, etc.)
as explicit checkable items with thresholds (e.g. RANO measurable-disease change %). L5
applies these **deterministically** over the baseline↔follow-up comparison to produce an
auditable `ProgressionAssessment`.

### Terminology / ontology — grounding
Normalizes diagnoses, regimens, and trial criteria to shared codes so plan elements align
with the `Diagnosis` they treat and persist cleanly as Connectome's `TreatmentPlan`
(→ FHIR CarePlan/Procedure/MedicationRequest, `docs/components/connectome/04-domain-model.md`).

## Contract stability

Higher layers code against `TreatmentOption`/`TrialMatch`/`ResponseAssessment` (`04`),
never these payloads. Swap a guideline source or trial registry → only its L2 adapter
changes; the deterministic response logic (L5) is KB-driven and unaffected. (Guideline/
criteria currency and trial-data governance are **Sentinel** (C8); event scheduling and the
monitoring trigger are wired by **Conductor** (C6). Pathways just calls.)
