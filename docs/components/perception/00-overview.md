# Perception (C2) — Overview *(stub — to brainstorm)*

> **Status:** placeholder. Named and scoped; not yet specified as a full L1–L6 component.

## Role

**Perception** is the **sensory** component: it orchestrates imaging-AI MCP servers to
turn raw images into structured `Finding`s that **Connectome** (C1) can store and
correlate. It calls segmentation/detection/characterization/measurement capabilities and
shapes their output into the domain `Finding` shape — pure aggregate/orchestration code,
no model inference of its own.

## Depends on

- **C1 Connectome** — for the `Finding` / `Series` / `AnatomicalLocation` schema it
  produces against (`docs/components/connectome/04-domain-model.md`).
- MCP servers (L1): **segmentation / detection**, **registration** (for localization),
  **terminology** (for coding findings).

## Key MCP servers

| Server | Use |
|---|---|
| Segmentation / detection | lesion masks, measurements, characterization |
| Registration | localize findings to an atlas / common frame |
| Terminology | code finding kind & attributes |

## Open questions (for the brainstorm)

- Boundary with Connectome's L2 `FindingAdapter` — does Perception *own* finding
  production end-to-end, or hand raw results to Connectome's adapter?
- How are multi-model ensembles / disagreements represented (confidence, multiple
  candidate findings)?
- Sync (on-ingest) vs. async (batch re-processing) production of findings.

When promoted, this folder grows the same `01`–`09` layer docs as Connectome.
