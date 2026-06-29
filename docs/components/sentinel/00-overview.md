# Sentinel (C8) — Overview *(stub — to brainstorm)*

> **Status:** placeholder. Named and scoped; not yet specified as a full L1–L6 component.

## Role

**Sentinel** is the **cross-cutting governance** component: it guards provenance, audit,
consent / PHI, access control, model & safety governance, and evaluation / drift
monitoring **across every other component**. It is not in the build-up chain — it sits
over the whole stack and enforces the rules the others must honor.

## Applies to

- **All components.** Most directly: the **provenance + confidence** stamped on every
  Connectome node/edge (`docs/components/connectome/04-domain-model.md` shared attributes)
  is a Sentinel-governed, stack-wide invariant.

## Responsibilities

| Concern | Notes |
|---|---|
| Provenance & audit | who/what asserted each fact, when, how — immutable trail |
| Consent / PHI | privacy boundaries, especially cross-patient discovery (with **Recall**) |
| Access control | who can read/write which data and run which journeys |
| Model governance | versioning, approval, eval / drift monitoring of MCP models |
| Safety | guardrails on automated assertions; human-confirmation policy |

## Open questions (for the brainstorm)

- Enforcement model: a service others call, a policy layer, or both.
- How candidate→confirmed promotion (`06`) is gated and logged.
- Audit storage and retention; alignment with healthcare compliance regimes.

When promoted, this folder grows the same `01`–`09` layer docs as Connectome.
