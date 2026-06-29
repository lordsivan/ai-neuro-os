# Reasoner — Overview (C4)

**Reasoner** is the **cognition** component of `ai-neuro-os`. It reads the Connectome
graph (and Recall's precedent) to produce a **differential diagnosis**, apply formal
**criteria**, surface the **evidence**, and recommend the **next discriminating test** —
then proposes a **candidate** `Diagnosis` that a clinician confirms.

Like every component it follows the stack rules: **no low-level code** (the LLM and
knowledge bases run in **MCP servers**), **layered** (each layer depends only on the one
below), **domain + aggregate code only**.

## Why this exists

Connectome *records* findings and Recall *finds look-alikes*, but neither **decides what
the patient has**. Diagnosis is reasoning: weigh the findings, the history, the precedent,
and the formal criteria; rank the possibilities; say what evidence is missing. Reasoner is
where that happens — once, as a shared, auditable service, instead of buried in a UI.

## How it reasons (hybrid)

> **Deterministic where it must be, flexible where it helps.**

- **Formal criteria** (McDonald for MS, RANO for neuro-onc response, WHO CNS, mRS) are
  applied as **deterministic rule-checklists** — each criterion is met / not-met /
  indeterminate against explicit graph evidence, so the conclusion is auditable.
- **Differential generation & explanation** use an **LLM** (MCP) to propose and articulate
  candidates — but every candidate is **grounded in graph evidence** (findings, history),
  never free-floating.

The two meet in the differential: the LLM proposes, the rules adjudicate, the evidence
anchors.

## The output boundary (propose, don't persist)

> **Reasoner proposes; Connectome persists; a human confirms.**

Reasoner produces a **candidate `Diagnosis`** (plus the differential, the evidence trail,
and the recommended next test) and hands it to Connectome's adapter flagged **candidate**.
A clinician (via **Console**, C7) confirms before it becomes a confirmed `Diagnosis`. This
mirrors the produce/persist + candidate discipline of Perception (C2) and Recall (C3) —
fitting, because diagnosis is the highest-stakes assertion in the stack.

```
Connectome graph + Recall precedent ──▶ REASONER (differential → criteria → next-test)
                                              │  candidate Diagnosis + evidence
                                  Connectome adapter (flagged candidate) ──▶ graph
                                              │
                                    human confirms (Console) ──▶ confirmed Diagnosis
```

## Scope: diagnosis + next discriminating test

In scope: differential generation, criteria application, evidence surfacing, uncertainty,
and **"what test would best confirm/refute"** (diagnostic workup guidance). **Out of
scope:** treatment planning and monitoring — those are **Pathways** (C5).

## What it is (and is not)

| Reasoner **is** | Reasoner **is not** |
|---|---|
| A diagnostic reasoning service | The LLM / knowledge base (those are MCP servers) |
| Hybrid rules + LLM over graph evidence | A graph store (that's Connectome) |
| A producer of **candidate** diagnoses | The confirmer (a human is) |
| Diagnosis + next-test guidance | A treatment planner (that's Pathways) |

## Scope (this phase)

- **Inputs:** Connectome graph (findings, lesion, history) + Recall precedent + criteria KB.
- **Output:** a candidate `Diagnosis` with a ranked differential, per-criterion evidence,
  calibrated confidence, and a recommended next discriminating test.
- **Deliverable:** design specification + a worked sample. **No code.**

## Design principles

1. **No low-level code** — LLM & knowledge bases are MCP servers; Reasoner orchestrates.
2. **Strict layering** — L1→L6, each depends only on the one below (`01`).
3. **Grounded** — every differential item cites graph evidence; no ungrounded claims.
4. **Auditable criteria** — formal criteria are deterministic checklists, not LLM guesses.
5. **Always candidate** — diagnoses are proposed, never self-confirmed.
6. **Calibrated uncertainty** — output a ranked differential with calibrated confidence,
   not a single overconfident answer.

## Requirements traceability

| Requirement | Where |
|---|---|
| Differential generation | `05-differential-generation.md` |
| Criteria application (McDonald/RANO/WHO CNS) | `06`, `08-criteria-catalog.md` |
| Evidence surfacing | `03` (gather), `05`/`06` (cite) |
| Next discriminating test | `06-criteria-and-confidence.md` |
| Precedent (similar cases) | `03` (calls Recall) |
| Propose candidate, human-confirmed | `07-proposal-and-handoff.md` |
| Grounded, calibrated, auditable | `05`, `06`, `09` |

## Reading order

`01` layers → `02` capabilities → `03` evidence & context → `04` domain model →
`05` differential → `06` criteria & confidence → `07` proposal & handoff →
`08` criteria catalog → `09` reference. Then the worked example in `samples/reasoner/`.
