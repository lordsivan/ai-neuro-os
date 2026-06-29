# Console — Reference (diagrams + attributes)

The visual model and attribute-level reference. Pairs with `04` (entities) and `05`–`07`
(composition / explanation & trust / interaction API).

## End to end

```mermaid
flowchart TD
    UTT[Clinician utterance / action]:::in --> L6[L6 · Interaction API\nask · navigate · confirmCandidate · draftReport]
    L6 --> L5[L5 · Explanation & Trust\nmark · trust profile · explain · report]
    L5 --> L4[L4 · View Composition\nassemble around the lesion]
    L4 --> L3[L3 · Domain Model\nSession · Query · ViewModel · …]
    L3 --> L2[L2 · Adapters\nintent → requests · results → view models]
    L2 --> L1[(L1 · MCP — LLM/NLP, rendering, notification)]:::mcp
    L2 -. reads/calls via Conductor .-> SIB[[Connectome · Reasoner · Pathways · Recall]]:::ext
    L6 -. confirmation routes to .-> OWN[[owner / Connectome · Sentinel-governed]]:::ext
    classDef in fill:#fff,stroke:#333;
    classDef mcp fill:#eee,stroke:#999;
    classDef ext fill:#ffe9b3,stroke:#b8860b;
```

## Data shapes

```mermaid
classDiagram
    class Session { id actor subject trustProfile }
    class Query { id utterance intent subject requests }
    class ViewModel { id kind panels items trustProfile sources }
    class Explanation { id target text evidenceRefs confidence }
    class ReportDraft { id subject sections status }
    class ConfirmationAction { id target targetType decision owner }
    class TrustProfile { name statusFilter assertedBy includeDiscovery }

    Session --> Query : issues
    Session --> TrustProfile : applies
    Query --> ViewModel : produces
    ViewModel --> Explanation : explained_by
    ViewModel --> ReportDraft : drafted_into
    ViewModel --> ConfirmationAction : confirmed_via
```

## Attribute reference

### Query / ViewModel
| Attribute | Type | Notes |
|---|---|---|
| `Query.intent` | enum | `navigate` \| `explain` \| `discover` \| `confirm` \| `report` |
| `Query.trustProfile` | enum | `strict` \| `exploratory` |
| `ViewModel.kind` | enum | `lesion-dashboard` \| `timeline` \| `differential` \| `discovery` \| `report-preview` |
| `ViewModel.items[].status` | enum | `confirmed` \| `candidate` (borrowed from source) |
| `ViewModel.sources` | ref[] | the sibling results composed |

### Explanation / ReportDraft
| Attribute | Type | Notes |
|---|---|---|
| `Explanation.evidenceRefs` | ref[] | findings / criteria / precedent / edges cited |
| `Explanation.text` | string | LLM-generated, constrained to evidence |
| `ReportDraft.sections[]` | obj[] | each: text + `evidenceRefs[]` |
| `ReportDraft.status` | enum | `draft` (never auto-finalized) |

### ConfirmationAction
| Attribute | Type | Notes |
|---|---|---|
| `targetType` | enum | `diagnosis` \| `treatmentPlan` \| `correlation` \| `discovery` |
| `decision` | enum | `confirm` \| `reject` |
| `owner` | enum | component the promotion routes to (Reasoner / Pathways / Connectome / Recall) |

### Shared provenance
`id`, `actor` (audit), `timestamp`; on generated content `asserted_by` (`algorithm`),
`method`, `evidenceRefs[]`. Presented items keep their **source** `status` + provenance;
Console displays and **requests** promotion, it never writes.

## Enumerations

| Enum | Values |
|---|---|
| `Query.intent` | `navigate`, `explain`, `discover`, `confirm`, `report` |
| `TrustProfile.name` | `strict`, `exploratory` |
| item `status` | `confirmed`, `candidate`, `discovered`, `superseded` |
| `ConfirmationAction.targetType` | `diagnosis`, `treatmentPlan`, `correlation`, `discovery` |
| `ConfirmationAction.decision` | `confirm`, `reject` |
| `ReportDraft.status` | `draft` |

## Worked instance

A concrete run — a clinician asks *"what's going on with `pat-001`'s left frontal lesion?"*,
Console composes the lesion dashboard (cross-modal findings + candidate `dx-001` differential
+ recommended biopsy + plan `tx-001` + `prog-001`→`prog-002` timeline) with confirmed vs
candidate marked, the clinician **confirms** `dx-001` and `tx-001`, and the promotions route
back — is in `samples/console/`, matching the Connectome / Reasoner / Pathways case.
