# ai-neuro-os

> An AI operating system for **neuro-imaging & neurology** — built as a set of
> cooperating **components**, each one a domain/orchestration module that calls
> **MCP servers** for all heavy lifting. The OS itself implements **no low-level
> code**: no image processing, no DICOM parsing internals, no model inference. It
> *orchestrates* capabilities and *represents knowledge*.

This repository is currently in an **architecture & design phase**. It contains
design specifications and illustrative sample data — **no application code yet**.

## The stack

`ai-neuro-os` is composed of components that all share the same rules:

1. **No low-level code** — every concrete capability (imaging analysis, DICOM/FHIR
   access, registration, terminology, pathology, embeddings) is provided by an
   **MCP server** the OS calls.
2. **Layered** — within a component, each layer depends only on the layer directly
   below it (capabilities → adapters → domain model → graph → correlation → navigation).
3. **Domain + aggregate code only** — components hold domain-specific logic and
   aggregation/orchestration logic, nothing lower.

See **[`docs/stack/component-map.md`](docs/stack/component-map.md)** for the full
component map.

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
