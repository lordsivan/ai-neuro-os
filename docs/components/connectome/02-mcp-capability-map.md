# Connectome — MCP Capability Map (L1)

L1 is **external**. Connectome implements none of these functions; it *calls* MCP
servers that provide them. This document is the **contract** the rest of the component
depends on: what each server is for, what it takes in, what it returns, and which higher
layer consumes it. (Contracts are described at design level — no implementations here.)

> Rule: only **L2 adapters** call these servers. Nothing above L2 sees a raw MCP
> response.

## Capability catalog

| MCP server | Provides (domain function) | Representative inputs | Representative outputs | Feeds |
|---|---|---|---|---|
| **Imaging-archive** (DICOM / PACS) | Find & fetch imaging studies/series + metadata | patient id, modality, date range | study/series list, DICOM tags, image refs | `ImagingStudy`, `Series` (via L2) |
| **FHIR** | Clinical record access | patient id, resource type | `Patient`, `Encounter`, `Condition`, `CarePlan`, `DiagnosticReport`, `Specimen` | `Patient`, `PatientHistory`, `Diagnosis`, `TreatmentPlan` |
| **Segmentation / detection** | Produce imaging Findings (masks, measurements, characterization) | image ref, modality | lesion masks, volumes, RECIST/RANO measurements, attributes | `Finding` |
| **Registration** | Align studies to a common frame; map points across studies | two study refs (+ optional atlas) | transform, mapped coordinates, atlas labels | spatial `corresponds_to`, `AnatomicalLocation` |
| **Pathology** | Specimen / slide / stain + IHC & molecular results | specimen id, slide ref | histology findings, grade, IHC/molecular markers | `Finding`, `Specimen`, `Diagnosis` support |
| **Terminology / ontology** | Code & normalize clinical terms | free text or partial code | SNOMED CT / ICD-11 / RadLex / FMA codes + labels | coded attributes on all entities |
| **Embedding & vector-search** | Multimodal embeddings + ANN search | image ref or text, or a query vector | embedding vectors; ranked nearest neighbors + scores | discovery (L5/L6), embedding `corresponds_to` |
| **Retrieval / search** | Free-text + hybrid lexical/vector search | query string, filters | ranked findings/reports | discovery journeys (L6) |

## Per-server notes

### Imaging-archive (DICOM / PACS)
The system of record for MRI/CT/US acquisitions. Returns the Study → Series → Instance
hierarchy and metadata; pixel data stays in the archive (Connectome stores references,
not images). Histology whole-slide images are addressed via the **Pathology** server.

### FHIR
Backbone for non-imaging clinical data. Connectome maps FHIR resources 1:1 onto its
domain entities (see `03`, `04`) rather than inventing parallel structures.

### Segmentation / detection
The producer of imaging `Finding`s. Connectome treats it as a black box: in goes an
image reference + modality, out come structured findings with measurements and
attributes. (In the wider stack this is orchestrated by the future **C2 Perception**
component; Connectome only consumes the resulting findings.)

### Registration
Enables **mechanism 3** of cross-modal correlation (`06`): map a point/region in one
study to another, or to a standard space (e.g. MNI) for atlas labeling — including
**radiologic–pathologic** mapping of a histology specimen back to its imaging location.

### Pathology
The **confirmatory / ground-truth** modality. Supplies histology findings plus the
grade, IHC and molecular markers that often *settle* a diagnosis (e.g. IDH, MGMT, 1p/19q
in neuro-onc; these are general examples — the model is scope-agnostic).

### Terminology / ontology
Every coded attribute is normalized here, so findings/diagnoses from different sources
become comparable. This is what lets "shared anatomical location" (mechanism 2) work.

### Embedding & vector-search — the discovery substrate
Produces **multimodal embeddings**: image embeddings for MRI/CT/US/histology and text
embeddings for reports & patient history. Runs approximate-nearest-neighbor search over
them. (In the wider stack the index lifecycle is owned by the future **C3 Index**
component.) Backs the **discovery role** below.

### Retrieval / search
Hybrid lexical + vector search for free-text journeys ("find prior cases mentioning
tumefactive demyelination").

## Discovery role

Beyond navigating *known* edges, Connectome has a **discovery role** powered by
embeddings. Given a `Finding`, `Lesion`, `Study`, or a free-text query, it can
**discover** semantically/visually similar items **across modalities and across the
entire patient cohort** — surfacing candidate correlations and analogous prior cases
that were never explicitly linked.

- **Realized at:** L5 (correlation *discovery* — proposes candidate links) and L6
  (case/cohort discovery, semantic navigation).
- **Backed by:** the L1 embedding & vector-search server.
- **Wired in:** embeddings attach to L3 entities (`Finding`, `Study`, report text) as
  **vector attributes**, populated by L2 adapters when entities are normalized.
- **Trust:** discovery output is always **candidate** and tagged
  `asserted_by = algorithm` with a similarity `confidence` — never silently promoted to a
  confirmed link (see `06`).

## Contract stability

Higher layers code against the **domain shapes** in `04`, not against these server
payloads. If a server is swapped (e.g. a different segmentation vendor), only the L2
adapter for it changes — L3–L6 are untouched. That isolation is the payoff of the
layering rule.
