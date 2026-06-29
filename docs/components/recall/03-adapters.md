# Recall — Adapters / Normalization (L2)

L2 is the **only** layer that calls L1 MCP servers. It wraps the embedding, ANN, lexical
and store servers behind uniform domain shapes (`Embedding`, `Neighbor`) so everything
above L2 is engine-agnostic.

> L2 **translates**; it does not compute. No vector math, no distance calculation, no
> index building happens here — those are L1 (math) or L4 (index orchestration). L2 maps
> requests and tags provenance + vector space.

## Adapter responsibilities

Each adapter:

1. **Calls** its one MCP server.
2. **Maps** domain requests → server calls and server responses → domain objects.
3. **Tags the VectorSpace** (`model`, `modelVersion`, `modality`, `dim`) on every vector.
4. **Stamps provenance** — `source` (server), `asserted_by = embedding`, `method`,
   `timestamp` — and carries the `sourceRef` (which Finding/Study/case/text it came from).
5. **Normalizes scores** — raw distances pass through unchanged here; *calibration* is L5.

## Adapter catalog

| Adapter | Source (L1) | Emits |
|---|---|---|
| **EmbeddingAdapter** | Embedding | `Embedding{ vectorRef, space, sourceRef }` |
| **AnnAdapter** | Vector-search | `Neighbor[]{ sourceRef, distance, space }` |
| **LexicalAdapter** | Retrieval | `Neighbor[]{ sourceRef, score }` (text) |
| **StoreAdapter** | Vector store | upsert/get acks; `vectorRef`s |

## Mapping examples (design-level)

**Embed a Finding**
```
embedFinding(find-mri-001, space=mri-img-v1)
   ──EmbeddingAdapter──▶ Embedding {
        id, sourceRef: find-mri-001, granularity: finding,
        space: {model: img-embed, version: v1, modality: MRI, dim: 768},
        vectorRef: "vec://mri/find-mri-001",
        asserted_by: embedding, timestamp }
```

**ANN search**
```
annSearch(queryVector, space=mri-img-v1, k=20, filters={scope: cohort})
   ──AnnAdapter──▶ [ Neighbor{ sourceRef: find-mri-cohort-417, distance: 0.06, space },
                     Neighbor{ sourceRef: find-mri-cohort-088, distance: 0.09, space }, … ]
```

## The `vectorRef` contract

Recall returns a **`vectorRef`** (an opaque pointer into its store) for every embedded
object. **Connectome stores the `vectorRef`, not the vector** — keeping vectors out of the
graph and under Recall's lifecycle control (`05`, `08`). Above L2, vectors are only ever
referenced, never inspected.

This adapter boundary is what lets Recall swap embedding models or ANN engines without any
caller — Connectome, Reasoner — noticing: they hold refs and call the API, not the engine.
