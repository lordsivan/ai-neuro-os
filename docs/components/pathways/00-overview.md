# Pathways (C5) — Overview *(stub — to brainstorm)*

> **Status:** placeholder. Named and scoped; not yet specified as a full L1–L6 component.

## Role

**Pathways** is the **action / planning** component: given a diagnosis, it assembles a
**treatment plan**, matches against **guidelines and clinical trials**, and runs
**longitudinal monitoring** — tracking response/progression over time using the
appropriate scheme (RANO, McDonald, mRS). It writes `TreatmentPlan` and
`ProgressionAssessment` nodes back to Connectome.

## Depends on

- **C4 Reasoner** — the diagnosis it plans around.
- **C1 Connectome** — `Diagnosis`, `Lesion`, the progression timeline
  (`docs/components/connectome/04-domain-model.md`).
- MCP servers (L1): **FHIR** (CarePlan/Procedure/MedicationRequest), **guideline / trial**
  matching.

## Key MCP servers

| Server | Use |
|---|---|
| FHIR | persist CarePlan / Procedure / MedicationRequest |
| Guideline / trial registry | match eligible guidelines & trials |

## Open questions (for the brainstorm)

- Monitoring cadence & triggers (when to re-assess; what counts as progression).
- Trial-matching eligibility logic vs. external matching MCP server.
- Relationship to Reasoner — does response assessment feed back into re-diagnosis?

When promoted, this folder grows the same `01`–`09` layer docs as Connectome.
