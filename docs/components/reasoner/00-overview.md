# Reasoner (C4) — Overview *(stub — to brainstorm)*

> **Status:** placeholder. Named and scoped; not yet specified as a full L1–L6 component.

## Role

**Reasoner** is the **cognition** component: it reasons over the Connectome graph to
generate a **differential diagnosis**, apply formal **criteria** (McDonald for MS, RANO
for neuro-onc response, WHO CNS classification, etc.), and surface the **evidence** that
supports or refutes each candidate. It reads the graph and similar cases; it does not
process images.

## Depends on

- **C1 Connectome** — the findings, lesions and history it reasons over.
- **C3 Recall** — similar prior cases / precedent for analogical reasoning.
- MCP servers (L1): **terminology**, and an **LLM / reasoning** capability.

## Key MCP servers

| Server | Use |
|---|---|
| Terminology / ontology | normalize & relate diagnostic concepts |
| LLM / reasoning | hypothesis generation, criteria application, explanation |

## Open questions (for the brainstorm)

- How criteria sets are encoded (rules vs. LLM-applied vs. hybrid) and kept auditable.
- Representing uncertainty: ranked differential with probabilities + evidence links.
- Writing conclusions back as `Diagnosis` nodes with provenance vs. proposing them as
  candidates for clinician confirmation.

When promoted, this folder grows the same `01`–`09` layer docs as Connectome.
