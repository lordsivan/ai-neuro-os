# app/ — Implementation (future repo `ai-neuro-ba-app`)

> **Folder-separated, not yet a repo.** This `app/` folder is the **implementation** —
> destined to become the **`ai-neuro-stack/ai-neuro-ba-app`** repo. It is kept here, under
> its own top-level folder, only because git operations to split repos aren't available yet
> (see `../REPO-LAYOUT.md`). Treat the folder boundary as the repo boundary.

## What this is

The **Phase-1, BA-facing functional prototype of the end-user application** — the genuine
Console (C7) clinician experience (ask / navigate / explain / confirm / notify) on **mock +
realistic data**, built so a **business analyst understands the product surface and the
requirements by *using* it**. *Not* a clinical product; *not* a doc/requirements browser.
(Full intent + honest limits: `../docs/roadmap/implementation-phases.md`.)

## How it must relate to the design (the rules it lives by)

- **Conforms to the design via a pinned contract.** It depends on a **pinned version** of
  the root `../contracts/` (the seam — ADR-0003). It does **not** reach into `../docs/` or
  `../samples/`.
- **One-way dependency.** Nothing in the design (repo root) imports anything from `app/`.
- **Layering enforced by tooling.** When code lands here: one module per component, each
  internally structured **L1–L6**, with a **boundary/dependency linter** in CI failing the
  build on any violation (L(n) imports only L(n−1); only L2 touches MCP clients; only
  Connectome writes the graph; cross-component only via L6).
- **"100% replica" = four CI gates green** for the pinned contract version: contract
  conformance, layering conformance, invariant conformance, worked-case parity
  (reproduces the `pat-001…` case against the design's `../samples/` golden file).

## Status

**Placeholder.** No application code yet — this establishes the separated home and the
conformance rules. Implementation begins once the contract artifact (`../contracts/`) has a
first tagged version to pin and the app repo/tooling is set up.

## Intended structure (when code lands)

```
app/
  contracts.lock            # the pinned @ai-neuro-stack/contracts version
  src/
    connectome/  l1.. l6..  # one module per component, internally L1–L6
    perception/  ...
    recall/      ...
    reasoner/    ...
    pathways/    ...
    conductor/   ...
    console/     ...        # the Phase-1 mobile surface
    sentinel/    ...
  test/                     # conformance + worked-case parity + unit/integration
  ci/                       # boundary linter + the four conformance gates
```
