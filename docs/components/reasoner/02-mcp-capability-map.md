# Reasoner — MCP Capability Map (L1)

The capabilities Reasoner orchestrates. All are **external MCP servers** — Reasoner runs
no model and hosts no knowledge base. Only **L2** calls these. (Its other inputs —
Connectome and Recall — are **sibling components**, not L1; see `03`.)

## Capability catalog

| MCP server | Provides | Inputs | Outputs | Feeds |
|---|---|---|---|---|
| **LLM / reasoning** | hypothesis generation & explanation | reasoning context (evidence) | candidate diagnoses + rationale text | differential (L4) |
| **Criteria / guideline KB** | formal diagnostic criteria definitions | disease / scheme id | criterion sets (items, logic, thresholds) | criteria application (L5) |
| **Terminology / ontology** | code & relate clinical concepts | term / code | SNOMED CT / ICD-11 / WHO CNS codes + relations | grounding & coding |

> Terminology is **shared** with Connectome's & Perception's L1 catalogs — same server,
> reused. The LLM and criteria KB are Reasoner's distinctive capabilities.

## Per-capability notes

### LLM / reasoning — the hypothesis engine
Given the assembled reasoning context (findings, history, precedent), the LLM proposes a
**differential** and articulates **why** each candidate fits or doesn't. It is used for
the open-ended part of reasoning — generation and explanation — **not** for applying
formal criteria (that is deterministic, L5). Every LLM-proposed candidate must cite
evidence already in the context; ungrounded candidates are dropped at L4.

> The LLM is a tool, not the authority. Criteria adjudication and the final calibrated
> confidence come from deterministic logic; the LLM widens and explains the space.

### Criteria / guideline knowledge base — the rulebook
Serves machine-readable **criterion sets**: McDonald (MS dissemination in space/time),
RANO (neuro-onc response), WHO CNS (tumor classification), mRS (functional outcome), and
others (`08`). Each criterion is an explicit, checkable statement with its logic and
thresholds, so L5 can mark it met / not-met / indeterminate against graph evidence and
record exactly which evidence drove the result.

### Terminology / ontology — grounding
Normalizes the diagnoses, findings and criteria terms to shared codes so a candidate
("high-grade glioma") aligns with the finding evidence and with how Connectome will store
the resulting `Diagnosis` (`docs/components/connectome/04-domain-model.md`).

## Contract stability

Higher layers code against `DifferentialItem`/`Criterion`/`DiagnosisProposal` (`04`),
never these payloads. Swap the LLM vendor → only its L2 adapter and prompt templates
change; the criteria logic (L5) is unaffected because it is deterministic and KB-driven.
(LLM/model governance — approval, eval, drift, safety — is **Sentinel** (C8); scheduling
is **Conductor** (C6). Reasoner just calls.)
