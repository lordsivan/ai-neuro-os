# Implementation & Delivery Phases — *separate from the design*

> **Scope boundary.** This document is **delivery/implementation planning**, kept
> **separate from the architecture & design** that is the subject of this branch. Nothing
> here is part of the design specification. The design (`docs/components/`, `docs/stack/`,
> `samples/`) stands on its own and does not depend on any of this. Treat this as a parking
> place for downstream plans so they don't distract from the architecture work.
>
> Architecture rationale lives in `docs/stack/design-decisions.md` (ADR-0001, ADR-0002).
>
> **The implementation has its own repo:** **`ai-neuro-ba-app`** (the **`ai-neuro-stack`**
> repo is the design source of truth). The app repo **pins a versioned contract** published
> from the stack repo and proves conformance in CI — see ADR-0003 (the seam) and ADR-0004
> (the lifecycle layering / repo mapping). The notes below describe *what* Phase 1 builds;
> *how* it must conform lives in those ADRs.

## Phase 1 — Functional prototype of the end-user app, built for BA understanding

A runnable, **mobile-first** app that is **the real product experience** — the
clinician-facing Console (C7) journeys: ask / navigate the cross-modal lesion view / see
findings, diagnosis, plan, progression / discovery / confirm a candidate / get notified —
running on **mock + realistic data**.

- It is **NOT** a document navigator, a requirements/metrics dashboard, or a
  spec-visualization tool. It is the **end-user app itself**.
- **Audience & purpose:** the **business analyst**. BAs (and stakeholders) understand the
  product surface and the requirements — which live across the large spec corpus — **by
  *using the real application*,** not by reading documents. The app *is* the requirements
  made tangible.

**Phase-1 shape**
- Same UX as the eventual product (the genuine Console journeys/screens), not a meta layer.
- High-quality **mock data**, optionally supplemented by **realistic public data** for
  visual fidelity (carry dataset provenance; respect licenses).
- **Mock cognition behind the real contracts:** the AI/clinical results are mocked, but each
  component / MCP capability is a **stub conforming to its specified role + interface** — so
  building the app also **conformance-tests the inter-component contracts** (ADR-0002) and
  feeds under-specified seams back into the design docs.
- **UI: mobile-first**, scoped to the Console interaction surface (ask / navigate / explain /
  confirm / notify). Not diagnostic image reading.
- Reuse the worked-case IDs (`pat-001`, `les-001`, `dx-001`, `tx-001`, `prog-001/002`,
  cohort `pat-417/512`) so app and docs stay aligned.

**Honest limits**
- It is the real app experience, so it **will look like a working clinical product while the
  cognition is mock.** Keep it **internal / BA-facing** and **clearly labeled mock-data, not
  clinically validated** — it is a requirements vehicle, not evidence the AI works.
- Phase 1 validates the **surface**, not the **capability**. Real components, validation, and
  regulatory work are Phase 2.
- Realistic public data raises visual fidelity only; real images paired with mock findings
  must be marked as such.

## UI roadmap (beyond Phase 1)

**mobile-first → web → chat-based / agentic.** The interaction surface evolves; the
architecture beneath does not.

## Phase 2 — Clinical / rollout application

The app hardened for clinical use and deployment: real components, clinical validation,
regulatory pathway (SaMD/CE), PACS/EHR integration, safety/liability. A different product
with a different bar — out of scope for Phase 1 and not discharged by it.
