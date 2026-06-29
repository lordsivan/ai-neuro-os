# Recall — Index / Aggregate (L4)

L4 owns the **index lifecycle**: it builds and refreshes the searchable indexes, organizes
them by **vector space** and **granularity**, and **aggregates** case-level vectors from
finer ones. This is the "owns the embedding + vector-index lifecycle" mandate made
concrete.

## Index organization

Indexes are partitioned so a search only ever compares like with like:

```
namespace = <vectorSpace> / <granularity> / <scope>
   e.g.  mri-img-v1 / finding / cohort
         text-v2     / case    / patient:pat-001
```

- **By vector space** — never mix models/versions in one index (`08`).
- **By granularity** — finding / study / case / text indexes are separate (a finding
  query searches finding vectors, a case query searches case vectors).
- **By scope** — patient-local vs. cohort partitions support privacy scoping at search
  time (`06`, Sentinel-governed).

## Build & refresh

| Trigger | Action |
|---|---|
| New Finding/Study embedded (on ingest, via `embed()`) | upsert IndexEntry into its namespaces |
| Connectome node updated (re-perceived finding) | re-embed → upsert (same stable id) |
| New embedding model/version approved | create new VectorSpace + **backfill re-embed** (`08`) |
| Node retracted | remove IndexEntry |

Bulk operations (e.g. backfilling a new space across the cohort) are **scheduled by
Conductor** (C6); L4 performs them idempotently keyed by `sourceRef` + space.

## Case-level aggregation

A **case** vector is built from a patient's finer vectors so similar-case search (journey
G) has something to query:

- Inputs: the patient's finding vectors + study vectors + history/report **text** vectors.
- Method: a defined aggregation (e.g. attention/weighted pool over findings + history),
  recorded so it is reproducible and explainable.
- Output: one `case` Embedding per patient per space, refreshed when the patient's
  findings change.

> Aggregation is *composition of existing embeddings*, not new model inference of its own —
> consistent with "no low-level code" (any learned pooling is itself an MCP capability).

## Idempotency & identity

- Index entries are keyed by `(sourceRef, space, granularity)` → re-embedding **updates**
  rather than duplicates.
- `vectorRef`s are stable for a given `(sourceRef, space)` so Connectome's stored ref stays
  valid across refreshes within a space.

## What L4 does *not* do

- It does not **search** (that's L5) or **serve** (L6).
- It does not **call embedding models** (that's L2/L1) — it *orchestrates* when to.
- It does not **calibrate** similarity — raw vectors only; calibration is L5.

L4's product: well-partitioned, current indexes that L5 queries.
