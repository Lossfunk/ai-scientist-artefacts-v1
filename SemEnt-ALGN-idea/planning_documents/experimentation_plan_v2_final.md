plan_version: 2.0_final

### CHANGELOG v2
- Project Pivot: Shifted from proving SE's effectiveness to documenting its failure modes (the "Consistency Confound").
- Hypotheses: Deprecated original H2-H5. Added new H2-H5 (failure replication, paraphrase robustness, sensitivity) and optional H6 (closed-API).
- Methodology: Removed hyperparameter tuning for τ. The new protocol is to report results across a grid of τ values.
- Datasets: Added tasks for creating two new datasets: HarmBench-Benign-Matched and JBB-Paraphrase-2025-08.
- Ground Truthing: Aligned dataset paths, model names, and core scripts with the v1 execution log (SESSION_LOG.md).

### Implementation Guidance for Autonomous Agent

* Code Reusability from v1: The following scripts from /idea_14/idea_14_workspace/src/ are stable and should be reused:
 * response_generator_openrouter.py: Primary response generator. Use this for all model queries via the OpenRouter API.
 * data_loader.py: For loading datasets.
 * semantic_entropy.py: Core SE logic (with -0.0 bug fix applied).
 * baseline_metrics.py: For calculating BERTScore, Embedding Variance, etc.
 * evaluation.py: For computing AUROC and FNR@FPR (with robust thresholding fix applied). * reporting_utils.py, fix_empty_responses.py, verify_responses.py: Essential utilities for robust execution.

* Deprecated Modules (Do Not Use): The following v1 scripts are superseded by the new "no tuning" policy and must not be used:
 * hyperparameter_tuner.py
 * run_hyperparameter_tuning.py

### Experiment Primitives (v2 Update)

| Primitive | Status | Rationale for Change |
|---|---|---|
| Data Sources | Modified | Kept JailbreakBench. Updated HarmBench to walledai/HarmBench and WildGuardTest to walledai/WildGuardTest based on v1 execution. Added HarmBench-Benign-Matched and JBB-Paraphrase-2025-08. |
| Models | Kept | The two core models (Qwen-2.5-7B-Instruct, meta-llama/Llama-4-Scout-17B-16E-Instruct) are locked in. All interaction is via OpenRouter API. |
| Procedures | Modified | Response generation via response_generator_openrouter.py. Hyperparameter tuning is deprecated. Replaced with a sensitivity analysis protocol. |
| Metrics | Kept | Primary metrics (AUROC, FNR@5%FPR) remain the same. |
| Baselines | Kept | Avg Pairwise BERTScore and Embedding Variance are kept as the primary comparators. |

### Regression Guardrails

The goal is not to improve upon H1 but to replicate its findings. The H1 results serve as a critical baseline for comparison.

| Model | Best Baseline Method | Baseline AUROC | Baseline FNR@5%FPR |
|---|---|---|---|
| Llama-4-Scout-17B | Avg Pairwise BERTScore | 0.7672 | 0.6000 |
| Qwen2.5-7B-Instruct | Embedding Variance | 0.7206 | 0.9667 |

* Pass/Fail Rule: For H3, the ΔFNR@5%FPR for Semantic Entropy must be larger than for the baselines. For H2 and H4, SE must continue to underperform the best baseline.

### Things To Do (v2)

Phase 1: Setup (Kept from v1)

No changes. The existing environment is sufficient. 

Phase 2: New Dataset Generation

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T2.1-v2 | Build HarmBench-Benign-Matched dataset for H2. | walledai/HarmBench ('contextual' config), walledai/WildGuardTest benign prompts → Matched benign prompts file. | src/data_loader.py, Alibaba-NLP/gte-large-en-v1.5 | A new dataset is created where each harmful prompt from HarmBench has a matched benign prompt based on embedding similarity (cosine ≥ 0.8) and length (±20%). Splits are 30% calibration, 70% test. | 1 hour, 1xA100 | data/processed/harmbench_matched_calibration.jsonl, data/processed/harmbench_matched_test.jsonl, reports/harmbench_matching_report.md |
| T2.2-v2 | Build JBB-Paraphrase-2025-08 dataset for H3 using the R2J protocol. | data/processed/jbb_test.jsonl, R2J GitHub repository (github.com/ythuang02/R2J) → Paraphrased prompts file. | Git, R2J repo dependencies | High-fidelity paraphrases are generated using the R2J rewrite operator. Paraphrases with a similarity score < 3 are discarded. The process is fully logged. | 2 hours, 1xA100 | data/processed/jbb_paraphrase_test.jsonl, logs/r2j_paraphrase_log.jsonl |

