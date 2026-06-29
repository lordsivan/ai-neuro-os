# Console — View Composition (L4)

L4 is the **aggregate** layer: it takes the trust-flagged view-model parts that L2 gathered
from several sibling components and assembles them into **one coherent view**. It composes;
it does not re-query (that was L2) and it does not yet mark/explain (that is L5). A composed
`ViewModel` is a fixed snapshot of evidence the upper layers reason over.

## The composition job

A clinician question rarely maps to a single component. *"What's going on with this lesion?"*
needs the graph (cross-modal findings + timeline), Reasoner (the differential), and Pathways
(the plan + response history) — stitched around **one lesion**.

```mermaid
flowchart LR
    subgraph parts[L2 view-model parts]
      NAV[Connectome navigation\ncross-modal + timeline]:::ext
      DIF[Reasoner differential\n+ candidate dx]:::ext
      PLN[Pathways plan\n+ progression]:::ext
      DISC[Recall discovery\n(exploratory only)]:::ext
    end
    parts --> COMP[L4 · compose around the lesion]
    COMP --> VM[ViewModel · lesion-dashboard]
    VM --> L5[L5 · mark + explain]
    classDef ext fill:#ffe9b3,stroke:#b8860b;
```

## The lesion dashboard *(the flagship composition)*

For `les-001`, L4 assembles a multi-panel `ViewModel`:

| Panel | Source (via L2) | Contents | Trust note |
|---|---|---|---|
| **Cross-modal findings** | Connectome journey A | `find-mri-001`, `find-ct-001`, `find-us-001`, `find-histo-001` as manifestations of `les-001` | confirmed `manifestation_of` edges |
| **History** | Connectome journey B | `hist-001` (3-wk headaches + right-arm weakness) framing the findings | confirmed |
| **Differential** | Reasoner | ranked candidates; **top = candidate `dx-001`** (high-grade glioma) + look-alikes | **candidate** until confirmed |
| **Next test** | Reasoner `nextTest` | recommended **biopsy / histology** | recommendation |
| **Plan** | Pathways | `tx-001` (resection → chemoradiation → adjuvant) | candidate *or* active (post-confirm) |
| **Progression timeline** | Connectome/Pathways journey D | `prog-001` (stable) → `prog-002` (progression), `progresses_to` (73 d) | candidate assessments / confirmed timeline |
| **Discovery** *(exploratory)* | Recall journeys F/G | similar findings / cases (`pat-417`, …) | **discovered candidates** |

The dashboard is **lesion-centric** by construction — every panel hangs off `les-001`, the
same linchpin Connectome uses (`docs/components/connectome/00-overview.md`).

## Composition rules

1. **One anchor.** A view is composed around a single subject (here `les-001`); panels are
   joined by graph identity, not by guesswork.
2. **Preserve provenance.** Each item keeps its source `status` + provenance verbatim — L4
   never upgrades a candidate to confirmed; that only happens via confirmation (`07`).
3. **Honor the trust profile.** Under **strict**, candidate panels/items (differential's
   unconfirmed dx, discovery) are filtered out or collapsed; under **exploratory** they are
   included and flagged (`06`).
4. **Order by clinical narrative.** Findings → history → differential → next test → plan →
   timeline mirrors the work-up, so the view reads like a case, not a data dump.
5. **No new facts.** Composition only arranges what L2 supplied; if a panel has no data it is
   shown empty, never fabricated.

## Other compositions over the same scaffold

| `ViewModel.kind` | Anchor | Panels |
|---|---|---|
| `lesion-dashboard` | lesion | the seven panels above |
| `timeline` | lesion | findings-over-time + `ProgressionAssessment` chain |
| `differential` | lesion / case | Reasoner candidates + evidence, for confirm/reject |
| `discovery` | finding / case | Recall similar items (exploratory), each a confirmable candidate |
| `report-preview` | patient / case | the sections a `ReportDraft` will render (`08`) |

All share the same L1–L4 path; only the anchor and panel set differ. The composed
`ViewModel` is handed up to L5 to be marked, trust-filtered for display, and explained.
