# Worked Example — Perception producing one MRI Finding

> **Illustrative only.** Not real, not runnable. Shows Perception's pipeline turning the
> MRI study from the Connectome case (`study-mri-001`) into the single `Finding`
> (`find-mri-001`) that Connectome later correlates across modalities. Data:
> [`study-mri-perception.json`](study-mri-perception.json). Ties to
> [`../connectome/patient-example.md`](../connectome/patient-example.md).

## The input

One MRI study, three sequences, plus the radiology report:

| Series | Sequence | What a model sees |
|---|---|---|
| `ser-mri-flair` | FLAIR | hyperintense lesion, left frontal |
| `ser-mri-t1c` | T1c (post-contrast) | ring enhancement |
| `ser-mri-dwi` | DWI/ADC | rim diffusion restriction |
| report | — | "Ring-enhancing mass in the left frontal lobe with surrounding edema; no hemorrhage." |

## Step 1 — Detection (L1/L2)

Four sources fire; adapters normalize each into a `Detection`:

| Detection | Source | kind | score / provenance |
|---|---|---|---|
| `det-flair` | FLAIR segmentation (image) | FLAIR-hyperintense lesion | 0.90, algorithm |
| `det-t1c` | T1c enhancement detector (image) | ring-enhancing mass | 0.92, algorithm |
| `det-dwi` | DWI restriction classifier (image) | rim diffusion restriction | 0.81, algorithm |
| `det-report` | report-NLP (text) | ring-enhancing mass, left frontal, **no hemorrhage** | human |

The report also yields a **negated** finding ("no hemorrhage") → recorded as absent, not
emitted.

## Step 2 — Fusion (L4)

All four detections **co-locate** (same left-frontal region after registration; report
laterality = left) and are **semantically compatible** → merged into one
**CandidateFinding** (`cand-001`). The three sequences are *views* of one lesion, and the
report sentence matches the image region → `human` provenance joins the group.
Disagreements: none material (all concur a left-frontal enhancing mass).

## Step 3 — Characterize + QC + calibrate (L5)

- **Attributes (reconciled):** `{ flair: hyperintense, enhancement: ring,
  diffusionRestriction: rim, edema: moderate }`.
- **Measurements:** volume 18.4 ml, diameter 33 mm (from segmentation).
- **Location:** left frontal (MNI), from registration.
- **QC:** pass.
- **Confidence:** calibrated to **0.91** — reinforced by image+report agreement across
  three sequences.
- **Status:** **confirmed**.
- **Parent series of record:** `ser-mri-t1c` (best defines the enhancing lesion).

## Step 4 — Production & handoff (L6)

L6 emits one `Finding` and hands it to **Connectome's `FindingAdapter`** — it does **not**
write the graph itself:

```
Finding find-mri-001  (modality MRI, ring-enhancing mass, vol 18.4ml, L frontal,
                       confidence 0.91, status confirmed,
                       evidence: [det-flair, det-t1c, det-dwi, det-report])
        ──▶ Connectome FindingAdapter ──▶ graph
```

This is exactly the `find-mri-001` that appears in the Connectome worked case — where
Connectome's **own** L5 then links it `manifestation_of` the left-frontal `Lesion`
alongside the CT, US and histology findings.

## The boundary, illustrated

| Perception did | Connectome will do |
|---|---|
| produce `find-mri-001` from 4 sources | persist it to the graph |
| characterize + calibrate confidence | attach it to its Series/Study |
| decide the finding **exists** | decide it's the **same lesion** as the CT/US/histology findings |

Perception asserts the finding; Connectome asserts the correlation.
