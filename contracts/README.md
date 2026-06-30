# contracts/ — the versioned seam between design and implementation

**Design-owned** (lives in the design SoT / repo root). This is the **machine-checkable
formalization of the design's seams** — the artifact the implementation
(`ai-neuro-ba-app`) **pins by version** and **proves conformance to** in CI (ADR-0002,
ADR-0003). It turns the prose specs into something an independent team can build against.

> **Status: v0.0.1 — draft starter.** Only a fragment of the domain model is formalized so
> far (`domain/shared.schema.json`). This is the beginning of the contract, not the whole.

## What this will hold

| Area | Source of truth in the design | Form |
|---|---|---|
| **Domain model** | `docs/components/connectome/04-domain-model.md`, `09-data-model-reference.md` | JSON Schema (entities, edges, provenance, enums) |
| **Component L6 interfaces** | each `docs/components/<c>/07-*.md` | typed interface (JSON Schema / OpenAPI) |
| **MCP capability contracts** | each `docs/components/<c>/02-*.md` | in/out schemas |
| **Invariants** | single-writer, provenance-on-write, candidate→confirmed, L(n)→L(n−1) | assertion specs |
| **Golden worked case** | `samples/` (`pat-001…`) | golden fixtures for worked-case parity |

## Versioning (semver)

- **patch** = clarification, no shape change · **minor** = additive / back-compatible ·
  **major** = breaking.
- Each release is **tagged**; the implementation pins `@ai-neuro-stack/contracts@X.Y.Z`.
- `VERSION` holds the current version.

## Rules

- Changes here are **design changes** — made on the design side, never to satisfy code.
- The implementation **consumes** these; it never edits them (it files change requests).
- Every contract element should trace back to the design doc it formalizes (and vice-versa),
  so **contract coverage = replica completeness**.
