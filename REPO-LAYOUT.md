# Repository layout — two repos under `ai-neuro-stack`

The design and the implementation live in **two separate repos** under the GitHub org
**`ai-neuro-stack`**, coupled only by a **versioned contract** (never by a git merge).

## Repo → contents mapping

| Repo (under org `ai-neuro-stack`) | Contents | SDLC layers (ADR-0004) |
|---|---|---|
| **`ai-neuro-os`** (this repo) — `README.md`, `docs/`, `samples/`, `contracts/` | architecture, design, the versioned contract artifact — the design / architecture **source of truth** | 1–2 |
| **`ai-neuro-ba-app`** | the implementation (Phase-1 BA app, then beyond); the Phase-1 **mobile requirement** lives in its `requirements/` folder | 3–4 |

## The separation rules

- **One-way dependency.** The implementation may depend on `contracts/` (by pinned
  version); **nothing in this design repo depends on the implementation.** The design never
  imports the implementation.
- **The seam is the contract.** The implementation conforms to a **pinned version** of
  `contracts/` (see ADR-0003). It does not reach into `docs/` or `samples/` directly.
- **No design edits from the app side.** Changes the implementation needs are raised as
  issues/PRs against the design (this repo), which re-publishes a new `contracts/` version.

## History — the split has happened

The implementation originally lived under a top-level `app/` folder in this repo while git
operations to split repos weren't available. It has since been **moved to
`ai-neuro-stack/ai-neuro-ba-app`** (the Phase-1 mobile requirement now lives in that repo's
`requirements/` folder). This repo holds the design only.

See `docs/stack/design-decisions.md` (ADR-0003 the seam, ADR-0004 the SDLC layering).
