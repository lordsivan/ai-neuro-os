# Recall — Retrieval & Search (L5)

L5 answers similarity queries over the indexes (L4): it runs ANN (and lexical/hybrid)
search, **filters** and **scopes** results, **calibrates** similarity, **dedups**, and
labels everything **candidate**. It is the query brain; L6 just exposes it.

## Query modes

| Mode | Engine | Use |
|---|---|---|
| **vector** | ANN over a space | finding/case similarity (journeys F, G) |
| **lexical** | BM25/keyword | exact-term report search |
| **hybrid** | vector + lexical, fused | free-text semantic search (journey H) |

A query carries: an **anchor** (an existing Embeddable/vector, or query text), the
**space**, `k`, and **filters**.

## Filtering & scoping

Filters are applied as index metadata constraints *before/with* ANN so results are
relevant and **safe**:

- **Modality** — e.g. restrict to MRI, or allow cross-modality (find a CT match for an MRI
  anchor).
- **Granularity** — finding vs. case vs. text index.
- **Time** — within a window, or prior-to a date (for precedent).
- **Scope (privacy)** — `patient` (this patient only) vs. `cohort` (cross-patient).
  **Cross-patient scope is Sentinel-governed** (`08`); L5 enforces the scope the caller is
  authorized for and records it.

## Similarity calibration

Raw ANN distance is **not** returned as-is. L5 maps distance → a **calibrated
`similarity`** (0–1) per vector space, so a "0.9" means the same confidence across spaces
and modalities — important because Connectome thresholds discovery candidates on it (`07`).
Calibration curves are per-space and versioned with the space.

## Cross-modal similarity

For an MRI anchor seeking CT/US/histology matches, L5 queries the appropriate
**cross-modal space** (a shared/aligned space, or per-modality spaces bridged by a learned
mapping — itself an MCP capability). Cross-modal neighbors are *exactly* the candidate
`corresponds_to` proposals Connectome's correlation mechanism 5 consumes
(`docs/components/connectome/06-cross-modal-correlation.md`).

## Dedup & re-ranking

- **Dedup** — collapse multiple vectors of the same `sourceRef` (e.g. several sequences of
  one study) to one neighbor.
- **Re-rank** — optional fusion of vector + lexical scores (hybrid), and down-weighting of
  near-duplicates or same-patient self-matches.

## Output: always candidate

Every `Neighbor` L5 returns is:

- `status: candidate`, `asserted_by: embedding`, with calibrated `similarity`, the
  `space`/`method`, and the `scope` it was retrieved under.
- **Never** a confirmed correlation. Promotion to a confirmed graph edge is the **caller's**
  decision (Connectome correlation, or a human via Console) — not Recall's.

> Recall says "these look similar". Whether that means "same lesion" (Connectome) or
> "relevant precedent" (Reasoner) is decided by the caller, with provenance intact.

## What L5 does *not* do

- It does not **build** indexes (L4) or **embed** (L2/L1).
- It does not **persist** results to the graph (Connectome does, if it promotes them).
- It does not **decide truth** — only ranked, calibrated, scoped candidates.
