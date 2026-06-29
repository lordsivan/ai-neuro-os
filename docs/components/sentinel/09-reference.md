# Sentinel — Reference (diagrams + attributes)

The visual model and attribute-level reference. Pairs with `04` (entities), `06`
(enforcement) and `08` (audit / provenance).

## Pipeline (end to end)

```mermaid
flowchart TD
    subgraph callers[Sibling components]
      C3[[Recall]]:::ext
      C7[[Console]]:::ext
      C6[[Conductor]]:::ext
      CM[[Perception / Reasoner / Pathways]]:::ext
    end
    callers --> L6[L6 · Governance API / hooks]
    L6 --> L5[L5 · Enforcement & evaluation\nallow/deny · promotion gate · provenance · eval/drift · de-id]
    L5 --> L4[L4 · Governance context\nprincipal · consent · scope · model versions]
    L4 --> L2[L2 · Adapters · normalize]
    L2 --> L1[(L1 · MCP — policy, audit, consent/PHI, identity, eval/drift)]
    L5 -->|append| AUD[(audit trail · hash-chained)]
    classDef ext fill:#ffe9b3,stroke:#b8860b;
    style L1 fill:#eee,stroke:#999
```

## Data shapes

```mermaid
classDiagram
    class Policy { id appliesTo obligations version }
    class AccessDecision {
      id
      principal
      action
      scope
      effect      // allow | deny
      obligations // e.g. deident
      policyRefs
      reason
    }
    class ConsentRecord { id patient status purpose scope restrictions }
    class ScopeGrant { id principal scope partitions obligations expiresAt }
    class ProvenanceRecord {
      id
      target
      asserted_by  // human | algorithm | embedding
      method
      confidence
      completeness // complete | incomplete
    }
    class ModelGovernanceRecord { id model version evalStatus driftStatus approvalState }
    class AuditEvent { id actor action target scope decision refs seq prevHash hash }

    Policy --> AccessDecision : evaluates_into
    ConsentRecord --> ScopeGrant : authorizes
    ScopeGrant --> AccessDecision : bounds
    ProvenanceRecord --> AccessDecision : gates_promotion
    ModelGovernanceRecord --> AccessDecision : gates_model_use
    AccessDecision --> AuditEvent : recorded_as
```

## Attribute reference

### AccessDecision
| Attribute | Type | Notes |
|---|---|---|
| `principal` | ref | the `who` (subject + roles) |
| `action` | string | e.g. `discoverSimilarCases`, `promote` |
| `scope` | enum/ref | `patient:<id>` \| `cohort` \| node/edge |
| `effect` | enum | `allow` \| `deny` |
| `obligations` | string[] | e.g. `deident`, `min-necessary` |
| `policyRefs` | ref[] | policies that drove the verdict |
| `reason` | text | human-readable basis |

### ConsentRecord
| Attribute | Type | Notes |
|---|---|---|
| `patient` | ref | subject of consent |
| `status` | enum | `active` \| `withdrawn` \| `expired` |
| `purpose` | string | purpose limitation |
| `scope` | enum | `patient:<id>` \| `cohort` |
| `restrictions` | string[] | extra constraints |

### ScopeGrant
| Attribute | Type | Notes |
|---|---|---|
| `principal` | ref | who is granted the scope |
| `scope` | enum | `patient:<id>` \| `cohort` |
| `partitions` | string[] | e.g. de-identified cohort partitions |
| `obligations` | string[] | e.g. `deident` |
| `expiresAt` | timestamp | grant validity |

### ProvenanceRecord
| Attribute | Type | Notes |
|---|---|---|
| `target` | ref | node/edge the provenance is for |
| `asserted_by` | enum | `human` \| `algorithm` \| `embedding` |
| `method` | string | e.g. `manual read`, `seg-v2`, `hybrid: rules+llm` |
| `confidence` | float | 0–1 |
| `completeness` | enum | `complete` \| `incomplete` |

### ModelGovernanceRecord
| Attribute | Type | Notes |
|---|---|---|
| `model` / `version` | string | the governed model version |
| `evalStatus` | enum | `pass` \| `fail` \| `pending` |
| `driftStatus` | enum | `stable` \| `drifting` \| `flagged` |
| `approvalState` | enum | `approved` \| `deprecated` \| `blocked` |

### AuditEvent
| Attribute | Type | Notes |
|---|---|---|
| `actor` | ref | principal |
| `action` / `target` / `scope` | — | what / on what / where |
| `decision` | enum | `allow` \| `deny` \| `permit` \| `block` |
| `refs` | ref[] | consent / policy / provenance / model |
| `seq`, `prevHash`, `hash` | — | append-only chain (`08`) |

## Enumerations

| Enum | Values |
|---|---|
| `AccessDecision.effect` | `allow`, `deny` |
| `scope` | `patient:<id>`, `cohort` |
| `ConsentRecord.status` | `active`, `withdrawn`, `expired` |
| `ProvenanceRecord.completeness` | `complete`, `incomplete` |
| `ModelGovernanceRecord.evalStatus` | `pass`, `fail`, `pending` |
| `ModelGovernanceRecord.driftStatus` | `stable`, `drifting`, `flagged` |
| `ModelGovernanceRecord.approvalState` | `approved`, `deprecated`, `blocked` |
| promotion verdict | `permit`, `block` |

## Worked instance

A concrete run — authorizing + auditing the cohort discovery `discoverSimilarCases(pat-001)`
reaching `pat-417` (consent + de-identification), then governing the candidate→confirmed
promotion of `dx-001` (role + provenance-complete check) — is in `samples/sentinel/`.
