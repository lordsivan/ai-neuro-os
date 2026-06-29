# ai-neuro-os — Design Decisions (ADRs)

Architecture Decision Records: the *why* behind the stack's load-bearing choices.
Each records the decision, the reasoning, the consequences, and what it does **not**
settle — so the rationale survives even when the prose elsewhere only states the *what*.

> **Scope & repos.** The **design / architecture source of truth** is the **`ai-neuro-stack`**
> repo (architecture & design only; owns the versioned `contracts/`). The **implementation**
> is the **`ai-neuro-ba-app`** repo (the Phase-1 BA-facing app, then beyond). The two are
> coupled only by a versioned contract, never by a git merge (ADR-0003, ADR-0004).
> Implementation/delivery planning is kept in `docs/roadmap/implementation-phases.md` so it
> does not distract from the design.
> *(Migration note: this design currently lives in the `ai-neuro-os` repo and is to be hosted
> as the canonical source in `ai-neuro-stack`.)*

> **Two meanings of "layer" — do not conflate.**
> - **Domain layers (L1–L6):** the stack *inside each component* (Capability → Adapters →
>   Domain Model → Aggregate → Correlation/L5 → Navigation/L6). See any component's `01`.
> - **Software-lifecycle (SDLC) layers:** the stack of *work products* — **Architecture →
>   Design → Implementation → Test** — where each conforms to the one above. Governed by
>   **ADR-0004**.
> Both obey the same "each layer depends only on / conforms only to the layer above it"
> discipline, but at different scales. When this repo says "the layering rule," context
> says which.

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

## ADR-0003 — Design ⇄ implementation: separation, versioned contracts, conformance enforcement

**Status:** accepted · **Scope:** how the design (this branch) and any implementation relate

### Decision

Design and implementation live in **separate branches/repos that never cross-merge**. The
**design is the single source of truth**; an implementation **depends on a pinned, versioned
contract** published from the design and **proves conformance in CI**. Change flows **one
way — design → implementation**. The two evolve on independent cadences, coupled only by a
**version number**, while the implementation is guaranteed to be a faithful,
layering-respecting replica of the design it pins.

### How it works

1. **Topology — two repos, version the seam, don't merge.** **Decided: two repositories.**
   - **`ai-neuro-stack`** — **design source of truth**; owns the architecture, the component
     specs, and the **`contracts/`** artifact, which it **semver-tags** on release.
   - **`ai-neuro-ba-app`** — the **implementation** (Phase-1 BA-facing app, then beyond).
   The implementation **pins** a contract version (a published package
   `@ai-neuro/contracts@X.Y.Z`, or a git submodule pinned to a release tag as the low-infra
   fallback). The only coupling is that **version pin** — there is **no git merge** between
   the repos, and the impl repo never writes back into the design repo (change flows as
   issues/PRs against the design — rule 6).
2. **Formalize the seams (this is what makes "100% replica" checkable).** Extract from the
   design, into the design branch, machine-checkable contracts: each component's **L6 public
   API** (OpenAPI/JSON-Schema/protobuf/types), the **domain model** (entities, edges,
   provenance attrs, `candidate/confirmed` enums), the **MCP capability** in/out, and the
   **invariants** (single-writer, provenance-on-every-write, candidate-until-confirmed, the
   L(n)→L(n−1) dependency rule). These *are* the design, formalized — and they close the
   "seams are only prose" gap (ADR-0002).
3. **Enforce layering by tooling, not discipline.** The implementation mirrors the
   architecture structurally (one module per component, each split L1–L6); a
   **boundary/dependency linter runs in CI and fails the build** on any violation (e.g.
   `import-linter`, `dependency-cruiser`, `ArchUnit`, `depguard`). Rules: *L(n) imports only
   L(n−1)*; *only L2 touches the MCP client*; *only Connectome writes the graph*;
   *cross-component imports hit only the sibling's L6*.
4. **Define "100% replica" as four CI gates** the implementation must pass to claim
   *conformant to design vX.Y.Z*: **(a) contract conformance** (APIs validate against the
   pinned schemas), **(b) layering conformance** (boundary linter green), **(c) invariant
   conformance** (runtime assertions/tests), **(d) worked-case parity** (reproduces the
   canonical `pat-001…` case end-to-end, output graph matches the design's `samples/` golden
   file).
5. **Independent evolution.** Design releases semver contract versions (patch = clarify,
   minor = additive, major = breaking). The impl stays pinned and upgrades **deliberately**,
   re-running the four gates. A scheduled **drift report** flags how many versions the impl
   is behind — drift is surfaced, never silent.
6. **One-way change flow.** The implementation never edits the design to fit code; it files a
   **change request / proposed ADR** against the design, which re-publishes a new contract
   version the impl then pins.
