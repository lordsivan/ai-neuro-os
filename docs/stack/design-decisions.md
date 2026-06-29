# ai-neuro-os — Design Decisions (ADRs)

Architecture Decision Records: the *why* behind the stack's load-bearing choices.
Each records the decision, the reasoning, the consequences, and what it does **not**
settle — so the rationale survives even when the prose elsewhere only states the *what*.

---

## ADR-0001 — Heavyweight capabilities live behind MCP boundaries

**Status:** accepted · **Scope:** stack-wide (the L1 capability layer of every component)

### Decision

Every concrete capability — image segmentation/detection/characterization, spatial
registration, multimodal embedding + vector search, terminology/ontology, pathology/WSI
analysis, the reasoning LLM, the policy/audit/consent engines — is consumed **only through
an MCP server contract**. `ai-neuro-os` components hold domain + aggregate/orchestration
code; they never embed a capability's implementation. This is the "no low-level code"
rule, and **MCP is the boundary that enforces it**.

### Why (the real rationale)

The MCP seam is **not** primarily a coding-style choice or a way to defer hard work. It is
an **organizational and lifecycle boundary**:

1. **Each capability is a heavyweight subsystem in its own right.** A segmentation model, a
   registration engine, a vector index, a terminology service, an LLM — each is a large,
   specialized system with its own stack, data, hardware, and release cadence. They do not
   belong *inside* an orchestration component; they are peers behind a contract.
