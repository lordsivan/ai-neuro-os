# Perception — Characterization & Confidence (L5)

L5 turns a fused **CandidateFinding** into a finished, trustworthy `Finding`: it attaches
the final **coded attributes** and **measurements**, runs **QC gating**, **calibrates
confidence**, **resolves conflicts**, and decides **candidate vs. promote**.

## Characterization — the final "what"

From the member detections, L5 assembles the finding's coded `attributes` map and
`measurements`, reconciling sources:

- **Attributes** — merge characterization scores into the agreed set, e.g. MRI
  `{flair: hyperintense, enhancement: ring, diffusionRestriction: rim}`, CT
  `{density: hypodense, calcification: false}`, histology `{grade: 4, necrosis: true}`.
  All terms are terminology-coded (RadLex/SNOMED).
- **Measurements** — choose the authoritative value per type (e.g. volume from the
  segmentation, longest diameter per RANO) and record alternates in `Evidence`.
- **Location** — finalize `located_at` from the registration hint (atlas region +
  laterality + coords).

Where image and report disagree on an attribute, the resolution policy (below) decides
which wins; the loser is retained in `Evidence`, not discarded.

## QC gating

The QC server's flags gate promotion:

| QC status | Effect |
|---|---|
| pass | normal flow |
| degraded (motion/artifact) | confidence down-weighted; finding kept as **candidate** |
| fail (unusable / failed segmentation) | finding **not produced** from that input; logged |

Perception does not emit findings from data QC rejects.

## Confidence calibration

The produced `confidence` is **calibrated**, not a raw model score:

- **Agreement** — independent models/sources concurring raises confidence; lone
  detections lower it.
- **Source mix** — image + report agreement is stronger than either alone.
- **QC** — degraded input down-weights.
- **Calibration** — raw scores are mapped to calibrated probabilities (e.g. reliability
  curves) so a `confidence` of 0.8 means the same thing across models/modalities — which
  matters because Connectome's trust filtering (`07`) thresholds on it.

The contributing factors are recorded in `Evidence` so the score is explainable.

## Conflict resolution

| Conflict | Default resolution |
|---|---|
| Image says enhancing, report says non-enhancing | flag; prefer higher-trust source; keep both in Evidence; lower confidence |
| Negated report finding vs. positive image detection | keep as low-confidence candidate, flagged for human review |
| Two models, opposite class | keep dominant by calibrated score; record disagreement |
| Histology vs. imaging (when both present) | histology (highest trust) wins for the attribute it speaks to |

Resolution never silently deletes evidence; it sets the finding's values + confidence and
records why.

## Candidate vs. promote

L5 labels each finding before handoff:

- **confirmed** — strong, QC-passed, multi-source agreement (or a human report assertion).
  Handed off as a normal `Finding`.
- **candidate** — single-source, degraded QC, or unresolved conflict. Handed off **flagged
  candidate**, so Connectome stores it as a low-trust observation for human/Reasoner
  confirmation (mirrors Connectome's candidate-vs-confirmed model,
  `docs/components/connectome/06-cross-modal-correlation.md`).

> Perception decides confidence/candidacy of a **finding's existence & attributes**.
> Whether two findings across modalities are the *same lesion* is **Connectome's**
> correlation job, not Perception's.

## Output to L6

A finished `Finding` (Connectome schema) per real observation: coded attributes,
measurements, `located_at`, calibrated `confidence`, `status` (confirmed|candidate),
provenance, and an `Evidence` trail. Ready for production & handoff (`07`).
