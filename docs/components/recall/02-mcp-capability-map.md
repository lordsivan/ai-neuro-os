# Recall — MCP Capability Map (L1)

The capabilities Recall orchestrates. All are **external MCP servers** — Recall runs no
models and no ANN engine. Only **L2 adapters** call these. Recall *fronts* these servers:
other components never call them directly, they call Recall.

## Capability catalog

| MCP server | Provides | Inputs | Outputs | Feeds |
|---|---|---|---|---|
| **Embedding (multimodal)** | turn content into a vector | image ref or text + target space | embedding vector (+ model/version, dim) | `Embedding` (L3) |
| **Vector-search / ANN** | nearest-neighbour search | query vector + space + filters + k | ranked ids + distances | `Neighbor` results (L5) |
| **Retrieval / lexical** | keyword/BM25 search | query text + filters | ranked ids + scores | hybrid search (L5) |
| **Vector store** | persist & fetch vectors | upsert/get by id | ack / vectors | index storage (L4) |

> The embedding & vector-search servers are **shared** with Connectome's L1 catalog
> (`docs/components/connectome/02-mcp-capability-map.md`) — but in the reconciled design,
> Connectome reaches them **through Recall**, not directly. Recall is the front door.

## Per-capability notes

### Embedding (multimodal) — the encoder
Produces vectors for both **images** (per-modality region embeddings: MRI/CT/US/histology)
and **text** (reports, history). A given model+version+target defines a **VectorSpace**
(`08`); the adapter tags every vector with it so spaces never get mixed at search time.

### Vector-search / ANN — the recall engine
Approximate nearest-neighbour over a space, with metadata filters (modality, patient
scope, time window, granularity). Returns ids + distances that L5 turns into calibrated,
scoped `Neighbor`s.

### Retrieval / lexical — the hybrid half
BM25/keyword search over report & history text, combined with vector search for **hybrid**
free-text journeys (journey H) so exact terms and semantics both count.

### Vector store — the shelf
Where vectors physically live (addressed by the `vectorRef` Connectome holds). L4 manages
upsert/refresh; the store is opaque to everything above L2.

## Contract stability

Higher layers code against `Embedding`/`Neighbor`/`VectorSpace` (`04`), never these
payloads. Swap the embedding vendor → new `VectorSpace` + re-embed (`08`); swap the ANN
engine → only its L2 adapter changes. (Model lifecycle — approval, eval, drift — is
governed by **Sentinel** (C8); scheduling of bulk re-embeds is **Conductor** (C6). Recall
calls the servers and owns the index.)
