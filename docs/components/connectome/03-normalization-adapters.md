# Connectome — Normalization / Adapters (L2)

L2 is the **only** layer that talks to L1 MCP servers. Its job: take raw server output
and emit clean **L3 domain entities** (`04`). Above L2, no one ever sees a DICOM tag, a
FHIR resource, or a raw vector — only domain objects. This is what makes the
"no low-level code" boundary real and enforceable.

> L2 **translates**; it does not compute. No segmentation, no registration math, no
> embedding generation happens here — those are L1. L2 maps fields and attaches codes.

## Adapter responsibilities

Each adapter:

1. **Calls** its MCP server (and only its server).
2. **Maps** the response onto one or more domain entities/edges.
3. **Codes** free-text/native codes via the terminology server into the controlled
   vocabularies in `data-dictionary.md`.
4. **Stamps provenance** — `source` (which server/system), `asserted_by`
   (human|algorithm|embedding), `method`, `confidence`, `timestamp` — on everything it
   emits.
5. **Assigns stable identity** — deterministic domain IDs so re-ingesting the same
   source updates rather than duplicates.

## Adapter catalog

| Adapter | Source (L1) | Emits (L3) |
|---|---|---|
| **ImagingStudyAdapter** | Imaging-archive | `ImagingStudy`, `Series`, `acquired_with`→`Modality`, `contains_series` |
| **ClinicalAdapter** | FHIR | `Patient`, `PatientHistory`, `Encounter`, `Diagnosis`, `TreatmentPlan` |
| **FindingAdapter** | Segmentation/detection | `Finding` (+ measurements/attributes), `has_finding`, `located_at` |
| **PathologyAdapter** | Pathology | `Specimen`, histology `Finding`, IHC/molecular attributes, `Diagnosis` support |
| **LocationAdapter** | Registration + Terminology | `AnatomicalLocation` (atlas-coded), registration transforms |
| **EmbeddingAdapter** | Embedding & vector-search | vector attributes on `Finding`/`Study`/report text |
| **TerminologyAdapter** | Terminology/ontology | coded attributes applied across all of the above |

## Mapping examples (design-level)

**DICOM → ImagingStudy / Series**

```
imaging-archive study  ──ImagingStudyAdapter──▶  ImagingStudy
  StudyInstanceUID            →  study.sourceId
  Modality (MR/CT/US)         →  study.modality            (Modality vocab)
  StudyDate/Time              →  study.acquiredAt
  series[*].SeriesDescription →  Series.label (e.g. "FLAIR")  (RadLex-coded)
  series[*].SeriesInstanceUID →  Series.sourceId
```

**FHIR → Patient / Diagnosis / TreatmentPlan**

```
FHIR Patient        ──ClinicalAdapter──▶  Patient
FHIR Condition      ──ClinicalAdapter──▶  Diagnosis      (SNOMED CT / ICD-11)
FHIR CarePlan +     ──ClinicalAdapter──▶  TreatmentPlan
  Procedure/MedicationRequest
FHIR DiagnosticReport ─────────────────▶  links report text → Finding/Diagnosis
```

**Segmentation → Finding**

```
segmentation result  ──FindingAdapter──▶  Finding
  mask + volume/diameter  →  finding.measurements
  characterization        →  finding.attributes (e.g. enhancement, diffusion) (RadLex/SNOMED)
  modality + study ref    →  has_finding edge from the Series
  centroid + atlas label  →  located_at → AnatomicalLocation (via LocationAdapter)
```

**Embedding → vector attribute**

```
embedding vector  ──EmbeddingAdapter──▶  finding.embedding = {model, dim, vectorRef}
                                          (vectorRef points into Recall's vector index;
                                           the raw vector is NOT stored in the graph)
```

> **Reconciled with Recall (C3).** The `EmbeddingAdapter` does **not** call the embedding
> MCP server directly — it delegates to **Recall** (`Recall.embed(sourceRef)`), the single
> vector-memory service, and stores the `vectorRef` Recall returns. Likewise, the discovery
> in `06` (mechanism 5) and journeys F/G/H in `07` call **Recall's** discovery API, not the
> raw MCP server. See `docs/components/recall/`.

## The adapter boundary contract

- **Input:** raw MCP payload.
- **Output:** valid L3 entities/edges per `04`, each carrying provenance + confidence.
- **Guarantee to L3+:** no raw formats leak upward; all codes are normalized; identity is
  stable; every assertion is attributable.

Because of this contract, swapping an MCP vendor changes exactly one adapter and nothing
above it.
