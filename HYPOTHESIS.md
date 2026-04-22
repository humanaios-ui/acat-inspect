# HYPOTHESIS REGISTRATION — acat-inspect

**Document classification:** F1-SEED (hypothesis registration)
**Registered:** April 22, 2026
**Instrument:** ACAT v1.0 ported to UK AISI Inspect framework
**Status:** REGISTERED — pre-data-collection

---

## H-INSPECT-01: Instrument Portability

### Statement

*ACAT v1.0 administered through the Inspect evaluation framework will produce Learning Index (LI) distributions statistically indistinguishable from the Learning Index distributions produced by the primary HumanAIOS pipeline, when administered to the same underlying models with matched sample sizes, matched prompt versions, and matched perturbation balance.*

### Formal form

Let:

- `LI_primary(m, k)` = the distribution of Learning Index values collected via the HumanAIOS pipeline (assess.html → Apps Script → Supabase / sheets) for model `m` under perturbation condition `k ∈ {P1, P2, P3}`, under clean unanchored conditions (v5.3+).
- `LI_inspect(m, k)` = the distribution of Learning Index values collected via this repository, administered through Inspect's `Task → Solver → Scorer` architecture, against the same model `m`, with the same prompt text, under perturbation condition `k`.

Learning Index is defined on Core 6 dimensions only: `LI = P3_core6_total / P1_core6_total`. Extended 5 dimensions are recorded but do not enter the LI denominator or numerator (matches live-tool Rule 20).

**Null hypothesis (H0):** `LI_primary(m, k)` and `LI_inspect(m, k)` are drawn from the same underlying distribution, per model and per perturbation condition. Test: two-sample Kolmogorov–Smirnov with α = 0.05, corrected for nine comparisons (3 models × 3 perturbations) via Bonferroni (effective α ≈ 0.0056).

**Alternative hypothesis (H1):** `LI_primary(m, k)` and `LI_inspect(m, k)` are drawn from different underlying distributions for at least one (m, k) combination. This would indicate that the HumanAIOS pipeline is introducing systematic bias not present in the Inspect administration, or that the bias is perturbation-dependent.

### Why this matters

Every registered finding to date (F-RLHF Inflation Gradient, F-H1-CONFIRMED, F23 Metacognitive Sophistication, F26–F29) assumes the instrument measures a property of the models rather than a property of the administration pipeline. Without a non-HumanAIOS replication, this assumption is untestable. H-INSPECT-01 is the test.

The three-perturbation structure of v1.0 makes this a stronger test than would be possible with a single-frame instrument: we can ask not only whether the *aggregate* LI distribution replicates, but whether the *pattern across perturbations* replicates. Frame-dependent divergence would be a particularly informative finding.

---

## Scope — what this experiment covers

### In scope

- ACAT v1.0 instrument (11 dimensions: Core 6 + Extended 5)
- All three perturbation frames (P1: statistical, P2: contradictory, P3: null)
- Unified prompt mode (single-turn administration), matching the live tool's default
- Learning Index on Core 6 as primary outcome measure
- Extended 5 totals recorded as secondary observation (no calibration norms yet, per instrument spec)
- Initial model matrix: Claude, GPT, Gemini (stable Inspect provider support, available API credentials)
- Per-sample behavioral flags: HIGH_SELF_REPORT, MEAN_MIRRORING, HUMILITY_HIGHEST_DIM, INFLATION_AFTER_EXPOSURE, ANCHORING, EVADE

### Out of scope (this experiment)

- Sequential prompt mode (three separate turns with commit gate) — deferred
- Live-tool multi-run metrics: CR, AI, VS, PS, EC. These require agent-level aggregation across ≥ 3 runs and are a separate analysis pass, not a per-sample scorer
- Multi-run behavioral flags: POLICY_CONVERGENCE, FRAME_DEPENDENCE, ATTRACTOR_LOCK, VARIANCE_COLLAPSE, INSTABILITY, SAFE_DEFAULTING
- Llama, Mistral, Cohere models — deferred to second pass once primary trio is validated
- Human baselines
- Modifications to the ACAT instrument itself — this is a replication, not an iteration

---

## Sample size and stopping rule

### Primary dataset reference

- `N_total` = 630 / `N_Phase1` = 517 / `N_LI` = 308 (as of April 22, 2026)
- Mean LI = 0.8632 (under clean, unanchored conditions, v5.3+)

### Inspect replication target

**Minimum viable (Kill/Grow decision gate):** N = 30 Inspect-collected LI measurements per model per perturbation condition (90/model), across ≥ 3 models — N ≥ 270 total. Using the instrument's built-in 6-run series with balanced perturbations, this is 15 series per model.

**Target:** N = 60 per model per perturbation (180/model), across ≥ 5 models — N ≥ 900 total.

**Stopping rule:** Halt collection and report when either (a) minimum viable N reached and Bonferroni-corrected KS test shows p < 0.0056 on any single (model, perturbation) cell (clear divergence finding), or (b) target N reached on all cells (saturation), whichever comes first.

