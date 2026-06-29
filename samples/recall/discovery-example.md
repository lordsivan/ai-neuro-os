# Worked Example — Recall serving discovery

> **Illustrative only.** Not real, not runnable. Shows Recall answering the two discovery
> queries from the Connectome case — finding-similarity and similar-cases — and how the
> results flow back as **candidates**. Data:
> [`discovery-example.json`](discovery-example.json). Ties to
> [`../connectome/patient-example.md`](../connectome/patient-example.md) (the
> `discoveryExample` block) and [`../perception/study-mri-perception.md`](../perception/study-mri-perception.md).

## Setup — embed on ingest

When Perception produced `find-mri-001` and Connectome persisted it, Connectome's
`EmbeddingAdapter` called **`Recall.embed(find-mri-001, kind=finding)`**. Recall:

1. embedded the finding image region into space `mri-img-v1` (dim 768),
2. indexed it under `mri-img-v1/finding/cohort`,
3. returned `vectorRef = vec://mri/find-mri-001`.

Connectome stored only that `vectorRef` on the finding — the vector lives in Recall.

## Query F — find similar findings

Connectome's discovery journey F calls:

```
Recall.discoverSimilar(anchor=find-mri-001, k=3, filters={ scope: cohort })
```

Recall (L5) runs ANN in `mri-img-v1`, scopes to the cohort (Sentinel-authorized),
calibrates distance→similarity, dedups, and returns **candidates**:

| sourceRef | patient | modality | similarity | status |
|---|---|---|---|---|
| `find-mri-cohort-417` | pat-417 | MRI | 0.94 | candidate |
| `find-mri-cohort-088` | pat-088 | MRI | 0.91 | candidate |
| `find-ct-cohort-203` | pat-203 | CT | 0.88 | candidate |

These are the same three candidates shown in the Connectome case's `discoveryExample`.
Connectome decides whether any becomes a confirmed `corresponds_to` — Recall only proposed
them (`asserted_by = embedding`).

## Query G — find similar cases

Reasoner (or Console) asks for precedent:

```
Recall.discoverSimilarCases(patientId=pat-001, k=2, filters={ scope: cohort, time: prior })
```

Recall searches the **case** index (`text+img-v1/case/cohort`) using `pat-001`'s aggregated
case vector (findings + history), returning prior patients with similar presentation:

| case | similarity | recorded diagnosis | recorded outcome |
|---|---|---|---|
| `pat-417` | 0.92 | high-grade glioma | progression at 4 mo |
| `pat-512` | 0.86 | tumefactive demyelination | steroid-responsive |

Note the second is a *different* disease that *looks* similar — exactly the value of
discovery: it surfaces the alternative worth ruling out. Both are **candidate** precedents;
the Reasoner weighs them, Recall does not diagnose.

## What Recall did vs. didn't

| Recall did | Recall did **not** |
|---|---|
| embed + index find-mri-001 | store the vector in the graph (only the `vectorRef`) |
| return ranked, calibrated, cohort-scoped candidates | decide they're the same lesion (Connectome) |
| surface a look-alike different diagnosis | decide the diagnosis (Reasoner) |
| record scope/space/provenance on each result | promote anything to confirmed |

Recall says "these are similar, under this authority, in this space." Meaning is the
caller's to assign.
