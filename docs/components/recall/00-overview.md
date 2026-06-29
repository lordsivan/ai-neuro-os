# Recall — Overview (C3)

**Recall** is the **associative memory** of `ai-neuro-os`. It owns the **embedding +
vector-index lifecycle** and serves **similarity / discovery** queries: "what looks or
reads like this?" — across modalities and across the whole patient cohort. It is the
substrate behind Connectome's discovery role and Reasoner's precedent reasoning.

Like every component it follows the stack rules: **no low-level code** (embedding &
ANN run in **MCP servers**), **layered** (each layer depends only on the one below),
**domain + aggregate code only**. Recall is the service that turns those raw model
capabilities into a governed, queryable memory.

## Why this exists

Connectome navigates **known** edges. But the highest-value clinical questions are often
about the **unknown**: "have we seen a lesion like this before?", "which prior patients
followed a similar course?", "find reports mentioning tumefactive demyelination". Those
need **semantic similarity**, not graph traversal. Recall provides it — and does so once,
as a shared service, so every component gets the same calibrated, privacy-governed
similarity instead of each rolling its own.

## The boundary (Recall fronts the vector stack)

> **Recall is the single vector-memory service.** It calls the embedding & vector-search
> MCP servers, owns the index, and exposes search. Other components call **Recall** — not
> the raw MCP servers.

- **Connectome** stores only a `vectorRef` on its `Finding`/`Study` nodes; its
  `EmbeddingAdapter` **delegates to Recall** to embed-on-ingest, and its discovery
  (`docs/components/connectome/06-cross-modal-correlation.md`, mechanism 5) and discovery
  journeys (F/G/H, `07`) **call Recall's API**.
- **Reasoner** (C4) calls Recall for similar prior cases (precedent).
- The actual vectors live in **Recall's index**, never in the graph.

```
embed-on-ingest:   Connectome EmbeddingAdapter ──▶ Recall.embed()  ──▶ vectorRef back to graph
discovery:         Connectome / Reasoner       ──▶ Recall.search() ──▶ candidate neighbors
                                                        │
                                          embedding & vector-search MCP servers (L1)
```

## What it embeds (multi-granularity)

Recall indexes at several granularities so it can answer different questions:

| Granularity | Embeds | Powers |
|---|---|---|
| **Finding** | image region (per modality) | finding-to-finding similarity (journey F) |
| **Study** | a study's salient content | study-level lookup |
| **Case** | aggregate of a patient's findings + history | similar-cases / cohort (journey G) |
| **Text** | report & patient-history narrative | free-text / semantic search (journey H) |

## What it is (and is not)

| Recall **is** | Recall **is not** |
|---|---|
| The vector-memory service (embed + index + search) | The embedding/ANN models (those are MCP servers) |
| Multi-granular similarity & cohort retrieval | The knowledge graph (that's Connectome) |
| A producer of **candidate** matches | A decider of truth (correlation = Connectome; diagnosis = Reasoner) |
| Calibrated, privacy-scoped similarity | A persister of clinical facts |

## Scope (this phase)

- **Inputs:** Connectome `Finding`/`Study`/case + report/history text.
- **Output:** ranked **candidate** neighbors / cases / text hits, each
  `asserted_by = embedding` with a calibrated similarity — never a confirmed link.
- **Cohort/cross-patient** search is privacy-scoped, governed by **Sentinel** (C8).
- **Deliverable:** design specification + a worked sample. **No code.**

## Design principles

1. **No low-level code** — embedding & ANN run in MCP servers; Recall orchestrates.
2. **Strict layering** — L1→L6, each depends only on the one below (`01`).
3. **One vector-memory** — every component shares Recall's index & calibration; no
   parallel embedding pipelines.
4. **Always candidate** — similarity output is a proposal (`asserted_by = embedding`),
   never silently promoted to a confirmed graph edge.
5. **Versioned vector spaces** — a model/version defines a space; searches never mix
   spaces (`08`).
6. **Privacy by scope** — cross-patient retrieval is gated by Sentinel-governed scope.

## Requirements traceability

| Requirement | Where |
|---|---|
| Embed findings/studies/cases/text | `04` domain model, `08` granularity |
| Own embedding + index lifecycle | `05-index.md` |
| Similarity & free-text search | `06-retrieval-and-search.md` |
| Serve Connectome/Reasoner discovery | `07-discovery-api.md` |
| Always-candidate, calibrated, provenance | `06`, `09` |
| Versioned spaces; model changes | `08-granularity-and-spaces.md` |
| Cohort privacy | `08` (Sentinel-governed) |

## Reading order

`01` layers → `02` capabilities → `03` adapters → `04` domain model → `05` index →
`06` retrieval → `07` discovery API → `08` granularity & spaces → `09` reference. Then
the worked example in `samples/recall/`.
