# Pathways — Overview (C5)

**Pathways** is the **action / planning** component of `ai-neuro-os`. It takes a confirmed
`Diagnosis` and answers the next two clinical questions: **"what do we do?"** (treatment
planning) and **"is it working?"** (longitudinal monitoring). It writes the case's
`TreatmentPlan` and `ProgressionAssessment`s back to Connectome — closing the timeline.

Like every component it follows the stack rules: **no low-level code** (guidelines, trial
matching and response criteria run in **MCP servers**), **layered** (each layer depends
only on the one below), **domain + aggregate code only**.

## Why this exists

Reasoner (C4) ends at a diagnosis. But a diagnosis isn't care. Someone must turn it into a
**plan** — grounded in guidelines, aware of eligible **trials**, informed by how similar
patients fared — and then **watch** whether the disease responds, applying formal response
criteria (RANO, McDonald) study after study. Pathways does both, once, as a shared service,
so the plan and every response assessment are captured in the graph with their rationale.

## Two tracks, one scaffold

Pathways runs **two workflows** over the same L1–L6 layers (`01`):

| Track | Trigger | Produces |
|---|---|---|
| **Planning** | a confirmed `Diagnosis` | a candidate `TreatmentPlan` (+ guideline & trial matches) |
| **Monitoring** | a new follow-up study (event-driven) | a `ProgressionAssessment` on the lesion timeline |

## How it works

- **Planning** is **hybrid like Reasoner**: treatment **guidelines** + **trial eligibility**
  as structured matching, plus **precedent** (how similar treated cases did, from Recall),
  assembled into a coherent plan with rationale.
- **Monitoring** is **event-driven**: when a follow-up study is ingested, Pathways
  automatically re-assesses response/progression by applying **response criteria
  deterministically** (RANO/McDonald) over the baseline↔follow-up comparison.

## Boundaries (propose / persist / confirm)

> **Pathways proposes; Connectome persists; a human confirms.**

- A **candidate `TreatmentPlan`** is handed to Connectome flagged `candidate`; a clinician
  or **tumor board (MDT)**, via **Console** (C7), confirms before it is `active`.
- A **`ProgressionAssessment`** is also produced as a proposal for clinician sign-off, then
  persisted on the lesion timeline (`assessed_by` / `progresses_to`).
- Pathways **reads** Connectome (diagnosis, lesion, history, timeline) and **calls** Recall
  (treated-case precedent) at L2 — sibling-component inputs, distinct from its MCP servers.
  It writes only via Connectome's adapter.

```
confirmed Diagnosis ──▶ PATHWAYS (planning)  ──▶ candidate TreatmentPlan ─▶ Connectome ─▶ MDT confirms
new follow-up study ──▶ PATHWAYS (monitoring) ──▶ ProgressionAssessment  ─▶ Connectome timeline
```

## Scope boundary with neighbours

| Pathways **does** | Pathways **does not** |
|---|---|
| treatment planning, trial matching | diagnose (that's Reasoner, C4) |
| apply response criteria over time (monitoring) | apply criteria for diagnosis (Reasoner) |
| produce TreatmentPlan + ProgressionAssessment | persist them (Connectome) or confirm them (a human) |
| recommend management | order/execute care (clinical systems, downstream) |

> Reasoner recommended the *next diagnostic test*; Pathways plans *treatment* and runs
> *monitoring*. RANO/McDonald are shared with Reasoner — same KB, used here for response,
> there for diagnosis.

## Scope (this phase)

- **Inputs:** Connectome (confirmed diagnosis, lesion, history, prior plan, timeline) +
  Recall precedent + guideline/trial/criteria KBs.
- **Output:** candidate `TreatmentPlan` (with guideline & trial matches + rationale) and
  event-driven `ProgressionAssessment`s (RANO/McDonald), all candidate-then-confirmed.
- **Deliverable:** design specification + a worked sample. **No code.**

## Design principles

1. **No low-level code** — guidelines/trials/criteria are MCP servers; Pathways orchestrates.
2. **Strict layering** — L1→L6, each depends only on the one below (`01`).
3. **Grounded plans** — every plan element cites a guideline / precedent / trial.
4. **Deterministic response criteria** — RANO/McDonald applied as auditable checklists.
5. **Propose, human-confirm** — plans and assessments are candidates until signed off.
6. **Event-driven surveillance** — monitoring is automatic on new studies, not on request.

## Requirements traceability

| Requirement | Where |
|---|---|
| Treatment planning | `05-plan-synthesis.md`, `06` |
| Guideline matching | `05`, `08-guidelines-and-trials.md` |
| Clinical-trial matching | `05`, `08` |
| Longitudinal monitoring (event-driven) | `07-proposal-and-monitoring.md` |
| Response assessment (RANO/McDonald) | `06-selection-and-response.md`, `08` |
| Precedent (treated cases) | `03` (calls Recall) |
| Propose candidate, human/MDT-confirmed | `07` |

## Reading order

`01` layers → `02` capabilities → `03` evidence & context → `04` domain model →
`05` plan synthesis → `06` selection & response → `07` proposal & monitoring →
`08` guidelines & trials → `09` reference. Then the worked example in `samples/pathways/`.
