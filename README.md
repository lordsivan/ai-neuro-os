# ai-neuro-os

> An AI operating system for **neuro-imaging & neurology** — built as a set of
> cooperating **components**, each one a domain/orchestration module that calls
> **MCP servers** for all heavy lifting. The OS itself implements **no low-level
> code**: no image processing, no DICOM parsing internals, no model inference. It
> *orchestrates* capabilities and *represents knowledge*.

This repository is currently in an **architecture & design phase**. It contains
design specifications and illustrative sample data — **no application code yet**.

> **Two-repo topology.** The **design / architecture source of truth** is the
> **`ai-neuro-stack`** repo (owns the versioned `contracts/`). The **implementation** is the
> **`ai-neuro-ba-app`** repo, which *pins* a contract version and proves conformance in CI —
> coupled only by that version, never by a git merge. See `docs/stack/design-decisions.md`
> (ADR-0003, ADR-0004). *(This content currently lives in the `ai-neuro-os` repo, pending
> migration to `ai-neuro-stack` as the canonical home.)*

> **This is a reference *architecture*, not a fixed product.** The invariant is the set of
> component **roles and the contracts between them**; the C1–C8 components below are **one
> reference implementation**. An adopter should be able to replace any component — or the
> whole set — with a completely different one and have the architecture remain the same and
> functional, as long as the contracts are honored. See `docs/stack/design-decisions.md`
> (ADR-0001, ADR-0002).

## The stack

`ai-neuro-os` is composed of components that all share the same rules:

1. **No low-level code** — every concrete capability (imaging analysis, DICOM/FHIR
   access, registration, terminology, pathology, embeddings) is provided by an
   **MCP server** the OS calls.
2. **Layered** — within a component, each layer depends only on the layer directly
   below it (capabilities → adapters → domain model → graph → correlation → navigation).
3. **Domain + aggregate code only** — components hold domain-specific logic and
   aggregation/orchestration logic, nothing lower.

The stack is **knowledge-first**: **Connectome** (C1) is the center — the shared
knowledge graph every other component reads from and writes to — with **Perception**,
**Recall**, **Reasoner**, **Pathways**, **Conductor**, **Console** and **Sentinel**
building outward from it. See **[`docs/stack/component-map.md`](docs/stack/component-map.md)**
for the full map and dependency diagram.

## Component 1 — Connectome

**Connectome** is the first and foundational component: a **cross-modal knowledge
graph** for neurology. It ingests and normalizes multimodal data — **histology,
ultrasound, MRI, CT** plus **patient history** — into one navigable graph, then:

- **correlates** a finding seen in one modality with the same entity in another,
- powers **discovery** of similar findings/cases via embeddings, and
- supports **navigation** across **diagnosis → treatment plan → progression**.

Full spec: **[`docs/components/connectome/`](docs/components/connectome/)**
(read `00-overview.md` first). Worked example: **[`samples/connectome/`](samples/connectome/)**.

## How to read this repo

| Start here | Then |
|---|---|
| `docs/stack/component-map.md` — the whole stack | `docs/components/connectome/00-overview.md` |
| `docs/components/connectome/01-layered-architecture.md` — the L1–L6 model | `02 → 09` bottom-up |
| `samples/connectome/patient-example.md` — see it end-to-end | `samples/connectome/correlation-graph.md` |

## Status

Design only. No code, no database, no API. Sample JSON is illustrative, not a
runnable program. Implementation is a later phase.
