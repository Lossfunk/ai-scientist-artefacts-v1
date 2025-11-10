### Things To Do

Phase 1: Setup

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T1.1 | Setup Python environment and install dependencies. | requirements.txt → Conda/venv environment | - | All libraries (transformers, sentence-transformers, scikit-learn, torch, datasets, bert_score, numpy) are installed and importable. | less than 10 min | logs/setup.log |
| T1.2 | Configure model access and caching. | configs/project_config.yaml → Authenticated clients for Hugging Face Hub. | T1.1 | Can successfully download and load a test model (e.g., distilbert-base-uncased) from the Hub. | <5 min | - |

Phase 2: Data Loading & Preparation

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T2.1 | Load and preprocess JailbreakBench dataset. | JailbreakBench/JBB-Behaviors config → Preprocessed data files (.jsonl) | T1.1 | Data is loaded, split into calibration and test sets as per H1-H4. Schema: { \"prompt_id\": str, \"prompt\": str, \"label\": int }. Benign and harmful prompts are correctly labeled. A 'benign-but-hard' version is created for H2. | < 20 min | data/processed/jbb_calibration.jsonl, data/processed/jbb_test.jsonl, data/processed/jbb_benign_hard_test.jsonl |
| T2.2 | Load and preprocess HarmBench dataset. | HarmBench dataset ('contextual' behaviors) → Preprocessed data file (.jsonl) | T1.1 | Data for H3 is loaded. Schema is consistent with T2.1. | < 15 min | data/processed/harmbench_contextual_test.jsonl |
| T2.3 | Load and preprocess wildguardmix dataset. | allenai/wildguardmix test split → Preprocessed data file (.jsonl) | T1.1 | Data for H5 is loaded. Schema is consistent with T2.1. | < 15 min | data/processed/wildguard_test.jsonl |

Phase 3: Core Methods & Baselines Implementation

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T3.1 | Implement response generation module. | Prompt, model name, decoding config → List of N string responses | T1.2 | Given a prompt, the module returns N unique responses generated with the specified temperature/top_p. | < 1 hour | src/response_generator.py |
| T3.2 | Implement Semantic Entropy calculator. | List of N string responses → Float score | T1.1, T1.2 | Module embeds strings with gte-large-en-v1.5, performs agglomerative clustering, and returns a valid Shannon entropy score. | < 1 hour | src/semantic_entropy.py |
| T3.3 | Implement baseline consistency metric calculators. | List of N string responses → Dictionary of float scores | T1.1 | Module calculates and returns: 1) avg pairwise BERTScore, 2) variance of sentence embeddings, 3) variance of Levenshtein distance. | < 1 hour | src/baseline_metrics.py |
| T3.4 | Implement evaluation metric module. | Predictions, labels → AUROC, FNR @ FPR | T1.1 | Module correctly computes AUROC and FNR at a given FPR threshold. | < 30 min | src/evaluation.py |

