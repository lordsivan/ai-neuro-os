# Perception — Fusion / Aggregate (L4)

L4 is where many `Detection`s become **one** finding. Segmentation, detection,
characterization, measurement and report-NLP each emit separate detections about the
*same* lesion; fusion groups them, so Connectome receives **one observation per real
finding**, not a pile of overlapping model outputs.

> Fusion is the heart of Perception. "Fuse, don't flood": Connectome must never see raw
> detections.

## What fusion groups over

Three axes of redundancy collapse here:

1. **Across models** — FLAIR segmentation + an enhancement detector both fire on the same
   region (ensemble agreement).
2. **Across series/sequences** — the same lesion appears on FLAIR, T1c and DWI of one MRI
   study; these are different *views* of one finding, not three findings.
3. **Across sources** — an image detection and a report sentence describe the same lesion
   (pixel ↔ narrative).

## Grouping signal

Two detections are candidates to fuse when they co-locate and concur:

- **Spatial overlap** — region IoU / centroid distance (image↔image), or atlas-region +
  laterality match after registration (cheap fallback, and the bridge to report
  detections that have only anatomy/laterality).
- **Semantic compatibility** — compatible `kind` (a "mass" and a "ring-enhancing mass"
  are compatible; a "mass" and an "old infarct" are not).
- **Report linkage** — when report-NLP provided an explicit image/series reference, that
  is a strong direct link.

Detections that overlap and are compatible are merged into a **CandidateFinding** with an
`Evidence` record listing its members.

## Fusion outcomes

| Situation | Outcome |
|---|---|
| Multiple models agree on a region | one CandidateFinding, confidence reinforced |
| Models disagree (one fires, one doesn't) | CandidateFinding kept, confidence lowered, disagreement recorded |
| Image detection + matching report sentence | fused; report contributes `human` provenance + attributes (e.g. laterality) |
| Report finding with **no** image detection | CandidateFinding from report alone (kept; flagged image-unconfirmed) |
| Image detection contradicted by a **negated** report finding | conflict flag for L5 to resolve |
| Two distinct lesions nearby | kept separate (overlap below threshold / incompatible kind) |

## Cross-sequence handling (MRI especially)

A finding's appearance differs per sequence and that is **information**, not duplication.
Fusion keeps the lesion as one CandidateFinding while preserving each sequence's
contribution, so characterization (`06`) can record "FLAIR-hyperintense, T1c
ring-enhancing, DWI rim-restricted" as attributes of the *single* finding. The parent
`Series` of record is chosen by a rule (e.g. the sequence the lesion is best defined on,
typically T1c for enhancing tumors).

## What L4 hands to L5

A set of **CandidateFinding**s for the study, each with:
- its member detections + measurements (`Evidence`),
- a merged region + `located_at` hint,
- a preliminary confidence from agreement,
- any conflict/disagreement flags.

L4 **does not** finalize attributes, calibrate confidence, or decide promotion — that is
L5 (`06`). L4 only decides **what is one finding**.
