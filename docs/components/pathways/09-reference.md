# Pathways — Reference (diagrams + attributes)

The visual model and attribute-level reference. Pairs with `04` (entities) and `05`–`06`
(synthesis / selection & response).

## Both tracks (end to end)

```mermaid
flowchart TD
    subgraph inputs[L2 inputs]
      CONN[[Connectome]]:::ext
      REC[[Recall precedent]]:::ext
      GL[Guideline KB]:::s
      TRL[Trial registry]:::s
      RC[Response-criteria KB]:::s
    end
    inputs --> L2[L2 · ManagementContext]
    L2 --> L4[L4 · Synthesis\nplanning: options+assemble · monitoring: comparison set]
    L4 --> L5[L5 · Selection & Response\nplanning: rank+eligibility · monitoring: apply criteria]
    L5 --> L6[L6 · Proposal & Monitoring]
    L6 -->|candidate TreatmentPlan| CA[[Connectome]]
    L6 -->|ProgressionAssessment| CA
    CA --> MDT[MDT confirms · Console]
    classDef s fill:#eee,stroke:#999;
    classDef ext fill:#ffe9b3,stroke:#b8860b;
```

## Data shapes

```mermaid
classDiagram
    class ManagementContext { diagnosis patientFactors guidelines trials precedent comparison }
    class TreatmentOption { id kind detail supporting confidence }
    class TrialMatch { id eligibility criteriaResults }
    class TreatmentPlan { intent components schedule status }
    class ResponseAssessment { id scheme verdict deltaMeasurements comparedStudies }
    class ProgressionAssessment { timepoint scheme status basis }

    ManagementContext --> TreatmentOption : grounds
    TrialMatch --> TreatmentOption : supports
    TreatmentOption --> TreatmentPlan : assembled_into (candidate)
    ManagementContext --> ResponseAssessment : compares
    ResponseAssessment --> ProgressionAssessment : becomes
```

## Attribute reference

### TreatmentOption / TreatmentPlan
| Attribute | Type | Notes |
|---|---|---|
| option `kind` | enum | surgery \| radiotherapy \| systemic \| trial \| surveillance |
| option `supporting` | ref[] | guideline / trial / precedent refs |
| plan `intent` | enum | curative \| palliative \| diagnostic \| surveillance |
| plan `components` | obj[] | sequenced selected options |
| plan `status` | enum | `candidate` → `active` (on MDT confirm) |
| plan `confidence` | float | calibrated |

### ResponseAssessment / ProgressionAssessment
| Attribute | Type | Notes |
|---|---|---|
| `scheme` | enum | RANO \| McDonald \| mRS |
| `verdict` / `status` | enum | response \| stable \| progression \| recurrence \| pseudo-progression |
| `deltaMeasurements` | map | measured change vs. baseline |
| `comparedStudies` / `basis` | ref[] | studies compared |
| `criteriaResults` | obj[] | per-criterion met/not-met/indeterminate + refs |

### Shared provenance
`id`, `source`, `asserted_by` (`algorithm`), `method`, `confidence`, `timestamp`; plans &
assessments are `status: candidate` until human/MDT confirmation.

## Enumerations

| Enum | Values |
|---|---|
| `TreatmentOption.kind` | `surgery`, `radiotherapy`, `systemic`, `trial`, `surveillance` |
| `TreatmentPlan.status` | `candidate`, `active`, `superseded` |
| response `verdict` | `response`, `stable`, `progression`, `recurrence`, `pseudo-progression` |
| `TrialMatch.eligibility` | `eligible`, `ineligible`, `unknown` |

## Worked instance

A concrete run — planning `tx-001` (resection → chemoradiation → adjuvant) + a trial match
for confirmed `dx-001`, then event-driven `prog-001` (stable) and `prog-002` (progression)
— is in `samples/pathways/`, matching the Connectome case.
