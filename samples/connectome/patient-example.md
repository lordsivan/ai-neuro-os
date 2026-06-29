# Worked Example — Cross-Modal Correlation in One Case

> **Illustrative only.** Not a real patient, not runnable code. This case instantiates the
> Connectome model (`docs/components/connectome/04`–`06`) to show the four modalities
> unified by one abstract `Lesion`, then diagnosis → treatment → progression. The data is
> in [`patient-example.json`](patient-example.json); the graph is in
> [`correlation-graph.md`](correlation-graph.md).

## The patient

A 52-year-old woman (`pat-001`) presents with **3 weeks of progressive headaches and
right-arm weakness** (`PatientHistory hist-001`). That history frames everything that
follows.

## Four modalities, one entity

The same left-frontal mass is observed four ways. In Connectome each observation is a
separate `Finding`, and all four are **`manifestation_of`** one `Lesion` (`les-001`) — the
linchpin that makes them navigable as a single entity.

| When | Modality | Study | Finding | What it shows | Trust |
|---|---|---|---|---|---|
| Dec 31 (ER) | **CT** | `study-ct-001` (non-contrast) | `find-ct-001` | hypodense lesion, mass effect, 4 mm midline shift | algorithm 0.86 |
| Jan 02 | **MRI** | `study-mri-001` (FLAIR/T1c/DWI) | `find-mri-001` | FLAIR-hyperintense, **ring-enhancing**, rim diffusion restriction | algorithm 0.91 |
| Jan 10 | **Ultrasound** | `study-us-001` (intra-op B-mode) | `find-us-001` | hyperechoic intra-axial lesion; guides resection | human 0.80 |
| Jan 12 | **Histology** | `study-histo-001` (H&E + IHC) | `find-histo-001` | high-grade glial neoplasm, grade 4, Ki-67 28% | human **0.98** |

All four `located_at` the same atlas region (`loc-lfront`: left frontal, MNI), which is
what let mechanisms 2–3 (`06`) propose the links before histology confirmed them.

## How the links were made (the five mechanisms in action)

- **CT ↔ MRI** — `corresponds_to`, **mechanism: location** (same atlas region), then
  ratified by the radiologist.
- **MRI → Lesion** — **mechanism: lesion** (radiologist asserted this is the lesion of
  interest); MRI is the **anchor** modality.
- **US → Lesion** — **mechanism: registration** (intra-op US registered to pre-op MRI to
  guide the biopsy).
- **Histology → Lesion** and **MRI ↔ Histology** — **mechanism: registration**
  (radiologic–pathologic correlation: the specimen's sampling site mapped back to the MRI
  lesion). Histology is the **highest-trust** node and **confirms** the diagnosis.

## Diagnosis → treatment → progression

- **Diagnosis** (`dx-001`): *High-grade glioma* — imaging **supported** it; histology
  **confirmed_by** it (grade 4).
- **TreatmentPlan** (`tx-001`): maximal safe resection → concurrent chemoradiation →
  adjuvant chemotherapy (`treated_by`).
- **Progression** (RANO): post-op **stable** (`prog-001`, Feb 01) → **progression**
  (`prog-002`, Apr 15, new enhancement at the resection margin). The `Lesion` is
  `assessed_by` both, ordered by `progresses_to`.

## The journeys this case supports (`07`)

- **A — finding → other modalities:** start at `find-mri-001`, reach CT/US/histology via
  `les-001`.
- **B — history → imaging → diagnosis:** `hist-001` → findings → `dx-001`.
- **C — diagnosis → treatment:** `dx-001` → `tx-001`.
- **D — progression timeline:** `les-001` → `prog-001` → `prog-002`.
- **E — histology → which radiology finding:** `find-histo-001` → `les-001` →
  `find-mri-001`/`find-ct-001`.
- **F — discovery:** `discover_similar(find-mri-001)` returns three **candidate** similar
  lesions from the cohort (see `discoveryExample` in the JSON) — embedding-backed,
  unconfirmed, for hypothesis generation.

## Generality (other neurology cases)

The same model handles, without change:

- **Multiple sclerosis** — a `Lesion` of `nature: demyelinating`; MRI FLAIR/T1c plaques as
  findings, McDonald criteria as the progression scheme; histology rarely needed.
- **Ischemic stroke** — CT (early hypodensity / ASPECTS) + MRI (DWI restriction) findings
  on one vascular `Lesion`; carotid **ultrasound** Doppler contributes the source; mRS as
  the outcome scheme.

In each, the four-modality `Finding` shape, the abstract `Lesion`, and the five correlation
mechanisms are identical — only the codes, schemes and which modalities participate differ.