2. **There are multiple viable alternative implementations of each.** Segmentation could be
   MONAI/nnU-Net or a vendor model; the vector store could be any of several engines; the
   LLM is interchangeable; terminology could be Ontoserver or another service. The contract
   must allow **swapping the implementation without touching the components above** — which
   is exactly what the L2-adapter-only boundary buys (see each component's `02`/`03`).
3. **They are built and owned by different teams.** Different specialists (imaging-ML,
   informatics/standards, infra/retrieval, governance/security) develop, evaluate, version,
   and operate these independently and in parallel. MCP gives each team a **stable public
   contract** to develop against without coordinating internals — Conway's law made
   deliberate: the system boundaries mirror the team boundaries.

So the MCP layer is a **modularity + ownership + swappability** decision: stable contracts
around independently-developed, replaceable, heavyweight subsystems.

### Consequences

- **Positive:** parallel, independent development by specialist teams; vendor/impl
  swappability with blast radius limited to one L2 adapter; the OS stays a thin
  domain/orchestration layer; capabilities can be evaluated/governed (Sentinel C8) per
  version without OS changes.
- **Cost / open risks (not settled by this ADR):**
  - **Transport fit** — MCP (request/response tool-call protocol) is *assumed* suitable for
    heavyweight payloads (DICOM volumes, whole-slide images) and for stateful services (a
    persistent vector index, long-running jobs). This has **not** been stress-tested; it may
    need streaming, async-job, or pass-by-reference patterns layered on top. Flagged for a
    future ADR.
  - **This decision justifies the *boundaries*, not the *contents*.** That each capability is
    a swappable MCP box does not make any given box buildable or accurate — the clinical/ML
    difficulty inside each contract is real and lives with the owning team.
  - **The inter-component orchestration/governance protocol is separate** (who routes, where
    Sentinel gates) and still owes its own spec — see the review notes / a future ADR.

### Implication for reviewers

The component decomposition and the "everything behind MCP" stance should be read as a
**team/ownership and swappability boundary**, not as the system claiming to have solved the
capabilities it delegates. Judge each MCP contract by whether it cleanly isolates a
heavyweight, separately-owned, replaceable subsystem — that is its job.

---

## ADR-0002 — This is a reference *architecture*; C1–C8 are one reference *implementation*

**Status:** accepted · **Scope:** the whole repo's intent

### Decision

`ai-neuro-os` ships **two layers of artifact, and the first is the product:**

1. **The reference architecture** (the invariant): the set of component **roles** and the
   **contracts between them** — the L1–L6 layering rule, the MCP capability boundary
   (ADR-0001), knowledge-first / graph-at-center, the single-writer rule, and the
   stack-wide **candidate→confirmed + provenance/confidence** discipline.
2. **A reference implementation** (one instantiation): the specific C1–C8 components
   (Connectome, Perception, Recall, Reasoner, Pathways, Conductor, Console, Sentinel) as
   specified here, with the worked case.

**An adopter must be able to replace any component — or the entire set — with a completely
different set of components, and have the overall architecture remain the same and
functional**, provided the replacements honor the same roles and contracts. The C1–C8 set
is illustrative and replaceable; the architecture is what's being standardized.

### Why (the rationale)

- **Portability over a single product.** The lasting value is a *pattern* an organization
  can adopt with its own building blocks, vendors, and teams — not a fixed application they
  must take wholesale. Different adopters will (and should) bring entirely different
  components.
- **It generalizes ADR-0001 up one level.** ADR-0001 makes each *capability* (inside a
  component) swappable behind MCP. ADR-0002 makes each *component* — and the whole set —
  swappable behind its **role + inter-component contract**. Same principle, two scales:
  swap an implementation, or swap the participant.
- **The reference implementation exists to prove the architecture is realizable and to
  give adopters a conformance target** — not to be the canonical product.

### Consequences

- **Conformance becomes the real spec.** For "swap any component and it still works" to be
  true, each component's **role and contract must be specified precisely enough that a
  replacement can be checked against it**: the role it plays, its layer obligations, what it
  reads/produces, how it stamps provenance, how it honors candidate/confirmed, and which MCP
  capabilities it depends on. A *conformance contract* per component is the artifact that
  makes ADR-0002 true rather than aspirational.
- **The seams are now load-bearing.** A reference architecture's entire value lives in the
  **precision of its interfaces**, not the richness of its components. This raises the
  priority of the (currently under-specified) **inter-component orchestration & governance
  protocol** — who routes, where Sentinel gates, the exact promotion path — from "nice to
  add" to **the critical path**: it is the thing a replacement component must conform to.

### What this ADR does **not** settle (honest limits)

- **Swap-ability is only as real as the contracts.** Today the inter-component contracts are
  largely **narrative prose**, not typed/with conformance tests. Until they are precise, the
  "drop in a different component set" promise is a goal, not a guarantee. This is the single
  most important gap to close for the reference-architecture claim to hold.
- **A reference architecture can standardize the wrong pattern.** Publishing it before any
  instantiation is *validated* risks ossifying an unproven design; adopters inherit both the
  good seams and any structural mistakes. The architecture itself still needs at least one
  end-to-end *validated* (not just hand-authored) instantiation before it should be treated
  as a standard.
- **"Same architecture, different components" assumes the role decomposition is right.** If
  the eight roles are themselves mis-cut, no amount of component-swapping fixes it; that
  decomposition is the part most worth pressure-testing.

### Implication for reviewers

Review this repo as a **reference architecture**: judge the **roles and the contracts
between components**, not the cleverness of any one component. The decisive questions are
(1) are the inter-component seams specified precisely enough that an independent team could
build a conformant replacement, and (2) is the role decomposition itself correct. The C1–C8
components should be treated as a worked example of the contracts — replaceable by design.

---

## ADR-0003 — Phase 1: a functional end-user app prototype, built for BA understanding; Phase 2: clinical rollout

**Status:** accepted (planned next deliverable) · **Scope:** the first runnable artifact and its purpose

### Decision — two phases, deliberately different purposes

- **Phase 1 — Functional prototype of the *actual end-user application* (this deliverable).**
  A runnable, **mobile-first** app that is **the real product experience** — the
  clinician-facing Console (C7) journeys: ask / navigate the cross-modal lesion view /
  see findings, diagnosis, plan, progression / discovery / confirm a candidate / get
  notified — running on **mock + realistic data**. It is **NOT a document navigator, a
  requirements/metrics dashboard, or a spec-visualization tool.** It is the end-user app
  itself. Its **audience and purpose in Phase 1 are the business analyst**: BAs (and
  stakeholders) come to **understand the product surface and the requirements — which live
  across the several-thousand-page spec corpus — by *using the real application*,** not by
  reading documents. The app *is* the requirements made tangible.
- **Phase 2 — Clinical/rollout application (later).** The same app hardened for clinical
  use and deployment (real components, validation, regulatory, integration). Out of scope
  for Phase 1.

### Phase-1 shape

- **Same UX as the eventual product:** the genuine Console end-user journeys and screens,
  not a meta/annotation layer over them.
- **Mock + realistic data:** high-quality hand-crafted mock data, optionally supplemented by
  **realistic public data** (open datasets) for visual fidelity.
- **Mock cognition behind the real contracts:** the AI/clinical results are mocked, but each
  component/MCP capability is a **stub conforming to its specified role + interface** — so
  building the app doubles as a **conformance test of the inter-component contracts**
  (ADR-0002) and feeds under-specified seams back into the specs.
- **UI: mobile-first**, scoped to the Console interaction surface (ask / navigate / explain /
  confirm / notify). Not diagnostic image reading.
- **UI roadmap (beyond Phase 1):** mobile-first → web → **chat-based / agentic**; the
  interaction surface evolves, the architecture beneath does not.

### Why (the rationale)

- **A working app communicates requirements better than a corpus.** BAs grasp scope, gaps,
  and intent by *operating the real surface* far faster and more reliably than by reading
  thousands of pages — and experiencing the product elicits/validates requirements that
  prose review misses. The app is a requirements **elicitation and validation** vehicle in
  the form of the genuine product.
- **It makes the architecture's seams concrete cheaply.** Mock components through the real
  contracts gives BAs a faithful product *and* proves the contracts are buildable — one
  artifact, two payoffs.
- **Mobile-first fits the Console interaction surface** (ask/navigate/confirm/notify); the
  heavyweight image reading is delegated, not reimplemented.

### Consequences

- **Fidelity to the specified product is the core property:** the prototype must faithfully
  realize the Console journeys and the worked case, so what a BA experiences *is* the
  specified surface.
- Reuse the existing worked-case IDs (`pat-001`, `les-001`, `dx-001`, `tx-001`,
  `prog-001/002`, cohort `pat-417/512`) so app and docs stay aligned.
- Building it will expose gaps/ambiguities in the specs; those flow **back into the docs**.
- Public data carries **dataset provenance** ("public dataset, not a real patient") and
  respects licenses.

### What this ADR does **not** settle (honest limits)

- **It is the real app experience, so it *will* look like a working clinical product —
  while the cognition is mock.** That persuasiveness is the point (for BA understanding) and
  the risk (false confidence). Mitigation: keep it **internal / BA-facing** and **clearly
  labeled mock-data, not clinically validated**; it is a requirements vehicle, not evidence
  the AI works.
- **Phase 1 validates the *surface*, not the *capability*.** Experiencing the journeys
  confirms what the product should do; it does not confirm any model is accurate. Real
  components, validation, and regulatory work are Phase 2 and are not discharged here.
- **Realistic public data ≠ validation;** it raises visual fidelity only, and real images
  paired with mock findings must be marked as such.

### Implication for reviewers

Judge the Phase-1 app as a **functional prototype of the end-user product used for BA
requirement understanding**: does operating it convey the real surface and requirements,
does it faithfully realize the specified Console journeys + worked case, and did building it
**exercise the inter-component contracts** (conformance signal)? It is the genuine app on
mock cognition — **not** a clinical demo and **not** evidence the capabilities work.
