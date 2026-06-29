# Connectome — Domain Model & Ontology (L3)

L3 defines the **entities** (graph nodes) and their **standards mappings**. Everything
above L3 speaks only this vocabulary. Each entity is mapped to its FHIR/DICOM and
terminology counterparts — we **reuse** established models rather than invent new ones.

## Entity catalog

### Patient
The subject of care.
- **Key attributes:** `id`, demographics, identifiers.
- **Maps to:** FHIR `Patient`.

### PatientHistory / ClinicalHistory
The longitudinal clinical context that frames every finding.
- **Key attributes:** presenting complaint, comorbidities, medications, family/risk
  history, prior events, onset/timeline.
- **Maps to:** FHIR `Condition`, `MedicationStatement`, `FamilyMemberHistory`,
  `Observation`.

### Encounter
A visit/admission that groups studies and decisions in time.
- **Key attributes:** `id`, period, type (inpatient/outpatient/intra-op), reason.
- **Maps to:** FHIR `Encounter`.

### Modality
Controlled vocabulary of acquisition types.
- **Values (this scope):** `MRI`, `CT`, `Ultrasound`, `Histology`.
- **Maps to:** DICOM Modality + extension for histology (digital pathology).

### ImagingStudy
One acquisition session for a given modality.
- **Key attributes:** `id`, `modality`, `acquiredAt`, `sourceId` (StudyInstanceUID),
  body region.
- **Maps to:** DICOM Study / FHIR `ImagingStudy`. For histology → FHIR `Specimen` + a
  slide set.

### Series / Sequence
A coherent acquisition within a study.
- **Examples:** MRI sequences (T1, **T1c**, T2, FLAIR, DWI/ADC, SWI), CT phases, US
  views/Doppler, histology stains (**H&E**, IHC panels).
- **Key attributes:** `id`, `label` (RadLex-coded), `sourceId` (SeriesInstanceUID).
- **Maps to:** DICOM Series.

### Finding (Observation) — *the unit of cross-modal correlation*
A discrete observation within a study/series.
- **Examples:** mass, ring-enhancing lesion, restricted diffusion, FLAIR hyperintensity,
  demyelinating plaque, hemorrhage, infarct core, cellular atypia, mitoses.
- **Key attributes:** `id`, `modality`, `kind` (coded), `attributes` (coded
  characteristics), `measurements` (volume, diameter, ADC, density…), `embedding`
  (vector ref), provenance + confidence.
- **Maps to:** FHIR `Observation` / AIM annotation. Coded by SNOMED CT / RadLex.

### AnatomicalLocation
A shared spatial frame so findings can be compared by *where* they are.
- **Key attributes:** `id`, atlas region code, laterality, coordinates (in a named
  space, e.g. MNI).
- **Maps to:** FHIR `BodyStructure`; coded by **FMA** + a neuroanatomy atlas (AAL /
  Harvard-Oxford).

### Lesion / EntityOfInterest — *the linchpin*
The **abstract** clinical entity ("the tumor", "this plaque") that **manifests as many
Findings across modalities and across time**. It has no pixels of its own; it is the
identity that ties manifestations together.
- **Key attributes:** `id`, `label`, working `nature` (neoplastic/demyelinating/
  vascular/…), first/last observed.
- **Maps to:** no single FHIR resource — realized as a clinical concept linking
  `Observation`s; closest analogues are a tracked `Condition` + an imaging
  "lesion of interest". **This abstraction is Connectome's key contribution.**

### Diagnosis
A diagnostic conclusion.
- **Key attributes:** `id`, coded condition, certainty, basis, asserted-by, date.
- **Maps to:** FHIR `Condition` / `DiagnosticReport`; coded by SNOMED CT / ICD-11
  (+ domain classifications such as WHO CNS or McDonald criteria where relevant).

### TreatmentPlan / Intervention
The chosen management.
- **Examples:** resection/biopsy, radiotherapy, chemo/immuno/pharmacotherapy,
  watchful waiting.
- **Key attributes:** `id`, intent, components, schedule, status.
- **Maps to:** FHIR `CarePlan` / `Procedure` / `MedicationRequest`.

### ProgressionAssessment / Outcome
A time-stamped statement of how a Lesion/disease is evolving.
- **Key attributes:** `id`, timepoint, scheme (RANO / McDonald / mRS), status
  (stable/response/progression/recurrence), basis (which studies compared).
- **Maps to:** FHIR `Observation` / `ClinicalImpression`.

## Standards mapping summary

| Entity | FHIR | DICOM | Terminology |
|---|---|---|---|
| Patient | Patient | — | — |
| PatientHistory | Condition, MedicationStatement, FamilyMemberHistory | — | SNOMED CT, ICD-11 |
| Encounter | Encounter | — | — |
| ImagingStudy | ImagingStudy | Study | DICOM Modality |
| Series | — | Series | RadLex |
| Finding | Observation | (referenced) | SNOMED CT, RadLex, AIM |
| AnatomicalLocation | BodyStructure | — | FMA, AAL/Harvard-Oxford |
| Lesion | (concept) | — | — |
| Diagnosis | Condition, DiagnosticReport | — | SNOMED CT, ICD-11, WHO CNS |
| TreatmentPlan | CarePlan, Procedure, MedicationRequest | — | SNOMED CT |
| ProgressionAssessment | Observation, ClinicalImpression | — | RANO, McDonald, mRS |

## Shared attribute set (every entity)

| Attribute | Meaning |
|---|---|
| `id` | stable domain identifier |
| `source` | originating system/server |
| `asserted_by` | `human` \| `algorithm` \| `embedding` |
| `method` | how it was produced (e.g. "manual read", "seg-v2", "ANN cosine") |
| `confidence` | 0–1 |
| `timestamp` | when asserted |

These power the trust filtering used throughout correlation (`06`) and navigation
(`07`). The full attribute-level reference and diagrams are in `09`.
