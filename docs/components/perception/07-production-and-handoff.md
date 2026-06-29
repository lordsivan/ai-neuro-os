# Perception — Production & Handoff (L6)

L6 is Perception's outward face: it runs the per-study **production pipeline** and hands
finished `Finding`s to **Connectome**. It is the only layer other components talk to, and
it calls only L5 — never a model directly.

## Triggering

Perception runs **per study**, triggered when a study is ingested:

- **Conductor** (C6) or Connectome's Intake (L2) signals "new study available".
- L6 opens a **`DetectionTask`** for the study and drives L1–L5.
- Mode is **asynchronous / batch per study** (not per-frame realtime): a study is
  processed as a unit, and can be **re-processed** later (new model version, new request)
  — re-runs produce updated findings with the same stable IDs.

```mermaid
flowchart LR
    ING[(Study ingested)] -->|signal| L6[L6 Production]
    L6 --> TASK[DetectionTask]
    TASK --> PIPE[L1-L5 detect → fuse → characterize]
    PIPE --> F[Finding(s)]
    F -->|handoff| FA[[Connectome FindingAdapter]]
    FA --> G[(Connectome graph)]
```

## The handoff contract (produce, don't persist)

Per the boundary rule (`00`), L6 hands findings to Connectome's **L2 `FindingAdapter`**
(`docs/components/connectome/03-normalization-adapters.md`); Connectome is the single
writer to the graph. The handoff payload per finding:

| Field | From |
|---|---|
| `Finding` (modality, kind, attributes, measurements, locationId, status) | L5 |
| provenance (`asserted_by`, model/version, method, calibrated `confidence`, timestamp) | L2–L5 |
| `Evidence` (detections, measurements, agreement, conflicts) | L4–L5 |
| stable `id` + source study/series refs | L3/L6 |

Connectome then attaches the finding to its `Series`/`Study`, applies `located_at`, and
(separately, in its own L5) decides cross-modal correlation. **Perception asserts the
finding; it does not assert the lesion link.**

## Idempotency & updates

- **Stable IDs** — a finding's `id` is deterministic from (study, series, region), so a
  re-run **updates** the existing graph finding instead of duplicating it.
- **Versioned provenance** — each production run records its model versions; a re-run with
  a newer model supersedes prior provenance while keeping the audit trail.
- **Withdrawal** — if a re-run no longer supports a previously emitted finding, L6 emits a
  retraction (status change) rather than a silent delete — governed by **Sentinel** (C8).

## Candidate findings

Findings L5 marked **candidate** are handed off flagged, so Connectome stores them as
low-trust and the **Console** (C7) / **Reasoner** (C4) can request human confirmation.
Perception never promotes a candidate on its own.

## What L6 does *not* do

- It does not **persist** to the graph (Connectome does).
- It does not **correlate** findings across modalities (Connectome L5 does).
- It does not **diagnose** (Reasoner does).
- It does not **schedule** itself across the fleet (Conductor does); it exposes the
  per-study entry point Conductor calls.

## Entry points (conceptual)

| Entry point | Purpose |
|---|---|
| `perceiveStudy(studyId, options)` | full pipeline for one study → Finding(s) handed off |
| `reperceiveStudy(studyId, modelSet)` | re-run with specified models/versions |
| `perceiveReport(reportId)` | report-NLP-only production (e.g. for legacy reports) |

These are design-level signatures (no implementation) showing the surface Conductor and
Intake invoke.
