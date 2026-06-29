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

## ADR-0003 — A runnable mock demonstrator (mobile-first), data-rich, capability-showcasing

**Status:** accepted (planned next deliverable) · **Scope:** the first runnable artifact

### Decision

Alongside the design docs, produce a **runnable mock implementation** that showcases the
**full feature capability** end-to-end, with:

- **Mock + realistic data:** high-quality hand-crafted mock data, supplemented by
  **realistic public data pulled from the internet** (open imaging/clinical datasets).
- **Mock components behind the real contracts:** each component (and its MCP capabilities)
  is a **stub/mock that conforms to the same role + interface** the design specifies — so
  the demonstrator doubles as the **first conformant reference implementation** (ADR-0002)
  and as a way to *test that the contracts are precise enough to build against*.
- **UI: mobile-first.** The Console (C7) surface — ask / navigate / explain / confirm /
  notify — rendered mobile-first.
- **UI roadmap:** mobile-first → web app → **chat-based / agentic** application. The
  interaction surface evolves; the architecture beneath does not.

### Why (the rationale)

- **Makes the seams concrete.** A reference architecture's value is its contracts; a
  runnable mock that wires mock components through those contracts is the cheapest way to
  prove the seams are real and complete (closes the "narrative contracts" gap in ADR-0002
  without waiting for real ML/clinical components).
- **Showcases capability without owning it.** It demonstrates the *experience and the
  flow* (cross-modal navigation, candidate→confirmed, discovery, monitoring) using mock
  cognition, so stakeholders can see the whole loop before any heavyweight component exists.
- **Mobile-first matches the Console surface.** The clinician interactions Console owns
  (natural-language ask, navigate, confirm a candidate, get notified) are a good fit for
  mobile; the heavyweight image-reading is delegated, not reimplemented in the demo.

### Consequences

- The demonstrator should **reuse the existing worked-case IDs** (`pat-001`, `les-001`,
  `dx-001`, `tx-001`, `prog-001/002`, cohort `pat-417/512`) so docs and demo stay aligned.
- Public data must carry **dataset provenance** ("public dataset, not a real patient") and
  respect dataset **licenses/usage terms**; the mock's `asserted_by`/provenance fields make
  this natural to record.
- Building the mock will surface under-specified contracts — that feedback should flow
  **back into the component specs** (the demo is also a spec test).

### What this ADR does **not** settle (honest limits — read before demoing)

- **A mock that showcases "full capability" is not evidence the capability works.** Like the
  worked case, a polished demo with **pre-baked findings/diagnoses/confidences** proves the
  *representation, flow, and UX* — not that any model is accurate. It must be **labeled as a
  capability/UX showcase with mock cognition**, or it will manufacture false confidence in
  stakeholders. The dangerous failure mode is a convincing mobile demo being mistaken for a
  working clinical product.
- **Mobile-first ≠ diagnostic image reading.** The mock UI should scope to the
  ask/navigate/confirm/notify surface; full multi-pane image review is not a mobile-first
  problem and is out of scope for the demonstrator.
- **Realistic public data ≠ validation.** Pulling real images in raises the *visual*
  realism but the cognition is still mock; it does not constitute clinical validation, and
  real public images paired with fabricated findings can mislead if not clearly marked.

### Implication for reviewers

Judge the demonstrator on whether it (a) exercises **every inter-component contract** with
conformant mock components (making it a real conformance test of the architecture), and
(b) is **unambiguously labeled** as mock-cognition. Do **not** read feature-completeness in
the demo as capability-completeness in the system.
