# Perception — Reference (diagrams + attributes)

The visual model and attribute-level reference. Pairs with `04` (entities) and `05`–`06`
(fusion/characterization).

## Pipeline (end to end)

```mermaid
flowchart TD
    subgraph L1[L1 · MCP servers]
      SEG[Segmentation]:::s
      DET[Detection]:::s
      CHAR[Characterization]:::s
      MEAS[Measurement]:::s
      NLP[Report-NLP]:::s
      QC[QC]:::s
      REG[Registration]:::s
    end
    L1 --> ADP[L2 · Adapters → Detection/Measurement]
    ADP --> FUSE[L4 · Fusion → CandidateFinding]
    FUSE --> CHARZ[L5 · Characterize + QC + calibrate → Finding]
    CHARZ --> PROD[L6 · Production]
    PROD -->|handoff| FA[[Connectome FindingAdapter]]
    classDef s fill:#eee,stroke:#999;
    style FA fill:#ffe9b3,stroke:#b8860b
```

## Detection → Finding (data shapes)

```mermaid
classDiagram
    class Detection {
      id
      source  // image | report
      kind
      attributes
      score
      region  // seriesId+mask|box  OR reportSpan
      negated
      laterality
      modelRunId
    }
    class Measurement { id type value unit region }
    class CandidateFinding { id memberDetections region locatedHint prelimConfidence flags }
    class Finding {
      id
      modality
      kind
      attributes
      measurements
      locationId
      confidence  // calibrated
      status      // confirmed | candidate
      provenance
      evidence
    }
    Detection --> CandidateFinding : fused_into
    Measurement --> CandidateFinding : quantifies
    CandidateFinding --> Finding : characterized_into
```

## Attribute reference

### Detection
| Attribute | Type | Notes |
|---|---|---|
| `source` | enum | `image` \| `report` |
| `kind` | code | SNOMED/RadLex |
| `attributes` | map | optional characterization scores |
| `score` | float | raw model score (pre-calibration) |
| `region` | obj | `{seriesId, mask\|box}` or `{reportSpan}` |
| `negated` | bool | report negation ("no enhancement") |
| `laterality` | enum | left/right/midline |
| `modelRunId` | ref | provenance to the `ModelRun` |

### Finding (produced — Connectome schema)
| Attribute | Type | Notes |
|---|---|---|
| `modality` | enum | MRI \| CT \| Ultrasound \| Histology |
| `kind` | code | finding type |
| `attributes` | map | coded characteristics (final, reconciled) |
| `measurements` | map | volume/diameter/HU/ADC/Ki-67… |
| `locationId` | ref | AnatomicalLocation (from registration) |
| `confidence` | float | **calibrated** 0–1 |
| `status` | enum | `confirmed` \| `candidate` |
| `evidence` | obj | detections/measurements/agreement/conflicts |

### Shared provenance (all entities)
`id`, `source`, `asserted_by` (`algorithm` \| `human`), `model`/`modelVersion`, `method`,
`confidence`, `timestamp`.

## Enumerations

| Enum | Values |
|---|---|
| `Detection.source` | `image`, `report` |
| `asserted_by` | `algorithm` (model), `human` (report) |
| `Finding.status` | `confirmed`, `candidate` |
| `QC status` | `pass`, `degraded`, `fail` |

## Worked instance

A concrete run — the MRI study from the Connectome case producing `find-mri-001` by fusing
FLAIR/T1c/DWI detections with the report sentence — is in `samples/perception/`.
