# Perception — Adapters / Normalization (L2)

L2 is the **only** layer that calls L1 MCP servers. It maps each server's raw, native
output into one common internal shape — the **`Detection`** (`04`) — so everything above
L2 reasons over a single uniform representation regardless of which model (or a report)
produced it.

> L2 **translates**; it does not compute. No mask math, no NMS, no embedding — those are
> L1 or L4. L2 reshapes fields and attaches codes + provenance.

## Adapter responsibilities

Each adapter:

1. **Calls** its one MCP server.
2. **Maps** the native output into one or more `Detection`s (region or report-derived).
3. **Codes** native labels/terms via the terminology server.
4. **Stamps provenance** — `source` (server), `asserted_by` (`algorithm` for models,
   `human` for report-NLP), `model`/`modelVersion`, `method`, raw `score`, `timestamp`.
5. **Attaches the region reference** (series + mask/box or report span) so fusion can
   decide co-location.

## Adapter catalog

| Adapter | Source (L1) | Emits |
|---|---|---|
| **SegmentationAdapter** | Segmentation | `Detection{region: mask}` |
| **DetectionAdapter** | Detection | `Detection{region: box}` |
| **CharacterizationAdapter** | Characterization | attribute scores on a region |
| **MeasurementAdapter** | Measurement | `Measurement` attached to a region |
| **ReportAdapter** | Report-NLP | `Detection{source: report, span, negated, laterality}` |
| **QCAdapter** | QC | QC flags on a study/series/region |
| **LocationAdapter** | Registration + Terminology | atlas region + coords for a region |

## Mapping examples (design-level)

**Segmentation → Detection**
```
segmentation result   ──SegmentationAdapter──▶  Detection
  mask (RLE)               →  detection.region   = { seriesId, mask }
  label "enhancing tumor"  →  detection.kind     (RadLex/SNOMED)
  score 0.88               →  detection.score
  (model id, version)      →  provenance{ asserted_by: algorithm, model, modelVersion }
```

**Report-NLP → Detection**
```
report sentence "Ring-enhancing mass in the left frontal lobe; no hemorrhage."
   ──ReportAdapter──▶  Detection (mass)
       kind        = "ring-enhancing mass"        (coded)
       laterality  = left ; anatomy = frontal lobe (FMA)
       negated     = false
       provenance  = { asserted_by: human, source: report, span }
   ──ReportAdapter──▶  Detection (hemorrhage, NEGATED → dropped or recorded as absent)
```

**Measurement → Measurement**
```
measurement result   ──MeasurementAdapter──▶  Measurement
  volume_ml 18.4, diameter_mm 33   →  attached to the region's detection group
```

## The `Detection` — common shape

After L2, every source — a mask, a box, a measurement, a report sentence — is one of:

- **`Detection`** — a candidate observation in a region (from pixels or report), with
  `kind`, optional `attributes`/`score`, a `region` (series + mask/box or report span),
  and provenance.
- **`Measurement`** — a quantitative value bound to a region.

Fusion (L4) then groups detections that describe the **same** real finding. The adapter
boundary guarantees fusion never has to know whether a detection came from a CNN, a
transformer, or a sentence.