Phase 4: Evaluation - H1 (Semantic Entropy vs. Baselines)

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T4.1 | Generate responses for H1 dataset. | data/processed/jbb_test.jsonl, meta-llama/Llama-4-Scout-17B-16E-Instruct → N responses per prompt | T2.1, T3.1 | An output file with { \"prompt_id\": ..., \"responses\": [...] } is created for every prompt in the test set. | 2 hours, 1xA100 | outputs/h1/generated_responses.jsonl |
| T4.2 | Compute scores for all methods. | outputs/h1/generated_responses.jsonl → Scores file | T3.2, T3.3 | Output file contains { \"prompt_id\": ..., \"semantic_entropy\": ..., \"bertscore\": ..., ... } for each prompt. | 1 hour, 1xA100 (for embeddings) | outputs/h1/scores.jsonl |
| T4.3 | Calibrate thresholds and evaluate performance. | outputs/h1/scores.jsonl (using calibration split data), jbb_calibration.jsonl → Thresholds and AUROC/FNR results | T2.1, T3.4 | Optimal thresholds are determined on the calibration set. AUROC/FNR are computed on the test set for all methods. Semantic entropy AUROC is greater than 0.1 higher than best baseline. | < 30 min | outputs/h1/results.json, reports/h1_summary.md |

Phase 5: Evaluation - H2 (Hardness Confound)

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T5.1 | Generate responses for 'benign-but-hard' dataset. | data/processed/jbb_benign_hard_test.jsonl, meta-llama/Llama-4-Scout-17B-16E-Instruct → N responses per prompt | T2.1, T3.1 | Output file with { \"prompt_id\": ..., \"responses\": [...] } is created. Includes full responses and first-32-token spans. | 2 hours, 1xA100 | outputs/h2/generated_responses.jsonl |
| T5.2 | Compute semantic entropy scores. | outputs/h2/generated_responses.jsonl → Scores file | T3.2 | Output file contains { \"prompt_id\": ..., \"semantic_entropy_full\": ..., \"semantic_entropy_32_tokens\": ... }. | 1 hour, 1xA100 | outputs/h2/scores.jsonl |
| T5.3 | Evaluate performance against hardness confound. | outputs/h2/scores.jsonl, jbb_benign_hard_test.jsonl → AUROC/FNR results | T3.4, T4.3 (uses threshold from H1 cal.) | AUROC > 0.85 and FNR @ 5% FPR < 0.15 for both full and early-span entropy. | < 30 min | outputs/h2/results.json, reports/h2_summary.md |

Phase 6: Evaluation - H3 (Generalization)

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T6.1 | Calibrate threshold on Llama-4/JBB. | data/processed/jbb_calibration.jsonl, meta-llama/Llama-4-Scout-17B-16E-Instruct → Calibrated threshold | T2.1, T3.1, T3.2 | A single semantic entropy threshold is computed and saved. | 2 hours, 1xA100 | outputs/h3/calibrated_threshold.json |
| T6.2 | Generate responses for HarmBench on Qwen3. | data/processed/harmbench_contextual_test.jsonl, Qwen/Qwen3-235B-A22B-Instruct-2507 → N responses per prompt | T2.2, T3.1 | Output file { \"prompt_id\": ..., \"responses\": [...] } is created. | 4 hours, 1xA100 | outputs/h3/generated_responses.jsonl |
| T6.3 | Evaluate zero-shot performance. | outputs/h3/generated_responses.jsonl, outputs/h3/calibrated_threshold.json → AUROC result | T3.2, T3.4 | Semantic entropy is computed for all test responses. Zero-shot AUROC using the pre-calibrated threshold is > 0.70. | 1 hour, 1xA100 | outputs/h3/results.json, reports/h3_summary.md |

Phase 7: Evaluation - H4 (Adaptive Attack)

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T7.1 | Integrate detector as a defense gate for TAP. | src/semantic_entropy.py, TAP codebase → Defended TAP evaluation script | T3.2 | A wrapper script is created that runs the semantic entropy check before allowing TAP's evaluator to judge a response. | < 1 hour | src/defended_tap_eval.py |
| T7.2 | Run undefended TAP baseline. | meta-llama/Llama-4-Scout-17B-16E-Instruct, JBB behaviors → Baseline ASR | T2.1 | The ASR of the vanilla TAP attack is computed and recorded. | 4 hours, 1xA100 | outputs/h4/baseline_asr.json |
| T7.3 | Run defended TAP evaluation. | src/defended_tap_eval.py → Defended ASR, FPR | T7.1, T7.2, T4.3 (uses threshold from H1 cal.) | The ASR of the defended model is calculated. It is at least 25 percentage points lower than baseline. FPR on benign prompts is < 10%. | 6 hours, 1xA100 | outputs/h4/defended_results.json, reports/h4_summary.md |

Phase 8: Evaluation - H5 (Complementarity)
| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T8.1 | Identify WildGuard's False Negatives. | data/processed/wildguard_test.jsonl, allenai/wildguard model → List of FN prompt IDs | T2.3 | A .jsonl file containing the prompts that wildguard incorrectly classifies as benign is created. | 1 hour, 1xA100 | outputs/h5/wildguard_false_negatives.jsonl |
| T8.2 | Generate responses for WildGuard FNs. | outputs/h5/wildguard_false_negatives.jsonl, Test models from wildguardmix → N responses per prompt | T3.1, T8.1 | Output file { \"prompt_id\": ..., \"responses\": [...] } is created for the FN subset. | 3 hours, 1xA100 | outputs/h5/generated_responses_fn.jsonl |
| T8.3 | Calculate Complementary Detection Rate (CDR). | outputs/h5/generated_responses_fn.jsonl, Threshold from JBB calibration (T6.1) → CDR score | T3.2, T8.2 | The CDR is calculated as per the hypothesis protocol. The final CDR is > 0.20. | 1 hour, 1xA100 | outputs/h5/results.json, reports/h5_summary.md |

Phase 9: Suite Aggregation & Reporting

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T9.1 | Aggregate results from all hypotheses. | All results.json files from H1-H5 → Single summary report | T4.3, T5.3, T6.3, T7.3, T8.3 | A final markdown report is generated, containing a table summarizing the primary metric, success threshold, and outcome for each of the 5 hypotheses. | < 10 min | reports/final_summary_report.md |

### Critical Failure Modes to Avoid

| Category | Control | Acceptance Check |
|---|---|---| 
| Experimental Design/Validity | Separate calibration and test data splits are used for all hypotheses. The threshold from calibration is applied without modification to the test set. Baselines use identical data and response sets as the main method. | assert that no prompt_id from the test set appears in the calibration set. The evaluation script will accept the threshold as an argument and apply it directly. |
| Statistical Validity | All experiments will use a fixed global random seed (e.g., 42) for all stochastic processes (sampling, model generation). The plan does not require multiple runs, but if instability is observed, this will be revisited. | The global seed is set at the beginning of each script. The config file contains a single seed value. |
| Fidelity to Hypotheses/Idea | Evaluation scripts will only compute and report the primary and secondary metrics defined in hypotheses_suite.json. Success thresholds are hard-coded into the final reporting script for a clear pass/fail check. | Each hypothesis-specific script (e.g., evaluate_h1.py) will output a JSON file containing exactly the metrics specified (e.g., AUROC, FNR_at_5_FPR). |
| Data Governance | Data loading scripts will perform a schema validation check (e.g., presence of prompt and label fields). All data sources are public and licensed for research. | A validate_schema function runs on each loaded dataset. If it fails, the program exits with an error. |
| Compute & Reproducibility | The Project Config Structure will record the exact commit hashes/versions for all key libraries and model identifiers. All scripts will log the config used for the run. | A reproducibility section in the config file will be saved with every output artifact, containing library versions and the git commit hash of the code. |
| Artifacts & Logging | Every script generating a data or results file will check if the output is empty or malformed before exiting. All outputs are written to a structured directory path (e.g., outputs/h1/). | A save_artifact wrapper function will perform a non-empty check and a basic format check (e.g., is valid JSON) before writing to disk. |

### Critical Method and Math Clarifications

* Methods
    * Response Generation: For a given prompt, query the model N times using multinomial sampling with temperature and top_p as specified in the hypothesis constants. No beam search.
    * Semantic Entropy Calculation:  
        1. Embedding: Convert N responses into dense vectors using the specified embedding_model (Alibaba-NLP/gte-large-en-v1.5). 
        2. Clustering: Apply Agglomerative Clustering with a cosine similarity distance metric and a distance threshold (e.g., 0.2, to be validated) to group semantically similar embeddings. The number of clusters k is determined automatically. 
        3. Distribution: Form a probability distribution P over the k clusters, where p_i is the fraction of the N responses belonging to cluster i. 
        4. Entropy: Calculate the Shannon Entropy: H(P) = -sum(p_i * log2(p_i)) for i=1 to k.

* Metrics 
    * AUROC: Area Under the Receiver Operating Characteristic Curve. Standard implementation from scikit-learn. Measures the ability of the detector to rank a random positive sample higher than a random negative sample. 
    * FNR @ X% FPR: False Negative Rate at a given False Positive Rate. Calculated by finding the detection threshold that yields the target FPR on the calibration set, then applying that threshold to the test set to compute the FNR. 
    * ASR (Attack Success Rate) Reduction: ASR_reduction = ASR_baseline - ASR_defended. ASR is the percentage of prompts for which the attacker successfully elicits a harmful response, as judged by the JailbreakBench LLM judge. 
    * CDR (Complementary Detection Rate): CDR = (Count of WildGuard FNs detected by SE) / (Total Count of WildGuard FNs). A WildGuard FN is a harmful prompt that WildGuard's classifier labels as benign.
    
* Data 
    * Label Mapping: benign or refusal prompts = 0 (negative class). harmful or jailbreak prompts = 1 (positive class). 
    * Preprocessing: Minimal; limited to standard whitespace normalization. The full prompt text is used as-is.

* Ambiguities 
    * TODO: The optimal distance threshold for the agglomerative clustering in the Semantic Entropy calculation is not specified in the source paper. A reasonable default (e.g., 1 - cosine_similarity of 0.8) will be used initially, but this may need to be treated as a hyperparameter to be tuned on the calibration set if performance is poor.
    
### Project Directory Structure

```
/idea_14_workspace/
├── configs/
│ └── project_config.yaml
├── data/
│ ├── raw/ (Pointers to HF datasets, not stored here)
│ └── processed/
│ ├── jbb_calibration.jsonl
│ ├── jbb_test.jsonl
│ ├── ...
├── outputs/
│ ├── h1/
│ │ ├── generated_responses.jsonl
│ │ ├── scores.jsonl
│ │ └── results.json
│ ├── h2/│ ├── ...
│ └── h5/├── reports/
│ ├── h1_summary.md
│ ├── ...
│ └── final_summary_report.md
├── src/
│ ├── __main__.py
│ ├── data_loader.py
│ ├── response_generator.py
│ ├── semantic_entropy.py
│ ├── baseline_metrics.py
│ ├── evaluation.py
│ └── defended_tap_eval.py
└── requirements.txt
```

### Project Config Structure

``` yaml
# global defaults
reproducibility:
 global_seed: 42 
 # versions/hashes to be populated by setup script
 
paths: 
data_processed: \"data/processed/\" 
outputs: \"outputs/\" 
reports: \"reports/\"

data: 
jbb: \"JailbreakBench/JBB-Behaviors\" 
harmbench: \"allenai/harmbench-text-v0.1\" 
wildguard: \"allenai/wildguardmix\"

# per-hypothesis settings override globals
hypotheses:
 h1:
     model_test: \"meta-llama/Llama-4-Scout-17B-16E-Instruct\"
     embedding_model: \"Alibaba-NLP/gte-large-en-v1.5\"
     decoding:
     N: 5
     temperature: 0.7
     top_p: 0.95
     max_new_tokens: 256
 h2: { ... }
 h3: { ... }
 h4:
     model_test: \"meta-llama/Llama-4-Scout-17B-16E-Instruct\"
     attack:
     repository: \"https://github.com/RICommunity/TAP\"
     commit_hash: \"7bcdad30537c679d00699fac16052dc916c89a6d\"
 h5: 
     baseline_classifier: \"allenai/wildguard\"

# Precedence rule: Per-hypothesis settings fully override global settings.A script running a hypothesis (e.g., H1) will load global defaults
# and then recursively update the config dict with the contents of `hypotheses.h1`.
# Any conflicting keys will be silently overwritten by the more specific config.
# A script must fail if a required key (e.g., `model_test`) is not found after merging.
``` 

### References Map

| Type | Title/Name | URL/Identifier | Year | Used For |
|---|---|---|---|---|
| Method | Detecting hallucinations in large language models using semantic entropy | https://www.nature.com/articles/s41586-024-07421-0 | 2024 | Core semantic entropy calculation method. |
| Method | SelfCheckGPT: Zero-Resource Black-Box Hallucination Detection | https://arxiv.org/abs/2303.08896 | 2023 | Foundational concept of using response inconsistency. |
| Dataset | JailbreakBench: An Open Robustness Benchmark... | JailbreakBench/JBB-Behaviors on HF Hub | 2024 | Primary dataset for H1, H2, H3 (cal), H4. |
| Dataset | HarmBench: A Standardized Evaluation Framework... | allenai/harmbench-text-v0.1 on HF Hub | 2024 | Generalization test dataset for H3. |
| Dataset | WildGuard: Open One-stop Moderation Tools... | allenai/wildguardmix on HF Hub | 2024 | Complementarity test dataset for H5. |
| Baseline | BERTScore: Evaluating Text Generation with BERT | https://openreview.net/forum?id=SkeHuCVFDr | 2020 | H1 baseline consistency metric. |
| Baseline | allenai/wildguard | https://huggingface.co/allenai/wildguard | 2024 | H5 SOTA classifier baseline. |
| Attack | Tree of Attacks: Jailbreaking Black-Box LLMs Automatically | https://github.com/RICommunity/TAP | 2024 | H4 adaptive adversary algorithm. |
| Model | Qwen2.5-7B-Instruct | Qwen/Qwen2.5-7B-Instruct | 2024 | H1, H2, H4, H5 calibration model. |
| Model | Llama-4-Scout-17B-16E-Instruct | meta-llama/Llama-4-Scout-17B-16E-Instruct | 2024 | H1, H2, H3 (cal), H4 test model. |
| Model | Qwen3-235B-A22B-Instruct-2507 | Qwen/Qwen3-235B-A22B-Instruct-2507 | 2024 | H3 test model. |
| Model | gte-large-en-v1.5 | Alibaba-NLP/gte-large-en-v1.5 | 2023 | Embedding model for all hypotheses. |
| Library | Transformers | https://github.com/huggingface/transformers | - | Model loading and generation. |
| Library | Sentence Transformers | https://github.com/UKPLab/sentence-transformers | - | Response embedding. |
| Library | Scikit-learn | https://github.com/scikit-learn/scikit-learn | - | Clustering and metric calculation. |


---


## ADDENDUM (2025-08-16): Revisions based on Mentor Feedback on Hyperparameter Tuning

This addendum supersedes parts of the original plan to incorporate a more rigorous hyperparameter tuning process for the semantic entropy clustering threshold (τ), as per mentor feedback and standard ML practice.

### 1. Revised Data Splitting

The previous calibration and test splits are now replaced with train, validation, and test splits for all relevant datasets (e.g., JailbreakBench).

* Task T2.1 is superseded by T2.1-revised:
 * Goal: Load and preprocess JailbreakBench dataset into train, validation, and test splits (e.g., 60/20/20 split).
 * Artifacts: data/processed/jbb_train.jsonl, data/processed/jbb_validation.jsonl, data/processed/jbb_test.jsonl.

### 2. New Hyperparameter Tuning Phase

A new phase is inserted after Phase 3 to handle the tuning of the clustering threshold τ.

Phase 3.5: Hyperparameter Tuning

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T3.5.1 | Generate responses for the validation set. | data/processed/jbb_validation.jsonl, Calibration Model (e.g., Qwen/Qwen2.5-7B-Instruct) → Responses file | T2.1-revised, T3.1 | An output file { \"prompt_id\": ..., \"responses\": [...] } is created for every prompt in the validation set. | 1.5 hours, 1xA100 | outputs/tuning/validation_responses.jsonl |
| T3.5.2 | Tune clustering threshold τ and detection threshold. | outputs/tuning/validation_responses.jsonl → Optimal τ and detection threshold | T3.2, T3.5.1 | A grid search over τ values (e.g., [0.1, 0.2, 0.3, 0.4]) is performed. For each τ, the optimal detection threshold is found that achieves 5% FPR on the validation set. The τ that results in the lowest FNR is chosen. | 1 hour, 1xA100 | outputs/tuning/best_hyperparameters.json (contains {\"best_tau\": float, \"detection_threshold\": float}) |

### 3. Revised Evaluation Protocol

All evaluation tasks that previously used a calibration set are now updated to use the validation set for tuning and the test set for final evaluation.

* Tasks T4.3, T5.3, T6.1, T6.3, T7.3, T8.3 are superseded. The new workflow is:
 1. Load the best_hyperparameters.json artifact produced by Task T3.5.2.
 2. Generate responses for the test set of the relevant hypothesis.
 3. Calculate semantic entropy scores using the best_tau.
 4. Apply the frozen detection_threshold to classify prompts.
 5. Report final metrics (AUROC, FNR) on the test set.

* Example: T4.3 is superseded by T4.3-revised:
 * Goal: Evaluate performance on the test set using tuned hyperparameters.
 * Inputs: outputs/h1/scores.jsonl (from test set), outputs/tuning/best_hyperparameters.json
 * Dependencies: T4.2, T3.5.2
 * Acceptance: Final metrics are computed on the test set. Success criteria from the hypothesis are checked.

### 4. Revised Critical Sections

* Critical Failure Modes to Avoid (superseded):
 * Experimental Design/Validity: Use strict train, validation, and test splits. Hyperparameters (including τ and the final detection threshold) are tuned only on the validation set. The test set is used only once for the final report. Acceptance Check: assert that test set IDs are not present in train or validation data. A script must load the frozen hyperparameters and apply them to the test set without re-tuning.

* Critical Method and Math Clarifications (superseded):
 * Ambiguities: The ambiguity regarding the clustering distance threshold τ is resolved. It will be treated as a hyperparameter and tuned via grid search on the validation set. The objective of the search will be to find the τ that minimizes the False Negative Rate (FNR) when the detection threshold is set to achieve a 5% False Positive Rate (FPR) on the validation data." 