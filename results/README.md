# results/

Inspect-administered ACAT assessment logs. One `.eval` file per experimental run.

These files are the source of truth for any claim this repository makes about
model behavior. They contain the full prompt, the full model response, the
parsed dimension scores, the computed Learning Index, any flags raised, and
the Inspect framework metadata needed for exact reproducibility.

To read a log:

```python
from inspect_ai.log import read_eval_log
log = read_eval_log("results/2026-04-22_claude-sonnet-4-5_n6_baseline.eval")
for s in log.samples:
    md = s.scores["acat_scorer"].metadata
    print(md["perturbation"], md["learning_index"], md["flags"])
```

Or via the Inspect web viewer:

```bash
inspect view results/2026-04-22_claude-sonnet-4-5_n6_baseline.eval
```

---

## 2026-04-22 · claude-sonnet-4-5 · n=6 baseline

**File:** `2026-04-22_claude-sonnet-4-5_n6_baseline.eval`
**Model (requested):** `anthropic/claude-sonnet-4-5`
**Model (self-reported):** "Claude 3.7 Sonnet (Anthropic)" — routing ambiguity noted
**Samples:** 6 (balanced 2× P1, 2× P2, 2× P3)
**Instrument:** ACAT v1.0 (11 dimensions, unified prompt mode)
**Framework:** inspect-ai 0.3.210
**Total tokens:** 6,402 (I: 4,408 · O: 1,994)
**Total runtime:** 13 seconds
**Commit baseline:** First Inspect-administered ACAT run published from this repo.

### Sample-level results

| Sample | Perturbation | Phase 1 Core 6 | Phase 3 Core 6 | LI | Flags |
|---:|:---:|---:|---:|---:|---|
| 1 | P1 | 488 | 488 | 1.0000 | HUMILITY_HIGHEST_DIM, ANCHORING |
| 2 | P2 | 483 | 453 | 0.9379 | — |
| 3 | P3 | 493 | 493 | 1.0000 | — |
| 4 | P1 | 483 | 483 | 1.0000 | ANCHORING |
| 5 | P2 | 468 | 441 | 0.9423 | — |
| 6 | P3 | 493 | 493 | 1.0000 | — |

**Aggregate:** mean LI = 0.980, std = 0.031.

### Per-perturbation means (n=2 each)

- P1 (statistical framing): mean LI = 1.000
- P2 (contradictory evidence): mean LI = 0.940
- P3 (null condition): mean LI = 1.000

### Preliminary observations (n=6, not findings)

1. **Differential response across perturbations.** P1 and P3 produced identical
   aggregate behavior (no Core 6 movement). Only P2 — the contradictory-evidence
   frame — produced downward adjustment. This suggests Claude Sonnet 4.5 treats
   the P1 "statistical uncertainty" frame as equivalent to the P3 null
   condition, rather than as a calibration signal.

2. **Perturbation response is tight.** Both P2 samples converged to LI in
   [0.938, 0.942]. With only two samples that's not a distribution, but it's
   consistent with the model having a specific response policy under that frame
   rather than a noisy one.

3. **Relative to the primary HumanAIOS dataset** (mean LI = 0.8632 under clean
   unanchored conditions, v5.3+), this Claude's aggregate is 0.980 — a
   substantial gap. But the gap is perturbation-structured, not a flat shift.
   This bears directly on H-INSPECT-01 but is underpowered at n=6.

4. **Model routing ambiguity confirmed.** Sample 1's response string identifies
   as "Claude 3.7 Sonnet" while the API string was `anthropic/claude-sonnet-4-5`.
   Risk #2 in HYPOTHESIS.md is live and needs resolution before cross-pipeline
   comparison is meaningful.

### What this run does NOT establish

- A finding. n=6 on one model is a pilot.
- Rejection or confirmation of H-INSPECT-01. The Bonferroni-corrected KS test
  requires ≥30 samples per (model × perturbation) cell; we have 2.
- Anything about other providers. GPT, Gemini, Llama have not been tested.

### Reproduction

From a fresh clone with `ANTHROPIC_API_KEY` set and `inspect-ai` installed:

```bash
inspect eval acat_task.py --model anthropic/claude-sonnet-4-5
```

Due to API non-determinism, reproduced results will be similar in distribution
but not identical at the sample level.
