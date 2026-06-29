# Console — Overview (C7)

**Console** is the **interaction** component of `ai-neuro-os`: the clinician-facing
agent / UI — the "shell" of the OS. It turns a natural-language question into the right
Connectome navigation journeys, **composes** the answers from across the stack into one
coherent view, **explains** the evidence (provenance + confidence), drafts reports, and is
**the place a human confirms a candidate**.

Like every component it follows the stack rules: **no low-level code** (NL understanding /
generation and rendering run in **MCP servers**), **layered** (each layer depends only on
the one below), **domain + aggregate code only**.

## Why this exists

Connectome (C1) holds the graph; Reasoner (C4) proposes a diagnosis; Pathways (C5) proposes
a plan and monitors response. None of them face the clinician. Someone must take a plain
question — *"what's going on with this lesion?"* — turn it into the cross-modal journeys
that answer it, lay the answer out so confirmed fact is visibly separated from machine
candidate, explain *why* each assertion is there, and let the clinician **own the decision**
by confirming or rejecting. Console does this, once, as the shared shell, so every other
component can stay headless.

## The shell, not the brain

Console computes almost nothing itself. It is an **orchestrating, presenting** layer:

- **Understanding** the question and **wording** the answer are an **LLM/NLP MCP server**.
- **Rendering** views and **formatting** reports are a rendering MCP capability.
- The **facts** come from sibling components (Connectome navigation, Reasoner differential,
  Pathways plan + timeline, Recall discovery), gathered at L2, usually routed via
  **Conductor** (C6).

Console's own code is intent→request mapping, result→view-model mapping, view composition,
trust application, and the confirmation handoff.

## Boundaries (compose / explain / confirm — then route back)

> **Console composes and explains; the clinician confirms; the owning component promotes;
> Connectome persists.**

- Console **never** writes the graph directly and **never** invents content. It reads via
  Connectome/Reasoner/Pathways/Recall and presents.
- It is **THE place a clinician confirms a candidate** — a `Diagnosis` from Reasoner
  (`docs/components/reasoner/07-proposal-and-handoff.md`), a `TreatmentPlan` from Pathways
  (`docs/components/pathways/07-proposal-and-monitoring.md`), or a
  correlation / discovery from Connectome / Recall
  (`docs/components/connectome/07-navigation-and-queries.md`).
- A `confirmCandidate` action routes the **candidate→confirmed promotion back to the owning
  component / Connectome**, governed by **Sentinel** (C8). Console requests the promotion;
  it does not perform the write.

```
NL question ──▶ CONSOLE (ask)    ──▶ composed view (confirmed vs candidate, explained)
clinician  ──▶ CONSOLE (confirm) ──▶ promotion request ─▶ owner/Connectome ─▶ confirmed
clinician  ──▶ CONSOLE (report)  ──▶ grounded ReportDraft (cited to the graph)
```

## What it is (and is not)

| Console **is** | Console **is not** |
|---|---|
| The clinician-facing agent / UI ("shell") | A diagnostic / planning engine (Reasoner C4 / Pathways C5) |
| NL intent → navigation → composed view | The knowledge graph or its writer (Connectome C1) |
| An explainer of evidence & provenance | The confirmer of record — a **human** confirms; Console relays |
| The human-confirmation entry point | An MCP server (LLM/rendering are external capabilities) |
| A caller of sibling components (via Conductor) | A re-implementation of their logic |

## Scope boundary with neighbours

| Console **does** | Console **does not** |
|---|---|
| parse NL questions, navigate, compose, explain | diagnose (Reasoner) or plan/monitor (Pathways) |
| distinguish confirmed vs candidate, apply trust profile | persist to the graph (Connectome) |
| relay the clinician's **confirm** to the owner | decide *for* the clinician |
| draft grounded reports | author free, ungrounded prose |
| route multi-step requests via Conductor | schedule / run the agent loop itself (Conductor C6) |

> Reasoner and Pathways *propose* candidates and stop. Console is where those proposals
> become **visible, explained, and confirmable**. The candidate→confirmed discipline used
> across the stack lands, for the human, here.

## Scope (this phase)

- **Inputs:** an NL question or navigation action + a chosen **trust profile**; sibling
  results (Connectome navigation, Reasoner differential, Pathways plan/timeline, Recall
  discovery); LLM/rendering MCP capabilities.
- **Output:** composed `ViewModel`s (confirmed vs candidate marked), grounded
  `Explanation`s, `ReportDraft`s, and `ConfirmationAction`s that route promotions back.
- **Deliverable:** design specification + a worked clinician-session sample. **No code.**

## Design principles

1. **No low-level code** — NL understanding/generation and rendering are MCP servers;
   Console orchestrates and presents.
2. **Strict layering** — L1→L6, each depends only on the one below (`01`).
3. **Read, never write directly** — Console reads via siblings and routes confirmations;
   Connectome is the single writer.
4. **Confirmed ≠ candidate, always** — every view visually distinguishes them and applies
   the trust profile (`06`).
5. **Grounded by construction** — every explanation and report element cites graph evidence;
   nothing is asserted that isn't backed.
6. **Human-confirmed** — Console relays the clinician's confirmation to the owning
   component; it never auto-confirms.

## Requirements traceability

| Requirement | Where |
|---|---|
| NL query → intent → request | `02` (LLM MCP), `03` (intent adapter) |
| Navigation (Connectome journeys) | `03`, `05` |
| Multi-component view composition | `05-view-composition.md` |
| Confirmed vs candidate + trust profile | `06-explanation-and-trust.md`, `08` |
| Grounded explanation generation | `06` |
| Report drafting | `06`, `08-presentation-and-reports.md` |
| Human confirmation → promotion routing | `07-interaction-api.md` |
| Entry points (`ask`/`navigate`/`confirmCandidate`/`draftReport`) | `07` |

## Reading order

`01` layers → `02` capabilities → `03` adapters → `04` domain model →
`05` view composition → `06` explanation & trust → `07` interaction API →
`08` presentation & reports → `09` reference. Then the worked example in `samples/console/`.
