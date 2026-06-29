# Recall — Granularity & Vector Spaces (cross-cutting)

Two cross-cutting concerns that shape everything Recall does: **what** it embeds
(granularity) and **how vectors stay comparable over time** (vector spaces + versioning),
plus the **privacy** scoping that rides on both.

## Granularities

| Granularity | Embeds | Index | Powers |
|---|---|---|---|
| **finding** | one Finding's image region (per modality) | `<space>/finding/<scope>` | finding-to-finding similarity (journey F); cross-modal candidates (Connectome mechanism 5) |
| **study** | a study's salient content | `<space>/study/<scope>` | study-level lookup |
| **case** | aggregate of a patient's findings + history (`05`) | `<space>/case/<scope>` | similar cases / cohort (journey G); Reasoner precedent |
| **text** | report & history narrative | `<text-space>/text/<scope>` | free-text / semantic search (journey H) |

A query searches **one** granularity's index. Finer vectors compose into the `case`
vector; the `case` vector is what cohort similarity compares.

## Vector spaces

A **VectorSpace** = `model + version + modality/textual + dim + metric`. Vectors are only
comparable **within** a space.

- **Per-modality image spaces** — e.g. `mri-img-v1`, `ct-img-v1`, `histo-img-v1`.
- **Cross-modal space(s)** — a shared/aligned space (or a learned bridge between per-modality
  spaces) so an MRI anchor can retrieve CT/US/histology neighbors (`06`).
- **Text space** — e.g. `text-v2` for reports/history.

Rule: **never compare across spaces.** A search names its space; the index is partitioned
by space (`05`).

## Versioning & model change

When an embedding model/version is approved (governed by **Sentinel**, C8):

1. A **new VectorSpace** is created (the old one is marked `deprecated`, not deleted).
2. Affected content is **re-embedded** into the new space (backfill, scheduled by
   **Conductor**, C6) — idempotent by `(sourceRef, space)`.
3. Searches move to the new space once backfill completes; deprecated spaces stay readable
   for audit/repro until retired.
4. Connectome's stored `vectorRef`s are stable within a space; a space change updates them
   via the same `embed()` path.

This is what keeps "discovery" reproducible: a candidate from last year can be re-derived
in the exact space + calibration that produced it.

## Privacy & scope (Sentinel-governed)

Cross-patient memory is powerful and sensitive. Scope is a first-class filter:

| Scope | Meaning |
|---|---|
| `patient:<id>` | only this patient's vectors |
| `cohort` | cross-patient (de-identified) retrieval |

- **Cohort scope is gated by Sentinel** (C8): who may run cohort discovery, on what
  de-identified partitions, and with what audit.
- L5 enforces the caller's authorized scope and records the scope used on every result, so
  a candidate always carries *under what authority* it was found.
- Indexes are physically partitioned by scope (`05`) so patient-only queries cannot leak
  cohort vectors.

## Why this lives in one doc

Granularity, spaces and scope are the three dials that every Recall operation sets —
embed (`03`), index (`05`), search (`06`), serve (`07`). Keeping them defined once, here,
keeps those layers consistent.
