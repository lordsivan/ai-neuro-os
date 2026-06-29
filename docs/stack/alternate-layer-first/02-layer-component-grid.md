# Layer × Component Grid (L1–L7 × C1–C7)

> **Status: alternate-approach analysis.** This grid reconciles the two architectures
> for `ai-neuro-os`: the layer-first model in
> [`01-layered-architecture.md`](01-layered-architecture.md) (the **rows**) and the
> canonical component-first model in [`docs/stack/component-map.md`](../component-map.md)
> (the **columns**). Each cell is what a component does at a given layer.

## The grid

Rows = the seven layer-first layers (L7→L1). Columns = components C1–C7.
**✔** = primary / owns it · **◑** = participates · **—** = absent.

| | **C1 Connectome** | **C2 Perception** | **C3 Recall** | **C4 Reasoner** | **C5 Pathways** | **C6 Conductor** | **C7 Console** |
|---|---|---|---|---|---|---|---|
| **L7 Copilot** | — | — | — | ✔ differential reasoning | ◑ treatment planning | ◑ agent loop | ✔ NL query, report draft |
| **L6 Workflows** | ◑ nav journeys | ◑ per-study pipeline | — | ◑ criteria workflows | ✔ treat + monitor orchestration | ✔ task plan / route | ◑ invokes workflows |
| **L5 Capability APIs** | ✔ nav / discovery API | ◑ Findings handoff | ✔ FindSimilarCases | ◑ differential API | ◑ trial-match / response | ◑ registry exposes caps | ◑ consumes APIs |
| **L4 Discovery** | ✔ correlation, knowledge gen | — | ✔ embeddings, similarity, vector / spatial search | ◑ evidence surfacing (via C3) | ◑ trial similarity | — | — |
| **L3 Processing** | — *(→ MCP)* | ✔ segment / detect / fuse (orch. MCP) | ◑ embedding generation | — | — | — | — |
| **L2 Resolution** | ✔ id / coord / atlas resolve, cross-modal joins | ◑ detection → region map | ◑ granularity / space resolve | — | — | — | — |
| **L1 Data Access** | ✔ ingest / normalize / store graph | ◑ image / report fetch | ✔ vector-index storage | — | — | ◑ MCP registry / health | — |

### Cross-cutting (do not fit a single row)

- **C6 Conductor ≈ the Execution Fabric** — it spans the whole grid (plan / route /
  registry / agent loop); the ◑ marks above are only where it is most visible.
- **C8 Sentinel** (off-grid; the request was C1–C7) — provenance, consent / PHI, access
  control and governance applied to *every cell*.

## How to read it

The fill pattern is the point: **each component is a vertical slice that lights up a
contiguous band of layers, and together the columns tile the whole L1–L7 stack.**

- **Lower-left is dense** — C1 / C2 / C3 (data & sensory) own L1–L4.
- **Upper-right is dense** — C4 / C5 / C7 (cognition & interaction) own L5–L7.
- **The diagonal is the architecture:** data components at the bottom, reasoning / UI at
  the top, with very little overlap. That is a clean, well-factored partition.

Two structural facts the grid exposes:

1. **L3 Processing has exactly one real occupant (C2)** — and even C2 only *orchestrates*
   it; the actual compute is in MCP. This confirms the canonical model does not *own*
   the Processing layer, it brokers it.
2. **Rows with multiple ✔ in different columns are the genuinely shared bands** — L4
   (C1 *and* C3) and L5 (C1 *and* C3). Rows with a single ✔ (L3→C2, L7→C7) are really
   vertical concerns wearing a layer's clothing.

## Why this matters

This grid is where the two models reconcile: **the layer-first spec gave us the rows;
the component-first spec gave us the columns.** Every cell is a place where a real
capability will — or deliberately will not — live.

Practically, the **empty cells are as useful as the filled ones**: they say what you do
*not* have to build. And the multi-✔ rows (L4, L5) flag exactly which horizontal bands
have earned promotion to shared infrastructure under the "≥2 components need it" rule —
without that evidence, a row stays a private cell, not a shared layer.

See [`01-layered-architecture.md`](01-layered-architecture.md) for the full layer-first
spec and [`docs/stack/component-map.md`](../component-map.md) for the canonical
component-first model.
