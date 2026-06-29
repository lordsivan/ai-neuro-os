# Connectome — Data Dictionary (controlled vocabularies)

The coding systems Connectome reuses, and where each is applied. Reusing these (rather
than inventing codes) is what keeps findings/diagnoses from different sources comparable
and the model interoperable.

## Coding systems

| System | Used for | Applied to |
|---|---|---|
| **HL7 FHIR** | resource model / data shapes | all clinical entities (`04`) |
| **DICOM** | acquisition hierarchy + metadata | `ImagingStudy`, `Series`, `Modality` |
| **SNOMED CT** | clinical findings & diagnoses | `Finding.kind`, `Diagnosis.condition`, history |
| **ICD-11** | diagnosis classification | `Diagnosis.condition` |
| **RadLex** | radiology terms | `Finding.attributes`, `Series.label` |
| **FMA** (Foundational Model of Anatomy) | anatomical structures | `AnatomicalLocation.atlasRegion` |
| **Neuroanatomy atlas** (AAL / Harvard-Oxford, in **MNI** space) | spatial regions/coords | `AnatomicalLocation` |
| **AIM** (Annotation & Image Markup) | image annotation/markup | `Finding` provenance |
| **RANO** | neuro-oncology response | `ProgressionAssessment.scheme` |
| **McDonald criteria** | MS diagnosis/dissemination | `Diagnosis`, `ProgressionAssessment` |
| **mRS** (modified Rankin Scale) | functional outcome | `ProgressionAssessment` |

> Disease-specific schemes (RANO, McDonald, mRS, WHO CNS) are **pluggable** — selected per
> case by `nature`/diagnosis, not hard-wired. The model is general-neurology; these are
> the schemes it plugs in when relevant.

## Modality vocabulary

| Code | Meaning | DICOM |
|---|---|---|
| `MRI` | Magnetic resonance imaging | MR |
| `CT` | Computed tomography | CT |
| `Ultrasound` | Ultrasound (incl. Doppler, intra-op) | US |
| `Histology` | Digital pathology / microscopy | SM / digital pathology |

## Common Series labels (RadLex-coded)

| Modality | Example Series labels |
|---|---|
| MRI | T1, T1c (post-contrast), T2, FLAIR, DWI, ADC, SWI, perfusion |
| CT | non-contrast, contrast (arterial/venous), bone window |
| Ultrasound | B-mode, color Doppler, spectral Doppler |
| Histology | H&E, IHC panel, molecular assay |

## Enumerations

| Enum | Values |
|---|---|
| `asserted_by` | `human`, `algorithm`, `embedding` |
| link `status` | `confirmed`, `candidate` |
| `corresponds_to.mechanism` | `lesion`, `location`, `registration`, `temporal`, `embedding` |
| `Lesion.nature` | `neoplastic`, `demyelinating`, `vascular`, `inflammatory`, `degenerative`, `unknown` |
| `ProgressionAssessment.status` | `stable`, `response`, `progression`, `recurrence`, `pseudo-progression` |
| `TreatmentPlan.intent` | `curative`, `palliative`, `diagnostic`, `surveillance` |

## Identifier conventions

- Domain `id`s are deterministic from source identifiers (e.g. DICOM
  StudyInstanceUID/SeriesInstanceUID, FHIR resource ids) so re-ingestion updates rather
  than duplicates.
- `Lesion.id` is **asserted** (created by correlation, `06`), not derived from any single
  source — it is the identity that survives across modalities and time.
