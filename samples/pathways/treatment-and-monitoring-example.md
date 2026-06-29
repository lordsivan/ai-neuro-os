# Worked Example — Pathways planning + monitoring

> **Illustrative only.** Not real, not runnable. Shows Pathways' two tracks on the running
> case: plan treatment for the confirmed diagnosis, then monitor response over two
> follow-ups. Data: [`treatment-and-monitoring-example.json`](treatment-and-monitoring-example.json).
> Ties to [`../connectome/patient-example.md`](../connectome/patient-example.md) (`tx-001`,
> `prog-001`, `prog-002`) and [`../reasoner/diagnosis-example.md`](../reasoner/diagnosis-example.md)
> (`dx-001`).

## Planning track (Jan 12 — diagnosis confirmed)

Histology confirmed `dx-001` = **high-grade glioma (grade 4)**. Pathways plans.

### L2 — gather
- **Connectome:** `dx-001`, lesion `les-001`, patient factors (52F, good performance
  status), the imaging timeline.
- **Recall:** similar **treated** cases — `pat-417` (glioma) treated with resection +
  chemoradiation + adjuvant, progression at 4 mo.
- **Guideline KB:** high-grade glioma guideline → resection → concurrent chemoradiation →
  adjuvant systemic.
- **Trial registry:** one eligible trial for newly-diagnosed high-grade glioma.

### L4/L5 — options → selected plan
| Option | Source | Selected |
|---|---|---|
| Maximal safe resection | guideline (high grade) | ✅ first |
| Concurrent chemoradiation | guideline (high grade) | ✅ |
| Adjuvant systemic therapy | guideline + precedent pat-417 | ✅ |
| Clinical trial GBM-2026-A | trial (eligible) | offered as alternative |
| Surveillance MRI q-interval | guideline | ✅ (monitoring hook) |

Calibrated plan confidence: **0.78** (strong guideline grade; consistent precedent).

### L6 — propose, MDT confirms
Pathways hands **candidate** `tx-001` (resection → chemoradiation → adjuvant, + trial
option) to Connectome (`treated_by` `dx-001`). The **tumor board** reviews the sequence,
the trial match, and the alternative, and **confirms** → `tx-001` becomes `active`. This is
the `tx-001` in the Connectome case.

## Monitoring track (event-driven)

### Feb 01 — post-op MRI ingested → `assessResponse`
- **Comparison:** pre-op vs. post-op MRI; resection cavity, no measurable residual /
  new lesion.
- **RANO applied (deterministic):** no progression criteria met, within expected post-op
  appearance.
- **Verdict:** **stable** → `prog-001` (RANO, post-op baseline), added to the lesion
  timeline as a candidate for sign-off.

### Apr 15 — follow-up MRI ingested → `assessResponse`
- **Comparison:** post-op baseline vs. current; **new enhancement at the resection margin**,
  measurable increase.
- **Confounder check:** outside the immediate post-chemoradiation pseudo-progression window
  → not suppressed.
- **RANO applied:** progression criteria **met** (new measurable enhancing disease).
- **Verdict:** **progression** → `prog-002`; `prog-001 ─progresses_to→ prog-002` (interval
  73 d). Matches the Connectome case.

### Loop closes
The `progression` verdict can **re-trigger planning** (`replanOnProgression`) — second-line
options — closing the treat → monitor → re-plan loop.

## What Pathways did vs. didn't

| Pathways did | Pathways did **not** |
|---|---|
| build a grounded, sequenced plan + trial match | persist it (Connectome) or confirm it (MDT) |
| apply RANO deterministically each follow-up | diagnose (Reasoner) |
| auto-assess on each new study (event-driven) | order/execute the treatment |
| flag pseudo-progression risk window | call progression inside it |
| propose candidate tx-001 / prog-001 / prog-002 | overrule the tumor board |
