# Worked Example — Instance Graph

The case from [`patient-example.md`](patient-example.md) / [`patient-example.json`](patient-example.json)
rendered as a Connectome graph. One abstract **Lesion** (`les-001`) ties the four
modality findings together; from it hang diagnosis, treatment and the progression
timeline.

## Full instance graph

```mermaid
flowchart TD
    P["Patient<br/>pat-001 · 52F"]
    H["PatientHistory<br/>headaches + R-arm weakness"]
    P -->|has_history| H

    P -->|has_study| SCT["ImagingStudy · CT<br/>2025-12-31"]
    P -->|has_study| SMR["ImagingStudy · MRI<br/>2026-01-02"]
    P -->|has_study| SUS["ImagingStudy · US (intra-op)<br/>2026-01-10"]
    P -->|has_study| SHI["ImagingStudy · Histology<br/>2026-01-12"]

    SCT -->|has_finding| FCT["Finding (CT)<br/>hypodense, mass effect"]
    SMR -->|has_finding| FMR["Finding (MRI)<br/>ring-enhancing mass"]
    SUS -->|has_finding| FUS["Finding (US)<br/>hyperechoic lesion"]
    SHI -->|has_finding| FHI["Finding (Histology)<br/>grade 4 glial neoplasm"]

    FCT -->|located_at| LOC["AnatomicalLocation<br/>L frontal (MNI)"]
    FMR -->|located_at| LOC
    FUS -->|located_at| LOC
    FHI -->|located_at| LOC

    FCT -->|manifestation_of<br/>loc · 0.84| LES(("Lesion<br/>les-001<br/>left frontal mass"))
    FMR -->|manifestation_of<br/>lesion · 0.97| LES
    FUS -->|manifestation_of<br/>registration · 0.90| LES
    FHI -->|manifestation_of<br/>registration · 0.95| LES

    FMR -.corresponds_to<br/>location.- FCT
    FMR -.corresponds_to<br/>registration.- FHI

    LES -->|supports| DX["Diagnosis<br/>High-grade glioma"]
    DX -->|confirmed_by| FHI
    DX -->|treated_by| TX["TreatmentPlan<br/>resection + chemoradiation"]

    LES -->|assessed_by| PR1["Progression · RANO<br/>2026-02-01 · stable"]
    LES -->|assessed_by| PR2["Progression · RANO<br/>2026-04-15 · progression"]
    PR1 -->|progresses_to · 73d| PR2

    style LES fill:#ffe9b3,stroke:#b8860b,stroke-width:2px
    style FHI fill:#e8f5e9,stroke:#2e7d32
```

## Reading the graph

- **The hub is the `Lesion`** — every modality finding points into it via
  `manifestation_of` (solid), each labeled with the **mechanism** (`06`) and **confidence**.
- **Dashed `corresponds_to`** edges are the direct finding↔finding links (CT↔MRI by
  location; MRI↔Histology by registration).
- **Histology** (green) is the highest-trust node and the one that `confirmed_by` the
  diagnosis.
- **From the lesion**, navigation reaches diagnosis → treatment, and the **progression
  chain** (`assessed_by` / `progresses_to`).

## The core journey, isolated

"Find this MRI finding in the other modalities" = pivot through the lesion:

```mermaid
flowchart LR
    FMR["MRI finding"] -->|manifestation_of| LES(("Lesion"))
    LES -->|manifestation_of⁻¹| FCT["CT finding"]
    LES -->|manifestation_of⁻¹| FUS["US finding"]
    LES -->|manifestation_of⁻¹| FHI["Histology finding"]
    style LES fill:#ffe9b3,stroke:#b8860b,stroke-width:2px
```

## Discovery (candidate, not stored)

`discover_similar(find-mri-001)` (journey F, mechanism 5) returns **candidate** cohort
matches — rendered distinctly because they are unconfirmed:

```mermaid
flowchart LR
    FMR["MRI finding<br/>find-mri-001"] -.embedding 0.94.-> C1["cohort: pat-417 (MRI)"]
    FMR -.embedding 0.91.-> C2["cohort: pat-088 (MRI)"]
    FMR -.embedding 0.88.-> C3["cohort: pat-203 (CT)"]
    style C1 stroke-dasharray: 5 5
    style C2 stroke-dasharray: 5 5
    style C3 stroke-dasharray: 5 5
```
