# Connectome — Knowledge Graph / Aggregate (L4)

L4 is the **unified domain graph**. It aggregates the normalized entities (L3) produced
from many studies, modalities, encounters and timepoints into one connected structure
that L5 (correlation) and L6 (navigation) traverse.

## Why a property graph

The request is fundamentally about **navigation *between* things** — finding ↔ finding,
patient → diagnosis → treatment → progression. A **property graph** (typed nodes + typed,
directed, attributed edges) models that directly: traversal is a first-class operation,
edges carry the provenance/confidence we need, and the structure is open to new edge
types without schema migration.

The same model maps cleanly to:
- **RDF/OWL** — each node = a subject IRI, each edge = a predicate, properties =
  reified statements; keeps the model standards-portable and reasoner-friendly.
- **FHIR references** — graph edges correspond to FHIR resource references, so the graph
  can be projected back to a FHIR-native store.

## Nodes

The L3 entities (`04`): `Patient`, `PatientHistory`, `Encounter`, `ImagingStudy`,
`Series`, `Finding`, `AnatomicalLocation`, `Lesion`, `Diagnosis`, `TreatmentPlan`,
`ProgressionAssessment`, plus the `Modality` vocabulary.

## Edge catalog

Every edge carries the shared provenance attributes (`asserted_by`, `method`,
`confidence`, `timestamp`) so any traversal can be filtered by trust.

| Edge | From → To | Meaning |
|---|---|---|
| `has_history` | Patient → PatientHistory | the patient's clinical history |
| `has_encounter` | Patient → Encounter | a visit/admission |
| `has_study` | Encounter → ImagingStudy | imaging acquired in an encounter |
| `acquired_with` | ImagingStudy → Modality | which modality produced it |
| `contains_series` | ImagingStudy → Series | series within a study |
| `has_finding` | Series → Finding | a finding observed in a series |
| `located_at` | Finding → AnatomicalLocation | where the finding is |
| **`manifestation_of`** | **Finding → Lesion** | **this finding is one appearance of the lesion (cross-modal linchpin)** |
| **`corresponds_to`** | **Finding ↔ Finding** | **direct cross-modal/temporal correspondence between two findings** |
| `supports` | Lesion/Finding → Diagnosis | evidence backing a diagnosis |
| `confirmed_by` | Diagnosis → Finding (histology) | diagnosis proven by pathology |
| `treated_by` | Diagnosis → TreatmentPlan | management chosen for a diagnosis |
| `assessed_by` | Lesion → ProgressionAssessment | a progression statement about the lesion |
| `progresses_to` | ProgressionAssessment → ProgressionAssessment | temporal ordering of assessments |

## The aggregate pattern

Findings arrive independently — different studies, modalities, days, sources. L4's job is
to **aggregate** them around stable identities:

1. Findings attach to their `Series`/`ImagingStudy` (`has_finding`, `contains_series`).
2. Findings get an `AnatomicalLocation` (`located_at`).
3. When correlation (L5) decides two/more findings are the same entity, it asserts (or
   reuses) a `Lesion` and links each finding `manifestation_of` it.
4. The `Lesion` then accretes its `Diagnosis` (`supports`/`confirmed_by`),
   `TreatmentPlan` (`treated_by`) and `ProgressionAssessment` timeline (`assessed_by`/
   `progresses_to`).

The result: starting from **any** finding you can reach its lesion, its other-modality
manifestations, its diagnosis, its treatment, and its whole timeline.

## Identity & deduplication

- Nodes have **stable IDs** from L2 (deterministic from source IDs), so re-ingestion
  updates instead of duplicating.
- `Lesion` identity is **asserted**, not derived from pixels — it is created/merged by
  correlation logic (L5) and can be split/merged as evidence changes, with provenance on
  every change.

## Temporal model

Time is carried on nodes (`acquiredAt`, `timestamp`) and on the
`assessed_by`/`progresses_to` chain. A `Lesion` is effectively a thread through time:
its findings across successive studies, ordered, are the substrate for progression
queries (`07`).

## What L4 does *not* do

- It does not **decide** correspondences — that is L5 (`06`).
- It does not **call** MCP servers — that is L2 (`03`).
- It does not **process** images or text — that is L1.

L4 is structure and aggregation only. A class/ER view of all nodes, edges and attributes
is in `09`.
