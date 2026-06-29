# Reasoner — Criteria Catalog (cross-cutting)

The formal **criteria sets** Reasoner applies, served by the criteria/guideline KB (L1)
and adjudicated deterministically at L5 (`06`). Criteria are **pluggable**: a presentation
selects the applicable sets; new sets are added to the KB without changing the pipeline.

## Catalog

| CriteriaSet | Domain | What it decides | Drives |
|---|---|---|---|
| **McDonald** (e.g. 2017) | Multiple sclerosis | dissemination in **space** & **time** | MS diagnosis |
| **WHO CNS** (e.g. CNS5) | CNS tumors | tumor type & grade (histology + molecular) | tumor diagnosis/grade |
| **RANO** | Neuro-oncology | treatment **response/progression** | progression (shared with Pathways/Connectome) |
| **mRS** | Any neuro | functional **outcome** (0–6) | outcome assessment |
| **Diagnostic-criteria (general)** | Vascular, infectious, etc. | e.g. stroke, abscess patterns | differential adjudication |

> Examples are illustrative; the model is general-neurology and scope-agnostic. RANO/mRS
> also appear in Connectome as progression schemes and in Pathways — Reasoner applies them
> for the **diagnostic** verdict; Pathways uses them for **monitoring**.

## Criterion shape

Each `Criterion` (`04`) is machine-checkable:

```
Criterion {
  id:            "mcdonald.dis"            // dissemination in space
  statement:     "≥1 T2 lesion in ≥2 of 4 CNS regions"
  logic:         count(distinct regions with T2 lesion) ≥ 2
  evidenceNeeded:[ finding.kind=T2-lesion, located_at.region ]
  result:        met | not-met | indeterminate   // set at L5
  evidenceRefs:  [ ... ]                          // the findings that satisfied it
}
```

A `CriteriaSet` combines its criteria with explicit logic (e.g. McDonald = DIS **and**
DIT). L5 records both the per-criterion results and the set-level verdict, each with
evidence refs — the basis of the auditable conclusion.

## Selection (which sets apply)

L2 (`03`) selects candidate sets from the presentation:

| Presentation signal | Candidate sets |
|---|---|
| demyelinating-pattern lesions | McDonald |
| mass lesion / neoplastic features | WHO CNS pathway |
| known tumor on treatment | RANO (response) |
| any, for outcome | mRS |

Multiple sets can apply; each produces its own verdict feeding the relevant
`DifferentialItem`.

## Versioning & governance

- Criteria sets are **versioned** (`McDonald-2017` vs a later revision); the version used
  is recorded on the result for reproducibility.
- Updates to the KB are governed by **Sentinel** (C8) — clinical criteria changes are
  controlled, audited, and the prior version stays available for re-deriving past
  conclusions.

## Why pluggable matters

New diseases/criteria become new KB entries + selection rules — **no pipeline change**.
The deterministic L5 evaluator works for any criterion expressed in the standard shape,
keeping Reasoner extensible across all of neurology.
