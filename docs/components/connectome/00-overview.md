# Connectome — Overview (C1)

**Connectome** is the foundational component of `ai-neuro-os`: a **cross-modal
knowledge graph** for neurology. It turns scattered, multimodal patient data into one
navigable graph so that a clinician — or an agent — can move from a finding in one
modality to the same entity in another, and onward to diagnosis, treatment and
progression.

## Why this exists

Neurological work-up is inherently multimodal. The *same* lesion is described by an
**MRI** report ("FLAIR-hyperintense, ring-enhancing mass, left frontal"), seen on
**CT** ("hypodense lesion with mass effect"), localized at the bedside or intra-op by
**ultrasound**, and finally *proven* by **histology** ("WHO grade 4, IDH-wildtype").
Today these live in separate systems with no shared identity for "the lesion". A
clinician mentally fuses them; nothing in the data does.

Connectome makes that fusion **explicit and queryable**: one abstract `Lesion` node ties
its four modality-specific `Finding`s together, carries the `Diagnosis` it supports, the
`TreatmentPlan` chosen, and the timeline of `ProgressionAssessment`s — each link tagged
with who/what asserted it and how confident they were.

## What it is (and is not)

| Connectome **is** | Connectome **is not** |
|---|---|
| A knowledge representation + data model | An image viewer / PACS |
| Aggregation & orchestration logic | An ML inference engine |
| A navigable graph over modalities & time | A DICOM/FHIR server |
| A caller of MCP capabilities | A re-implementation of those capabilities |

All concrete compute and data access is delegated to **MCP servers** (L1). Connectome
implements **no low-level code** — only domain + aggregate logic (L2–L6).

## Scope (this phase)

- **Modalities:** histology, ultrasound, MRI, CT.
- **Plus:** patient history.
- **Clinical scope:** general neurology (tumor, demyelinating, vascular,
  neurodegenerative) — not limited to neuro-oncology.
- **Headline capability:** cross-modal finding correlation ("find this finding in
  another modality").
- **Also:** diagnosis, treatment plan, progression, and **embedding-backed discovery**.
- **Deliverable:** design specification + illustrative sample data. **No code.**

## Design principles

1. **No low-level code** — delegate to MCP servers; hold only domain + aggregate logic.
2. **Strict layering** — each layer depends only on the one directly below (see `01`).
3. **Reuse, don't reinvent** — anchor to FHIR, DICOM, SNOMED CT, RadLex, FMA/atlas,
   AIM, and disease-specific criteria (McDonald, RANO, mRS).
4. **Everything asserted carries provenance + confidence** — human vs. algorithm vs.
   embedding, so navigation can be filtered by trust; *candidate* vs. *confirmed* links
   are always distinguishable.
5. **The abstract `Lesion` is the linchpin** — it is what makes cross-modal correlation
   and progression tracking possible.

## Requirements traceability

Every concept the request named maps to a place in this spec:

| Requirement | Where it lives |
|---|---|
| Histology | `08-modality-modeling.md` (Histology), domain `Specimen`/`Finding` |
| Ultrasound | `08-modality-modeling.md` (Ultrasound) |
| MRI | `08-modality-modeling.md` (MRI) |
| CT | `08-modality-modeling.md` (CT) |
| Patient history | `04-domain-model.md` (`PatientHistory`) |
| Navigation between modalities | `07-navigation-and-queries.md`, `06-cross-modal-correlation.md` |
| Disease diagnosis | `04` (`Diagnosis`), `07` journeys |
| Treatment plan | `04` (`TreatmentPlan`), `07` journeys |
| Progression | `04` (`ProgressionAssessment`), temporal edges in `05` |
| Finding in one modality → another | `06-cross-modal-correlation.md` (the core) |
| Knowledge representation | `05-knowledge-graph.md` |
| Data model | `04-domain-model.md`, `09-data-model-reference.md` |
| MCP-delegated capabilities | `02-mcp-capability-map.md` |
| Layered architecture | `01-layered-architecture.md` |
| Discovery role | `02` (Discovery role), `06` (mechanism 5), `07` (discovery journeys) |
| Embeddings | `02-mcp-capability-map.md` (embedding server) |

## Reading order

`01` layers → `02` capabilities → `03` adapters → `04` domain model → `05` graph →
`06` correlation → `07` navigation → `08` modalities → `09` reference →
`data-dictionary.md`. Then the worked case in `samples/connectome/`.
