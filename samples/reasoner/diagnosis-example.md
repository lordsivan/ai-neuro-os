# Worked Example — Reasoner producing a candidate diagnosis

> **Illustrative only.** Not real, not runnable. Shows Reasoner reasoning over the
> Connectome case's left-frontal lesion **before** histology is back: it builds a 3-way
> differential, applies criteria, recommends biopsy as the next test, and proposes a
> **candidate** `dx-001`. Data: [`diagnosis-example.json`](diagnosis-example.json). Ties
> to [`../connectome/patient-example.md`](../connectome/patient-example.md) and
> [`../recall/discovery-example.md`](../recall/discovery-example.md).

## Setup — the moment of reasoning

It is **Jan 3** — the MRI (`find-mri-001`) and CT (`find-ct-001`) are in Connectome; the
intra-op US and histology have **not** happened yet. Reasoner is asked to diagnose the
left-frontal lesion.

## L2 — gather evidence

| Source | Evidence gathered |
|---|---|
| Connectome | `find-mri-001` (ring-enhancing, FLAIR-hyperintense, rim diffusion restriction), `find-ct-001` (hypodense, mass effect), history (headaches + R-arm weakness, 3 wks) |
| Recall | `discoverSimilarCases(pat-001)` → `pat-417` (high-grade glioma), `pat-512` (tumefactive demyelination — a look-alike) |
| Criteria KB | candidate sets: WHO CNS pathway, McDonald (because of the demyelination look-alike) |

## L4 — differential generation (grounded)

Three candidates survive the grounding gate, each citing evidence:

| Candidate | Supporting | Refuting |
|---|---|---|
| **High-grade glioma** | ring enhancement, rim restriction, mass effect, edema; precedent pat-417 | — |
| **Tumefactive demyelination** | ring (often *open-ring*) enhancement; precedent pat-512 | less mass effect than expected |
| **Cerebral abscess** | ring enhancement, **central** diffusion restriction | restriction is *rim*, not central; no infection history |

## L5 — criteria + confidence + next test

- **Criteria:** WHO CNS tumor **type/grade cannot be met without tissue** → `indeterminate`
  (needs histology). McDonald DIS/DIT **not-met** (single lesion, no dissemination).
- **Calibrated confidence (ranked):**

  | Candidate | Confidence |
  |---|---|
  | High-grade glioma | 0.62 |
  | Tumefactive demyelination | 0.23 |
  | Cerebral abscess | 0.15 |

- **Next discriminating test:** **biopsy / histology** — highest expected info gain: it
  resolves the `indeterminate` WHO CNS criteria and cleanly separates all three candidates
  (neoplastic vs. demyelinating vs. infective).

## L6 — proposal & handoff

Reasoner proposes a **candidate** `Diagnosis` and hands it to Connectome:

```
DiagnosisProposal{
  top: dx-001  "High-grade glioma" (candidate, confidence 0.62)
  differential: [glioma 0.62, demyelination 0.23, abscess 0.15]
  nextTest: biopsy/histology
  status: candidate
}  ──▶ Connectome adapter ──▶ Diagnosis dx-001 (status=candidate) ─supports─ Lesion les-001
```

A clinician sees the full differential + evidence + "get a biopsy" — and the lesion goes to
surgery.

## Closing the loop (Jan 12)

The biopsy returns: histology `find-histo-001` = grade 4 glial neoplasm. On
`updateOnNewEvidence`, Reasoner's WHO CNS criteria now read **met**, glioma's confidence
jumps, and the histology finding **`confirmed_by`** `dx-001` — which a clinician confirms
to `status: confirmed`. This is exactly the confirmed `dx-001` in the Connectome case.

## What Reasoner did vs. didn't

| Reasoner did | Reasoner did **not** |
|---|---|
| build a grounded differential | persist anything (Connectome did) |
| apply WHO CNS / McDonald deterministically | confirm the diagnosis (the clinician did) |
| recommend the biopsy (next test) | order or plan it (Pathways/Console) |
| propose candidate dx-001 with calibrated confidence | claim certainty before tissue |
