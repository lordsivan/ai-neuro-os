# Recall — Reference (diagrams + attributes)

The visual model and attribute-level reference. Pairs with `04` (entities), `05`–`06`
(index/search), `08` (granularity/spaces).

## Write & read paths

```mermaid
flowchart TD
    subgraph write[Write path — embed on ingest]
      EA[Connectome EmbeddingAdapter] -->|embed sourceRef| API1[L6 embed]
      API1 --> ADP[L2 EmbeddingAdapter → MCP embedding]
      ADP --> IDX[L4 upsert IndexEntry]
      API1 -->|vectorRef| EA
    end
    subgraph read[Read path — discovery]
      CALL[Connectome / Reasoner] -->|discoverSimilar / search| API2[L6 serve]
      API2 --> SRCH[L5 ANN + filter + scope + calibrate]
      SRCH --> IDX
      SRCH -->|candidate Neighbors| CALL
    end
    style EA fill:#ffe9b3,stroke:#b8860b
    style CALL fill:#ffe9b3,stroke:#b8860b
```

## Data shapes

```mermaid
classDiagram
    class Embeddable { id kind sourceRef modality granularity }
    class VectorSpace { id model modelVersion dim metric status }
    class Embedding { id sourceRef granularity vectorRef space }
    class IndexEntry { embeddingId space namespace filters }
    class Query { id anchor space k filters mode }
    class Neighbor { sourceRef granularity distance similarity status scope }

    Embeddable --> Embedding : embedded_as
    Embedding --> VectorSpace : in_space
    Embedding --> IndexEntry : indexed_as
    Query --> Neighbor : returns
```

## Attribute reference

### Embedding
| Attribute | Type | Notes |
|---|---|---|
| `sourceRef` | ref | Connectome Finding/Study/case id, or text handle |
| `granularity` | enum | finding \| study \| case \| text |
| `space` | obj | `{model, modelVersion, modality, dim, metric}` |
| `vectorRef` | ref | pointer into the store — **the value Connectome holds** |
| `asserted_by` | enum | always `embedding` |

### Neighbor (served result)
| Attribute | Type | Notes |
|---|---|---|
| `sourceRef` | ref | resolves to a Connectome node |
| `granularity` | enum | which index it came from |
| `distance` | float | raw ANN distance |
| `similarity` | float | **calibrated** 0–1 |
| `space` | obj | the space searched |
| `scope` | enum | `patient:<id>` \| `cohort` (authority used) |
| `status` | enum | always `candidate` |

## Enumerations

| Enum | Values |
|---|---|
| `Embeddable.kind` / `granularity` | `finding`, `study`, `case`, `text` |
| `Query.mode` | `vector`, `lexical`, `hybrid` |
| `scope` | `patient:<id>`, `cohort` |
| `Neighbor.status` | `candidate` (always) |
| `VectorSpace.status` | `active`, `deprecated` |

## Worked instance

A concrete query — `discoverSimilar(find-mri-001)` returning cohort candidates, plus a
`discoverSimilarCases` for the patient — is in `samples/recall/`, tying back to the
Connectome discovery example.
