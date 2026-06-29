# Recall (C3) — Overview *(stub — to brainstorm)*

> **Status:** placeholder. Named and scoped; not yet specified as a full L1–L6 component.

## Role

**Recall** is the **associative memory**: it owns the embedding + vector-index lifecycle
that powers Connectome's **discovery role**. It manages how findings/studies/report-text
are embedded, indexed, refreshed, and queried by approximate-nearest-neighbour search.
**Cohort / similar-case analytics** lives here too.

## Depends on

- **C1 Connectome** — embeddings attach to its `Finding` / `Study` entities; discovery
  results are returned as **candidate** correlations (`docs/components/connectome/06-cross-modal-correlation.md`,
  mechanism 5).
- MCP servers (L1): **embedding & vector-search**, **retrieval / search**.

## Key MCP servers

| Server | Use |
|---|---|
| Embedding & vector-search | multimodal embeddings + ANN search |
| Retrieval / search | hybrid lexical + vector search |

## Open questions (for the brainstorm)

- Index granularity: per-finding, per-study, per-case vectors — or all of them?
- Re-embedding policy when models change (versioned vector spaces).
- Cohort privacy: cross-patient discovery vs. PHI boundaries (coordinate with **Sentinel**).
- Where the vector store lives vs. the graph (graph stores `vectorRef`, not the vector —
  see `docs/components/connectome/03-normalization-adapters.md`).

When promoted, this folder grows the same `01`–`09` layer docs as Connectome.
