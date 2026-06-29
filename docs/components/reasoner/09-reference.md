# Reasoner — Reference (diagrams + attributes)

The visual model and attribute-level reference. Pairs with `04` (entities) and `05`–`06`
(differential / criteria).

## Pipeline (end to end)

```mermaid
flowchart TD
    subgraph inputs[L2 inputs]
      CONN[[Connectome graph]]:::ext
      REC[[Recall precedent]]:::ext
      KB[Criteria KB]:::s
      LLMS[LLM]:::s
    end
    inputs --> L2[L2 · EvidenceBundle]
    L2 --> L4[L4 · Differential generation\nLLM + criteria + precedent → grounded candidates]
    L4 --> L5[L5 · Criteria adjudication + calibration + next-test]
    L5 --> L6[L6 · DiagnosisProposal]
    L6 -->|candidate Diagnosis| CA[[Connectome adapter]]
    CA --> HUMAN[Clinician confirms · Console]
    classDef s fill:#eee,stroke:#999;
    classDef ext fill:#ffe9b3,stroke:#b8860b;
```

## Data shapes

```mermaid
classDiagram
    class EvidenceBundle { findings history precedent candidateCriteriaSets codes }
    class DifferentialItem {
      id
      condition
      rank
      confidence   // calibrated at L5
      supporting   // evidence refs
      refuting
      criteriaResults
      rationale    // grounded LLM
    }
    class Criterion { id statement logic result evidenceRefs }
    class DiscriminatingTest { id test targets expectedInfoGain rationale }
    class DiagnosisProposal { id subject differential top nextTest status evidenceTrail }
    class Diagnosis { condition certainty basis status }

    EvidenceBundle --> DifferentialItem : grounds
    Criterion --> DifferentialItem : adjudicates
    DifferentialItem --> DiagnosisProposal : ranked_into
    DiscriminatingTest --> DiagnosisProposal : recommends
    DiagnosisProposal --> Diagnosis : top_becomes (candidate)
```

## Attribute reference

### DifferentialItem
| Attribute | Type | Notes |
|---|---|---|
| `condition` | code | SNOMED/ICD-11/WHO CNS |
| `rank` | int | position in differential |
| `confidence` | float | **calibrated** 0–1 |
| `supporting` / `refuting` | ref[] | evidence (findings/history/precedent/criteria) |
| `criteriaResults` | obj[] | per-criterion met/not-met/indeterminate + refs |
| `rationale` | text | grounded LLM explanation |

### DiagnosisProposal
| Attribute | Type | Notes |
|---|---|---|
| `subject` | ref | lesion/patient |
| `differential` | DifferentialItem[] | ranked |
| `top` | DifferentialItem | → proposed `Diagnosis` |
| `nextTest` | DiscriminatingTest | what would confirm/refute |
| `status` | enum | always `candidate` |
| `evidenceTrail` | ref[] | full auditable trail |

### Shared provenance
`id`, `source`, `asserted_by` (`algorithm`), `method` (`hybrid: rules+llm`), `confidence`,
`timestamp`.

## Enumerations

| Enum | Values |
|---|---|
| `Criterion.result` | `met`, `not-met`, `indeterminate` |
| `DiagnosisProposal.status` | `candidate` (always) |
| `ReasoningCase.question` | `diagnose`, `explain` |

## Worked instance

A concrete run — reasoning over the left-frontal lesion into a 3-way differential, applying
criteria, recommending biopsy as the next test, and proposing candidate `dx-001` (later
`confirmed_by` histology) — is in `samples/reasoner/`.
