# Repository layout — folder-separated now, two repos later

This is **one repository/branch today**, but it is structured so the **future two-repo
split** (under the GitHub org **`ai-neuro-stack`**) is a clean, mechanical extraction. We
use **top-level folders to separate** the two concerns now, because git operations to split
into real repos aren't available yet.

## Folder → future repo mapping

| Folder (now) | Contents | Future repo (under org `ai-neuro-stack`) | SDLC layers (ADR-0004) |
|---|---|---|---|
| **repo root** — `README.md`, `docs/`, `samples/`, `contracts/` | architecture, design, the versioned contract artifact | **`ai-neuro-os`** — design / architecture **source of truth** | 1–2 |
| **`app/`** | the implementation (Phase-1 BA app, then beyond) | **`ai-neuro-ba-app`** — implementation | 3–4 |

## The separation rules (same as the two-repo rules, enforced by folder)

- **One-way dependency.** `app/` may depend on `contracts/` (by pinned version); **nothing
  at the root depends on `app/`.** The design never imports the implementation.
- **The seam is the contract.** `app/` conforms to a **pinned version** of `contracts/`
  (see ADR-0003). It does not reach into `docs/` or `samples/` directly.
- **No design edits from the app side.** Changes the implementation needs are raised as
  issues/PRs against the design (the root), which re-publishes a new `contracts/` version.

## When git access is available — the split is mechanical

1. `ai-neuro-stack/ai-neuro-os` ← everything **except** `app/` (root design + `contracts/`).
2. `ai-neuro-stack/ai-neuro-ba-app` ← the **`app/`** folder (e.g. `git subtree split -P app`
   or `git filter-repo --path app/`), which then pins `@ai-neuro-stack/contracts`.

Until then, treat the folder boundary as if it were the repo boundary: **do not** add
imports or references that cross it except `app/ → contracts/` by version.

See `docs/stack/design-decisions.md` (ADR-0003 the seam, ADR-0004 the SDLC layering).
