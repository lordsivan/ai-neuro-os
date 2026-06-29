# Connectome — Modality-Specific Modeling (cross-cutting)

How each of the four modalities populates the domain model (`04`): what a Study/Series
looks like, the characteristic `Finding` attributes, and the modality's role in the
diagnostic pathway. All four share the same `Finding` shape so they can be correlated
(`06`); this doc records what differs.

## MRI — the neuro workhorse

- **Study → Series:** one study holds many **sequences**, each a `Series`:
  - `T1`, `T1c` (post-contrast), `T2`, **`FLAIR`**, `DWI`/`ADC`, `SWI`, sometimes
    perfusion/spectroscopy.
- **Characteristic Finding attributes:** signal on each sequence (e.g. FLAIR-hyperintense,
  T1-hypointense), **enhancement pattern** (ring/solid/none), **diffusion restriction**
  (DWI↑/ADC↓), mass effect, edema, dimensions/volume.
- **Role:** primary detection & characterization for most neurology (tumor, MS, stroke
  evolution, neurodegeneration). Usually the **anchor** modality for a lesion.
- **Cross-modal note:** sequence identity (which `Series`) matters — a finding "on FLAIR"
  vs "on T1c" are different observations of the same lesion.

## CT — fast, acute, bone & blood

- **Study → Series:** non-contrast and/or contrast phases; thin/thick slices; bone vs
  soft-tissue windows.
- **Characteristic Finding attributes:** **density** (hypo-/iso-/hyper-dense, HU),
  **calcification**, **acute hemorrhage** (hyperdense), bony involvement, mass effect,
  midline shift.
- **Role:** emergency/first-line (hemorrhage, acute stroke, trauma), surgical planning,
  and where MRI is contraindicated. Often the **earliest** finding in the timeline.
- **Cross-modal note:** complements MRI — calcification/bone and acute blood are seen
  better here; same lesion, different evidence.

## Ultrasound — bedside & intra-operative

- **Study → Series:** B-mode views/planes; **Doppler** (color/spectral) series; for neuro:
  carotid duplex, transcranial Doppler, neonatal cranial US, and **intra-operative US**.
- **Characteristic Finding attributes:** echogenicity, margins, vascularity/flow (Doppler
  velocities, stenosis %), real-time location.
- **Role:** bedside screening (e.g. neonatal, carotid stenosis) and **intra-operative
  localization/resection guidance** — its real-time nature makes it the bridge between
  pre-op imaging and the surgical field.
- **Cross-modal note:** intra-op US is frequently the step that links a pre-op MRI lesion
  to the **specimen** that goes to histology (mechanism 3, registration).

## Histology — the confirmatory / ground-truth modality

- **Study → Series:** a **`Specimen`** (biopsy/resection) → **slides** → **stains** as
  `Series`: **H&E**, plus IHC panels and molecular assays.
- **Characteristic Finding attributes:** cellularity, atypia, mitoses, necrosis,
  microvascular proliferation, **grade**, and **IHC/molecular markers** (e.g. proliferation
  index; in neuro-onc examples IDH/MGMT/1p19q — illustrative, the model is scope-agnostic).
- **Role:** **confirms** the diagnosis (`confirmed_by` edge). Histology findings are
  usually the **highest-trust** node and often settle conflicts in correlation (`06`).
- **Cross-modal note:** spatially tied back to imaging via registration of the specimen's
  sampling location — the heart of **radiologic–pathologic correlation**.

## Modality comparison

| | MRI | CT | Ultrasound | Histology |
|---|---|---|---|---|
| Best at | soft-tissue characterization | acute blood / bone / calcium | real-time, bedside, intra-op | definitive tissue diagnosis |
| Series are | sequences (FLAIR, T1c, DWI…) | phases/windows | views + Doppler | slides + stains (H&E, IHC) |
| Typical role | anchor / detection | first-line / acute | localization / screening | confirmation / ground truth |
| Trust in correlation | high | high | moderate | highest |
| Spatial frame | MNI-registrable | MNI-registrable | probe-relative (registered intra-op) | specimen→imaging via registration |

## Implication for the model

- The shared `Finding` shape (`04`) holds modality-specific characteristics in a coded
  `attributes` map, so MRI signal, CT density, US flow and histology grade all live in
  one structure and remain comparable.
- The `Lesion` linchpin (`05`,`06`) is what unifies these very different observations into
  one navigable entity — exactly the cross-modal capability this component exists for.
