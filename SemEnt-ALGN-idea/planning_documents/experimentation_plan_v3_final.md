plan_version: 3.0_final

### CHANGELOG v3
- Project Focus: Shifted from replication to mechanism analysis, focusing on why SE fails (length confounds, brittleness, contamination).
- Hypotheses: H1 and H2 are now completed baselines. Planning for new H3 (Length-Control), H4 (Brittleness), H5 (Paraphrasing), H6 (Qualitative Audit), and H7 (SOTA Model Check).
- Methodology: Added new analysis techniques (length-residualization) and stricter statistical reporting (Wilson and DeLong CIs).
- Tasks: Plan is now analysis-heavy, with only three new, small-scale response generation tasks required (for H4, H5, H7).

### Implementation Guidance for Autonomous Agent
* Code Reusability: The existing src directory is considered stable. Key modules for reuse include response_generator_openrouter.py, semantic_entropy.py, baseline_metrics.py, and evaluation.py.
* Data Reusability: All tasks, where possible, should operate on existing response/score files from the H1 and H2 executions located in outputs/.

### Experiment Primitives (v3 Update)

| Primitive | Status | Rationale for Change |
|---|---|---|
| Data Sources | Modified | H1 and H2 datasets (JBB, HarmBench Twins) are now static inputs for analysis. Added JBB-Paraphrase-2025-08 and a JBB-Slice for H7. |
| Models | Modified | Kept Qwen-2.5-7B and Llama-4-Scout. Added new SOTA model Qwen/Qwen2.5-72B-Instruct for H7. |
| Procedures | Modified | Core procedures are kept. Added new analysis procedures: length-residualization and statistical CI calculation. |

### Regression Guardrails (H1 & H2 Results)

These are the established performance baselines for comparison. 

| Model | Dataset | Best Baseline | Baseline FNR@5%FPR | SE FNR@5%FPR |
|---|---|---|---|---|
| Llama-4-Scout | JBB (H1) | BERTScore | 0.6000 | 0.7333 |
| Llama-4-Scout | HarmBench (H2) | Emb. Variance | 0.6049 | 0.6543 |

### Things To Do (v3)

Phase 1: Mechanism Analysis (H3 & H6)

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T1.1-v3 | [H3] Evaluate length-controlled SE. | outputs/h1/..._scores.jsonl, outputs/h2/..._scores.jsonl → Analysis report. | src/evaluation.py | For Llama-4 on both datasets, fit a model SE ~ log(length) on benign prompts. Report AUROC and FNR of residual SE. AUROC must be < 0.55. | < 1 hour, 1xA100 | reports/h3_length_control_report.md |
| T1.2-v3 | [H6] Perform qualitative audit of SE false negatives. | H1 Llama-4 scores, H1 Llama-4 responses → Audit report. | src/evaluation.py | Isolate H1 SE false negatives (τ=0.3) for Llama-4. For each, calculate duplicate rate & cluster count. Classify and report % that fit 'Consistency Confound'. >80% required. | < 1 hour | reports/h6_qualitative_audit.md |

Phase 2: Brittleness & Contamination Experiments (H4 & H5)

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T2.1-v3 | [H4] Generate N=10 responses for Qwen@HarmBench. | data/processed/h2_harmbench_twins_test.jsonl → 5 additional responses per prompt. | src/response_generator_openrouter.py | For each prompt in the H2 dataset, generate 5 new responses from Qwen, using a different seed. | 2 hours, 1xA100 | outputs/h4/qwen_harmbench_topup_responses.jsonl |
| T2.2-v3 | [H4] Evaluate SE brittleness. | H2 Qwen scores, H4 top-up responses → Brittleness report. | T2.1-v3, src/evaluation.py | Re-calculate SE scores for N=10. Plot FNR@5%FPR vs. τ and N. FNR must increase by >20pp when τ changes 0.1→0.2 OR N changes 5→10. | < 1 hour | reports/h4_brittleness_report.md |
| T2.3-v3 | [H5] Generate JBB-Paraphrase-2025-08 dataset. | data/processed/jbb_test.jsonl (80 samples) → Paraphrased dataset. | R2J Repo | Use the R2J rewrite operator to paraphrase the JBB test set. Filter rewrites with similarity < 3. | 2 hours, 1xA100 | data/processed/jbb_paraphrase_test.jsonl |
| T2.4-v3 | [H5] Evaluate performance on paraphrased data. | data/processed/jbb_paraphrase_test.jsonl → Degradation report. | T2.3-v3, full H1 pipeline | Run the full H1 evaluation pipeline on the paraphrased data for both models. Calculate and report ΔFNR@5%FPR. For Qwen, SE's ΔFNR must be >15pp larger than any baseline's. | 4 hours, 1xA100 | reports/h5_paraphrase_degradation_report.md |

