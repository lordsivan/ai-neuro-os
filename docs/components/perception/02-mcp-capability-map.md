# Perception — MCP Capability Map (L1)

The models Perception orchestrates. All are **external MCP servers** — Perception runs
none of them. Only **L2 adapters** call these. This is the contract the rest of the
component depends on.

## Capability catalog

| MCP server | Provides | Inputs | Outputs | Feeds |
|---|---|---|---|---|
| **Segmentation** | voxel/pixel masks of structures/lesions | image ref, modality, target | mask (RLE/contour), label, score | `Detection` (region) |
| **Detection** | bounding regions of findings | image ref, modality | boxes/points + class + score | `Detection` (region) |
| **Characterization / classification** | attributes of a region | image ref + region | coded attributes + scores (e.g. enhancement, restriction) | `Detection.attributes` |
| **Measurement / quantification** | sizes & quantitative values | image ref + region | volume, diameter, HU, ADC, Doppler velocity… | `Measurement` |
| **Report-NLP** | findings extracted from narrative | report text | finding spans, negation, laterality, anatomy, links | `Detection` (report) |
| **Quality control (QC)** | acquisition/segmentation quality | image ref / mask | QC flags, usability score | gating in `06` |
| **Registration** | localize a region to atlas/common frame | region + atlas | atlas label + coords | `located_at` hint for `Finding` |
| **Terminology** | code free-text terms | term/partial code | SNOMED CT / RadLex / FMA codes | coded attributes |

> Segmentation, detection, characterization and measurement are often the *same* vendor
> model exposing several functions; the catalog lists capabilities, not necessarily
> distinct servers. Registration and terminology are **shared** with Connectome's L1
> catalog (`docs/components/connectome/02-mcp-capability-map.md`) — same servers, reused.

## Per-capability notes

### Segmentation / detection — the region finders
Produce *where* a finding is. Segmentation gives precise masks (needed for volume);
detection gives faster coarse regions. Both emit a region + class + score that L2
normalizes into a `Detection`.

### Characterization / classification — the "what"
Given a region, return coded **attributes**: MRI enhancement pattern & diffusion status,
CT density & calcification, US echogenicity & vascularity, histology grade & markers.
These become the `Finding.attributes` map after fusion.

### Measurement / quantification
The numeric layer: volume, longest diameter (RECIST/RANO-style), HU, ADC values, Doppler
velocities, mitotic counts / proliferation index. Emits `Measurement`s.

### Report-NLP — the second source
Extracts findings already written in radiology/pathology reports: the finding phrase,
**negation** ("no enhancement"), **laterality**, **anatomy**, and uncertainty hedges.
Where the report references an image/series, it links the extracted finding to that
region so fusion can match it to a pixel detection. Output is `asserted_by = human`.

### QC — the gate
Flags unusable input (motion, artifact, failed segmentation). Low QC down-weights or
blocks promotion in `06`, so Perception doesn't emit findings from bad data.

### Registration & terminology — shared services
Registration localizes a detected region to an atlas region/coordinates (the
`located_at` hint Connectome needs). Terminology normalizes every attribute/finding term
to controlled codes so findings are comparable downstream.

## Contract stability

Higher layers code against the internal `Detection`/`Finding` shapes (`04`), never these
payloads. Swap a segmentation vendor → only its L2 adapter changes; fusion,
characterization and handoff are untouched. (In the wider stack, the *lifecycle* of these
models — versioning, eval, drift — is governed by **Sentinel** (C8); their scheduling is
**Conductor** (C6). Perception just calls them.)
