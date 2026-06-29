# Perception — Overview (C2)

**Perception** is the **sensory** component of `ai-neuro-os`. It turns raw inputs —
**images** and existing **report text** — into structured, characterized `Finding`s that
**Connectome** (C1) stores and correlates. It is the producer that fills the graph with
observations.

Like every component it follows the stack rules: **no low-level code** (all models run in
**MCP servers**), **layered** (each layer depends only on the one below), **domain +
aggregate code only**. Perception is almost entirely *aggregate/orchestration* logic —
calling vision/NLP MCP servers and fusing their outputs into clean findings.

## Why this exists

A finding rarely comes from one model on one image. The same MRI lesion is implied by a
FLAIR segmentation, a post-contrast enhancement detector, a diffusion-restriction
classifier, *and* the radiologist's report sentence. Naively, that's four disconnected
outputs in four formats. Perception's job is to **fuse** them into **one** characterized
`Finding` — with measurements, coded attributes, and a calibrated confidence — that
Connectome can treat as a single observation and correlate across modalities.

## Sources: pixels **and** reports

Perception draws findings from two streams and fuses them:

- **Pixels** — segmentation / detection / characterization / measurement MCP servers run
  on the images.
- **Reports** — a report-NLP MCP server extracts findings already stated in radiology /
  pathology narratives (and links them to the image regions when possible).

Report-derived findings carry `asserted_by = human` (the reporting clinician's words),
image-derived ones `asserted_by = algorithm`; fusion reconciles the two.

## Boundary with Connectome (the write rule)

> **Perception produces; Connectome persists.**

Perception does **detection → fusion → characterization** and hands **structured
`Finding`s** to Connectome's **L2 `FindingAdapter`** (`docs/components/connectome/03-normalization-adapters.md`),
which is the single writer to the graph. Perception never writes graph nodes itself. This
keeps one writer, one place to govern provenance, and a clean seam between "seeing" (C2)
and "remembering" (C1).

```
images + reports ──▶  PERCEPTION (detect→fuse→characterize)  ──▶  Finding(s)
                                                                     │
                                                  Connectome L2 FindingAdapter ──▶ graph
```

## What it is (and is not)

| Perception **is** | Perception **is not** |
|---|---|
| Orchestration of imaging-AI + report-NLP | The models themselves (those are MCP servers) |
| Fusion of many outputs into one Finding | A graph store (that's Connectome) |
| Characterization + confidence calibration | A diagnostic reasoner (that's C4 Reasoner) |
| A producer of Findings | A persister of Findings |

## Scope (this phase)

- **Modalities:** MRI, CT, ultrasound, histology (same four as Connectome).
- **Inputs:** images (via imaging-archive refs) + radiology/pathology report text.
- **Output:** characterized `Finding`s (Connectome's schema) with measurements, coded
  attributes, provenance + calibrated confidence; low-confidence ones flagged
  **candidate** for human/Connectome confirmation.
- **Deliverable:** design specification + a worked sample. **No code.**

## Design principles

1. **No low-level code** — every model runs in an MCP server; Perception only orchestrates.
2. **Strict layering** — L1→L6, each depends only on the one below (see `01`).
3. **Fuse, don't flood** — many model/report outputs collapse into one Finding per real
   observation; never emit raw detections to Connectome.
4. **Calibrated confidence + provenance on everything** — image vs. report, which model,
   QC status; ensembles and disagreements are represented, not hidden.
5. **Reuse Connectome's domain model** — Perception's output *is* the Connectome `Finding`
   shape; it invents no parallel schema.

## Requirements traceability

| Requirement | Where |
|---|---|
| Produce Findings from images | `02` (vision servers), `04`–`06` |
| Produce Findings from reports | `02` (report-NLP), `05` fusion |
| Fuse multi-model / multi-sequence outputs | `05-fusion.md` |
| Characterize + measure + score | `06-characterization-and-confidence.md` |
| Per-modality model suites | `08-modality-perception.md` |
| Hand off to Connectome (produce-not-persist) | `07-production-and-handoff.md` |
| Provenance / confidence / candidate vs confirmed | `04` shared attrs, `06` |

## Reading order

`01` layers → `02` capabilities → `03` adapters → `04` domain model → `05` fusion →
`06` characterization → `07` production/handoff → `08` modalities → `09` reference. Then
the worked example in `samples/perception/`.