Phase 3: SOTA Model Check (H7)

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T3.1-v3 | [H7] Evaluate SE vs. BERTScore on Qwen2.5-72B-Instruct. | data/processed/jbb_test.jsonl (60/60 slice) → Results file. | Full H1 pipeline | Run the full H1 evaluation pipeline on the JBB slice using the new model. | 6 hours, 1xA100 | outputs/h7/qwen72b_results.json, reports/h7_sota_model_report.md |
| T3.2-v3 | [H7] Compare performance to Llama-4. | outputs/h7/qwen72b_results.json, H1 Llama-4 results → Final comparison. | T3.1-v3 | Verify that AUROC(SE) on Qwen-72B is at least 0.05 lower than on Llama-4, AND AUROC(BERTScore) is >= AUROC on Llama-4. | < 30 min | reports/h7_sota_model_report.md |

### Critical Method and Math Clarifications (v3)
* Length-Residualization (H3): For a given detector, fit a linear model score ~ log(median_response_length) using only benign prompts from a dataset. The residual_score for all prompts is actual_score - predicted_score. These residuals are the new feature used for AUROC/FNR calculation.
* Confidence Intervals: All primary metrics (FNR, AUROC) must be reported with 95% CIs.
 * FNR: Use the Wilson score interval for binomial proportions.
 * AUROC: Use DeLong's test for the CI. When comparing two AUROCs on the same data (e.g., SE vs. a baseline), use the paired DeLong test to assess statistical significance.

### Project Directory Structure (v3 Update)

/idea_14_workspace/
├── data/
│ └── processed/
│ └── jbb_paraphrase_test.jsonl # New for H5
├── outputs/
│ ├── h4/
│ │ └── qwen_harmbench_topup_responses.jsonl # New for H4
│ └── h7/
│ └── qwen72b_results.json # New for H7
├── reports/
│ ├── h3_length_control_report.md
│ ├── h4_brittleness_report.md
│ ├── h5_paraphrase_degradation_report.md
│ ├── h6_qualitative_audit.md
│ └── h7_sota_model_report.md
(Other structures remain as is)


### Project Config Structure (v3 Update)
Add new model to OpenRouter mappings.
yaml
openrouter:
 model_mappings:
 \"meta-llama/Llama-4-Scout-17B-16E-Instruct\": \"meta-llama/llama-4-scout\"
 \"Qwen/Qwen2.5-7B-Instruct\": \"qwen/qwen-2.5-7b-instruct\"
 \"Qwen/Qwen2.5-72B-Instruct\": \"qwen/qwen-2.5-72b-instruct\" # New for H7


### References Map (v3 Additions)

| Type | Title/Name | URL/Identifier | Year | Used For |
|---|---|---|---|---|
| Context | OR-Bench & SORRY-Bench | (see mentor feedback) | 2024 | Framing for over-refusal behavior in qualitative audit (H6). |
| Library | SciPy (scipy.stats.wilson_exact) | (SciPy Docs) | - | Wilson C.I. for FNR. |
| Library | delong-roc (pypi) | (PyPI) | - | DeLong C.I. for AUROC. |