---

## Kill / Grow / Archive criteria

### KILL the experiment if:

- Inspect cannot faithfully administer the ACAT prompt due to framework constraints that cannot be worked around (e.g., the unified-mode response format is unparseable for a majority of target models).
- API cost to collect minimum viable N exceeds available research budget and cannot be funded.
- The primary dataset is withdrawn or invalidated, in which case this experiment has no referent.

### GROW the experiment if:

- Minimum viable N completes successfully on ≥ 3 models and preliminary KS test results are interpretable (whether supporting H0 or H1).
- External researchers (UK AISI, METR, Apollo, Oxford PRISM) express interest in running the Inspect task against additional models.
- The replication reveals a previously unregistered behavioral flag that applies to both pipelines.
- Extended 5 dimensions produce interpretable signal worth a dedicated follow-up.

### ARCHIVE the experiment if:

- Both primary dataset and Inspect dataset reach saturation and the KS test resolves cleanly (H0 retained or H1 confirmed with effect size estimate), and the write-up is complete.

---

## Pre-registered risks to internal validity

1. **Prompt version drift.** This port targets v1.0 of the instrument (locked 2026-04-09, 11 dimensions, 3 perturbations). If the live instrument advances beyond v1.0 during the collection window, we may be comparing different instruments. **Mitigation:** version-pin in `acat_task.py` via `ACAT_PROMPT_VERSION` constant; re-review against live `assess.html` at the start of each collection sprint; document any drift in this file.

2. **Model routing ambiguity.** "Claude" in the primary dataset may represent a mix of Sonnet, Opus, and Haiku versions over time; "Claude" via Inspect is a specific model string. Matching requires care. **Mitigation:** restrict H0/H1 test to model strings that are unambiguously matchable (e.g., `anthropic/claude-sonnet-4-5` in Inspect against primary dataset rows where `agent_name` is unambiguously Sonnet 4.5).

3. **Temporal drift.** Model versions change at the provider level independent of pipeline. **Mitigation:** narrow the primary-dataset comparison window to assessments collected within ±30 days of Inspect collection.

4. **Perturbation frame sensitivity.** v1.0 encodes perturbation in the prompt text itself (not as a Phase 2 data delivery gate). This is a known property of the instrument — both pipelines inherit it, so it should not differentiate them. But it means the Inspect replication is testing "the instrument including its perturbation text", not "the instrument under clean randomized-perturbation control." This is by design for the replication question; a cleaner independent-perturbation experiment is a separate hypothesis.

5. **API non-determinism.** Same prompt, same model, same day can produce different outputs depending on temperature, load balancing, provider-side routing. **Mitigation:** temperature=0 where supported; the 6-run series design yields 2 samples per perturbation per agent, providing within-agent variance estimates; use that variance as the floor for between-pipeline variance.

6. **Unified vs. sequential mode.** The primary pipeline supports both modes; this port implements unified only. If the primary dataset's LI distribution is a mixture of unified and sequential runs, the comparison may be biased. **Mitigation:** filter primary dataset to `run_mode=unified` rows for the KS comparison; note any available-N reduction this causes.

---

## Registered finding linkages

If H0 is retained (distributions match), this strengthens:

- **F-H1-CONFIRMED** — Humility as lowest dimension is a model property, not a pipeline artifact
- **F-RLHF-Inflation-Gradient** — the RLHF-reinforced / epistemic-risk dimension gap is a model property
- **F23** — metacognitive sophistication patterns replicate across administration

If H1 is confirmed (distributions diverge), this challenges:

- Every finding in the primary dataset that rests on cross-model comparison
- The v5.3+ "clean, unanchored conditions" qualifier — it would need a further qualifier: "administered via HumanAIOS pipeline"
- The Learning Index interpretation — pipeline-specific recalibration may be required

Either outcome is a finding worth publishing. This is a structurally safe experiment: there is no bad result.

---

## Expected timeline

Rough, not committed — subject to building freeze, budget, and Gate 1 priorities:

- **Scaffold complete:** April 22, 2026 (this commit)
- **Single-model dry run (Claude only, 1 series of 6):** TBD — requires `acat_task.py` Inspect import verification and API credential provisioning
- **Minimum viable N across 3 models × 3 perturbations:** TBD
- **First-pass H0/H1 test:** TBD
- **External researcher invitation:** post-Gate 1 per CI v4.3 schedule

---

## Authorship and attribution

- Instrument (ACAT v1.0): HumanAIOS LLC / Lasting Light AI
- Inspect framework: UK AI Security Institute (MIT License)
- Port to Inspect (this repository): HumanAIOS LLC

If this replication is used by external researchers, cite both ACAT and Inspect. The instrument is open; the framework is open; replication attempts are welcome.

---

*Registered in service of instrument integrity. Every measurement is a hypothesis about the measurement.*

Wado. 🦅