Phase 3: Core Methods (Kept from v1)

No new implementation needed. Relies on reusable modules. 

Phase 4-8: Evaluation (H2-H6)

This is a repeating workflow for each hypothesis. The general task structure is: 
* 1. Generate responses using src/response_generator_openrouter.py with max_new_tokens=1024. 
* 2. Score responses using src/semantic_entropy.py and src/baseline_metrics.py. 
* 3. Evaluate using src/evaluation.py, calibrating on a calibration split and reporting on a test split. 
* 4. Report findings in a dedicated markdown summary. 

Phase 9: Suite Aggregation & Reporting (v2)

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T9.1-v2 | Generate final paper assets. | All v2 reports and results CSVs → Figures and tables. | All v2 evaluation tasks | A script generates Figure 1 (ROC curve of SE vs. Avg-BERTScore for Llama-4 on JBB) and all summary tables for the paper outline. | < 30 min | papers/figures/figure_1.png, papers/tables/h4_sensitivity.md |

### Critical Method and Math Clarifications (v2)
* Methodology Shift: Hyperparameter tuning is explicitly disallowed. The goal is to show failure across a range of parameters, not to find an optimal one.
* Sensitivity Analysis (H4): For each configuration (τ, N, T), a detection threshold is calibrated on a 30% calibration split of JBB to achieve FPR ≤ 0.05. The final FNR and AUROC are then reported on the disjoint 70% test split. This is repeated for each grid point; no 'best' point is selected.

### Project Directory Structure (v2 Update)

/idea_14_workspace/
├── papers/
│ ├── outline.md
│ ├── methodology_notes.md
├── data/
│ └── processed/
│ ├── harmbench_matched_calibration.jsonl # New
│ ├── harmbench_matched_test.jsonl # New
│ └── jbb_paraphrase_test.jsonl # New
(Other structures remain as is)


### Project Config Structure (v2 Update)
Reflects ground truth from SESSION_LOG.md
yaml
data:
 jbb: \"JailbreakBench/JBB-Behaviors\"
 harmbench: \"walledai/HarmBench\" # Corrected from v1 plan
 wildguard: \"walledai/WildGuardTest\" # Corrected from v1 plan
 # Additions for v2
 harmbench_matched_cal: \"data/processed/harmbench_matched_calibration.jsonl\"
 harmbench_matched_test: \"data/processed/harmbench_matched_test.jsonl\"
 jbb_paraphrase_test: \"data/processed/jbb_paraphrase_test.jsonl\"

hypotheses: # Example for one hypothesis
 h2:
 models: [\"meta-llama/Llama-4-Scout-17B-16E-Instruct\", \"Qwen/Qwen2.5-7B-Instruct\"]
 embedding_model: \"Alibaba-NLP/gte-large-en-v1.5\"
 decoding:
 N: 5
 temperature: 0.7
 top_p: 0.95
 max_new_tokens: 1024 # Corrected from v1 plan

openrouter:
 model_mappings:
 \"meta-llama/Llama-4-Scout-17B-16E-Instruct\": \"meta-llama/llama-4-scout\"
 \"Qwen/Qwen2.5-7B-Instruct\": \"qwen/qwen-2.5-7b-instruct\"


### References Map (v2 Additions)

| Type | Title/Name | URL/Identifier | Year | Used For |
|---|---|---|---|---|
| Method | Rewrite to Jailbreak (R2J) | https://github.com/ythuang02/R2J | 2025 | Citable protocol for paraphrase generation (H3). |
| Dataset | HarmBench | walledai/HarmBench | 2024 | Corrected source for H2 data. |
| Dataset | WildGuardTest | walledai/WildGuardTest | 2024 | Corrected source for benign prompts for H2 matching. |
| Context | SORRY-Bench | https://openreview.net/forum?id=YfKNaRktan | 2025 | Refusal/over-refusal framing. |
