# Console (C7) — Overview *(stub — to brainstorm)*

> **Status:** placeholder. Named and scoped; not yet specified as a full L1–L6 component.

## Role

**Console** is the **interaction** component: the clinician-facing agent / UI. It turns
natural-language questions into Connectome navigation journeys, presents cross-modal
correlations and timelines, **explains** the evidence (provenance + confidence), and
drafts reports. It is the "shell" of the OS.

## Depends on

- **C1 Connectome** — the navigation journeys (`docs/components/connectome/07-navigation-and-queries.md`).
- **C4 Reasoner**, **C5 Pathways** — diagnosis & plan content to present.
- **C6 Conductor** — to dispatch multi-step requests.

## Responsibilities

| Concern | Notes |
|---|---|
| NL query → journey | map clinician questions to L6 traversals |
| Presentation | cross-modal views, timelines, lesion-centric navigation |
| Trust display | always distinguish confirmed vs. candidate / discovered links |
| Report drafting | structured summaries grounded in graph evidence |

## Open questions (for the brainstorm)

- Surface form: chat agent, structured viewer, or both.
- How the **strict vs. exploratory** trust profile (`07`) is chosen and shown.
- Human-in-the-loop confirmation of candidate correlations / discoveries.

When promoted, this folder grows the same `01`–`09` layer docs as Connectome.
