# Perception — Modality-Specific Perception (cross-cutting)

Each modality has a different model suite, region semantics, and characteristic
attributes/measurements. All produce the same `Finding` shape (`04`) so Connectome can
correlate them; this doc records what differs per modality. Pairs with Connectome's
`docs/components/connectome/08-modality-modeling.md`.

## MRI

- **Per-sequence models** — segmentation on FLAIR/T2 (lesion extent, edema), enhancement
  detection on T1c, diffusion-restriction classification on DWI/ADC, susceptibility on SWI.
- **Fusion** — cross-sequence: one lesion, many sequence views → one finding with
  per-sequence attributes (`05`).
- **Attributes produced** — `flair`, `enhancement` (ring/solid/none), `diffusionRestriction`,
  `edema`, mass effect.
- **Measurements** — volume (from segmentation), longest diameter, ADC values.
- **Report-NLP** — radiology report sentences per lesion, often with sequence references.

## CT

- **Models** — detection/segmentation tuned for density-based findings; hemorrhage and
  acute-stroke (ASPECTS-style) detectors; bone/calcification.
- **Attributes produced** — `density` (HU class), `calcification`, `hemorrhage`, mass
  effect, midline shift.
- **Measurements** — HU values, diameter, midline shift (mm).
- **Report-NLP** — frequently the *first* report in the timeline (ER); strong source.

## Ultrasound

- **Models** — B-mode lesion detection; **Doppler** flow quantification; carotid stenosis
  estimation; intra-operative lesion localization.
- **Region semantics** — probe-relative; registration to pre-op MRI is what gives an atlas
  `located_at` (and the bridge to histology, per Connectome `06` mechanism 3).
- **Attributes produced** — `echogenicity`, `margins`, `vascularity`.
- **Measurements** — diameter, Doppler velocities, stenosis %.
- **Note** — often human-read intra-op; report-NLP / structured operative notes are a key
  source, so `asserted_by` skews `human`.

## Histology

- **Models** — whole-slide image (WSI) analysis: tile-level classification, cellularity /
  mitosis detection, IHC quantification, grading support.
- **Region semantics** — slide/tile coordinates; the specimen's sampling site maps back to
  imaging via registration (radiologic–pathologic correlation).
- **Attributes produced** — `cellularity`, `atypia`, `mitoses`, `necrosis`,
  `microvascularProliferation`, `grade`, IHC/molecular markers.
- **Measurements** — proliferation index (e.g. Ki-67 %), mitotic count.
- **Trust** — pathologist-confirmed histology findings are the **highest-confidence**
  Perception produces; they often resolve cross-modal conflicts downstream.

## Comparison

| | MRI | CT | Ultrasound | Histology |
|---|---|---|---|---|
| Model suite | per-sequence seg/detect/classify | density/hemorrhage/stroke | B-mode + Doppler | WSI tile analysis |
| Fusion emphasis | cross-sequence | report + image | registration to MRI | tile → region aggregation |
| Dominant source | image + report | report (acute) + image | human/report + image | image + pathologist report |
| Confidence ceiling | high | high | moderate | highest |

## Implication

The modality differences live in **which L1 models run** and **which attributes/measurements**
result — the L2→L6 machinery (normalize → fuse → characterize → hand off) is identical
across modalities. Adding a modality means adding its model adapters (L2) and attribute
vocabulary (terminology), not changing the pipeline.