7. **Traceability.** Each impl module back-references the design element it implements
   (`implements: docs/components/<c>/<doc> @ contracts X.Y.Z`); **contract coverage =
   replica completeness**, and is reportable.

### Consequences

- "Faithful replica" and "independent evolution" stop being in tension: the **version pin**
  reconciles them — the impl is always a 100%-conformant replica of *the version it pins*,
  and the design can move ahead freely.
- It forces the design's seams to become **precise and testable** (the ADR-0002 critical
  path) — because they are now the contract an implementation is checked against.

### What this ADR does **not** settle

- **The contracts must actually be authored.** Today the seams are prose; until the
  `contracts` artifact exists and is versioned, conformance is aspirational. Producing it is
  the prerequisite for any two-branch implementation work.
- **Conformance ≠ correctness.** Passing the four gates proves the impl matches the *design*;
  it does not prove the design (or any model) is clinically correct — that is validation,
  separate and downstream.

---

## ADR-0004 — Software-lifecycle layering: Architecture → Design → Implementation → Test

**Status:** accepted · **Scope:** how the work products (artifacts) relate across the SDLC
**Note:** this is the **SDLC/artifact** layering, **distinct from the domain L1–L6 layering**
inside a component (see the disambiguation at the top).

### Decision

Govern the work products as a **strict vertical stack of lifecycle layers — Architecture,
Design, Implementation, Test** — where **each layer derives from and conforms to the layer
above it**, may **evolve on its own version**, and feeds change **upward as requests** (never
silent divergence). The same dependency discipline as the domain L(n)→L(n−1) rule, applied
at lifecycle scale.

### The stack

| # | SDLC layer | Artifact (where) | Conforms **up** to | Verified **down** by | Repo |
|---|---|---|---|---|---|
| 1 | **Architecture** | invariants, roles, ADRs, contracts (`docs/stack/`, `contracts/`) | — (top) | design review / traceability | `ai-neuro-stack` |
| 2 | **Design** | component L1–L6 specs, domain model, interface specs, samples (`docs/components/`, `samples/`) | Architecture | impl conformance gates | `ai-neuro-stack` |
| 3 | **Implementation** | prototype code (module-per-component, internally L1–L6) | Design (a **pinned** contract version) | Test | `ai-neuro-ba-app` |
| 4 | **Test** | conformance + worked-case parity + unit/integration | Design & Architecture | CI green | `ai-neuro-ba-app` |

### Rules

1. **Downward derivation, upward feedback.** Nothing appears in a lower layer that isn't
   justified by the layer above. Lower-layer discoveries become **change requests** to the
   layer above — never local edits that diverge.
2. **No skipping.** Implementation conforms to **Design**; Design conforms to
   **Architecture**. You don't justify code straight from "architecture intent," bypassing
   Design.
3. **Per-layer versioning + pinning.** Architecture `vA`; Design `vD` (conforms to `vA`);
   Implementation **pins** `vD`; Test targets `vD`. Each layer revs on its own clock;
   propagation is top-down and deliberate.
4. **Conformance gate between every adjacent pair.** Architecture→Design = review /
   traceability. **Design→Implementation = the four CI gates of ADR-0003** (contract,
   layering, invariant, worked-case parity). Implementation→Test = tests must cover the
   pinned contract.
5. **Repo mapping (two repos).** The **`ai-neuro-stack` repo** carries layers 1–2
   (Architecture + Design + the `contracts/` artifact); the **`ai-neuro-ba-app` repo**
   carries layers 3–4 (Implementation + Test). The **seam between the repos is the versioned
   contract** — exactly the Design↔Implementation boundary (mechanism in ADR-0003). No git
   merge crosses the repo boundary; the impl repo consumes a **pinned** contract version.

### Where the two requirements land

- **"100% replica, doesn't violate the design"** = the **Design→Implementation gate is
  green** for the pinned contract version.
- **"Architecture evolves independently of implementation"** = **per-layer versioning**;
  Implementation only sees a new Design when it chooses to **bump its pin**, then re-runs the
  gates.

### What this ADR does **not** settle

- It requires the **contracts artifact** (ADR-0003) to exist; the Architecture↔Design and
  Design↔Implementation gates are only as strong as the formalized contracts.
- **Cross-layer conformance proves fidelity, not clinical correctness.** Validation
  (is the design *right*?) sits **beside** the Test layer as a separate concern, not above it.

---

## Out of scope on this branch — implementation & delivery

Delivery planning — the **Phase-1 mobile end-user-app prototype** (built for BA
understanding), the **mobile → web → chat/agentic** UI roadmap, and **Phase-2 clinical
rollout** — is intentionally **kept out of the design** and lives in
**`docs/roadmap/implementation-phases.md`**. It is not part of the architecture
specification and the design does not depend on it. ADR-0003 above governs *how* such an
implementation must relate to (and conform to) this design; the implementation's own plans
live in the roadmap.
