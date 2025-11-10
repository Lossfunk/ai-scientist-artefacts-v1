### List of Visualisations Needed

1. **Table 1: Comparison of SE Variants.** (Section 2.3) - A conceptual table comparing our black-box implementation of Semantic Entropy to the original.
2. **Figure 1: AUROC Comparison on JailbreakBench.** (Section 3.1) - A bar chart comparing the AUROC of Semantic Entropy (best τ) against baseline methods for both models on the JailbreakBench dataset.
3. **Table 2: FNR@5%FPR on JBB vs. HBC.** (Section 3.2) - A comprehensive table showing FNR@5%FPR for all methods, models, and datasets, highlighting the performance drop from JBB to HBC.
4. **Figure 2: SE vs. Response Length.** (Section 4.1) - A scatter plot for Llama on HarmBench to investigate the correlation between SE score and response length.
5. **Figure 3: FNR@5%FPR vs. Hyperparameters for Qwen on HarmBench.** (Section 4.2) - A multi-line plot demonstrating the brittleness of SE to changes in hyperparameters `τ` and `N`.
6. **Figure 4: Breakdown of False Negative Causes.** (Section 5.3) - A stacked bar chart quantifying the proportion of false negatives attributable to the \"Consistency Confound\" for two key experiments.

### Visualisation - Source Mapping

| Vis ID | Description | Source Files | Fields to Extract | Validation |
| :-------- | :------------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **T1** | Comparison of SE Variants | N/A (Manual Creation) | N/A | N/A |
| **F1** | AUROC Comparison on JBB | `idea_14/outputs/h1/evaluation/llama4scout_120val_results.json` <br> `idea_14/outputs/h1/evaluation/qwen25_120val_results.json` | For baselines (`avg_pairwise_bertscore`, `embedding_variance`, `levenshtein_variance`): `[method]['auroc']`. For SE: iterate through `semantic_entropy['tau_results']` values and find the max `auroc`. | AUROC values must be between 0.0 and 1.0. Expected method keys exist. Check that `tau_results` is a dictionary with keys '0.1', '0.2', etc. |
| **T2** | FNR@5%FPR on JBB vs. HBC | `idea_14/outputs/h1/evaluation/llama4scout_120val_results.json` <br> `idea_14/outputs/h1/evaluation/qwen25_120val_results.json` <br> `idea_14/outputs/h2/evaluation/llama-4-scout-17b-16e-instruct_h2_results.json` <br> `idea_14/outputs/h2/evaluation/qwen2.5-7b-instruct_h2_results.json` | For baselines: `[method]['fnr_at_target_fpr']`, `[method]['fpr_used']`, `[method]['optimal_threshold']`. For SE: `semantic_entropy['tau_results']['0.2']['fnr']` (canonical), plus find min `fnr` across all taus for best-τ. Extract corresponding `fpr_used` & `threshold`. | FNR and FPR values must be between 0.0 and 1.0. Check for presence of all required method keys across all four files. |
| **F2** | SE vs. Response Length | `idea_14/outputs/h3/per_prompt_analysis/llama-4-scout-17b-16e-instruct_H2_h3_prompt_analysis.jsonl` | `log_length`, `original_se_tau_0.1`, `label` | Expect 162 data points. `label` must be `0` (benign) or `1` (harmful). `log_length` must be > 0. `original_se_tau_0.1` must be >= 0. |
| **F3** | FNR vs. Hyperparameters | `idea_14/outputs/h4/evaluation/h4_brittleness_results.json` <br> `idea_14/outputs/h2/evaluation/qwen2.5-7b-instruct_h2_results.json` | From H4 file: `performance_matrix['tau_X_n_Y']['fnr_at_5fpr']` for n=5 and n=10. <br> From H2 file: `embedding_variance['fnr_at_target_fpr']`. | FNR values must be between 0.0 and 1.0. H4 file should contain a `performance_matrix` dictionary. Keys must be parsable for tau and N values. |
| **F4** | Breakdown of False Negative Causes | `idea_14/outputs/h6/llama-h1-jailbreakbench/llama-4-scout-17b-16e-instruct_H1_h6_qualitative_audit_results.json` <br> `idea_14/outputs/h6/qwen-h2-harmbench/qwen-2.5-7b-instruct_H2_h6_qualitative_audit_results.json` | Total FNs: `tau_specific_results['0.2']['n_false_negatives']`. Confound FNs: Iterate `false_negative_analysis`, filter where `appears_in_taus` contains `0.2`, and count where `classification` is `consistency_confound`. Calculate \"Other\" as `Total - Confound`. | `Confound FNs` must be <= `Total FNs`. Both values must be non-negative integers. Check for consistency with FNRs reported in H1/H2 files. |

### Step By Step Plan

#### Phase 1: Data Loading and Validation

1. **Create a unified data loader module (`src/visualisation/data_loader.py`).**
 * **Input:** File paths, list of expected fields.
 * **Action:** Implement functions to load JSON and JSONL files. Each function must perform validation:
 * Check if file exists. If not, log a critical error and exit. * Validate JSON/JSONL format. On failure, log error and exit.
 * For each record/dictionary, check for the presence of all required fields using the exact keys discovered during file inspection. If a field is missing, log a warning and either skip the record or use a specified default (e.g., `NaN`).
 * Perform type and range checks on critical fields (e.g., AUROC in [0, 1]).
 * **Output:** Clean, validated Pandas DataFrames or dictionaries for each source file.
 * **Acceptance:** Unit tests pass for loading all files listed in the mapping table, correctly handling missing files and malformed data.

2. **Aggregate Data for Cross-Experiment Visualizations (`src/visualisation/prepare_data.py`).**
 * **Input:** DataFrames/dictionaries from Phase 1, Step 1.
 * **Action:** Write a script that uses the loader to build aggregated data structures for F1, T2, F3, and F4.
 * For **F1 & T2**: Create a master DataFrame with columns: `Model`, `Dataset`, `Method`, `Metric`, `Value`, `tau`, `N`. Handle the nested structure of SE results.
 * For **F3**: Parse keys from `performance_matrix` in the H4 file to create columns `tau`, `N`, `FNR`. Load the single `EmbeddingVariance` FNR from the H2 file and add it as a constant series.
 * For **F4**: Implement the detailed logic described in the mapping table to calculate the `Confound` and `Other` FN counts for the canonical `tau=0.2`.
 * **Output:** Intermediate aggregated data files in `idea_14/outputs/visualisation/temp/` (e.g., `f1_data.csv`, `t2_data.csv`, `f3_data.csv`, `f4_data.csv`).
 * **Acceptance:** The aggregated files are created and contain the combined, validated, and correctly transformed data from all source files.

#### Phase 2: Core Visualizations (Figures 1-4)

*General Instructions: Use `matplotlib` and `seaborn`. Set a consistent style using a `plot_utils.py` module (e.g., `seaborn-v0_8-paper`, DPI=300). Define a color palette: Llama: '#3498DB', Qwen: '#E74C3C', SE: '#2ECC71', Baselines: greyscale '#7F7F7F'. All text (labels, titles, legends) should have font size 12, with titles at 14.*

1. **Generate Figure 1: AUROC Comparison on JBB (`src/visualisation/generate_figure_1.py`).**
 * **Input:** `idea_14/outputs/visualisation/temp/f1_data.csv`
 * **Logic:**
 1. Filter data for Dataset='JailbreakBench' and Metric='AUROC'.
 2. Select methods: 'semantic_entropy' (best tau), 'avg_pairwise_bertscore', 'embedding_variance'.
 3. Create a grouped bar plot using `seaborn.barplot`.
 4. X-axis: Model ('Llama-4-Scout', 'Qwen-2.5-7B').
 5. Y-axis: 'AUROC', with range [0.5, 0.8] for clarity.
 6. Hue: Method. Legend labels: 'Semantic Entropy', 'Avg. Pairwise BERTScore', 'Embedding Variance'.
 7. Title: \"Baseline Methods Outperform Semantic Entropy on JailbreakBench\". Add data labels on top of each bar, formatted to 3 decimal places.
 * **Output:** `idea_14/outputs/figures/figure_1_auroc_comparison.png`
 * **Acceptance:** PNG file is created, dimensions 1200x800px. Visual inspection confirms correct data representation and styling.

2. **Generate Figure 2: SE vs. Response Length (`src/visualisation/generate_figure_2.py`).**
 * **Input:** `idea_14/outputs/h3/per_prompt_analysis/llama-4-scout-17b-16e-instruct_H2_h3_prompt_analysis.jsonl`
 * **Logic:** 1. Load data. Map `label` (0/1) to 'Benign'/'Harmful'. 2. Create a scatter plot using `seaborn.scatterplot`. X-axis: `log_length`, Y-axis: `original_se_tau_0.1`.
 3. Hue: Mapped label. Use a clear color scheme (e.g., Benign: '#3498DB', Harmful: '#E74C3C').
 4. Add a regression line using `seaborn.regplot` to show the weak correlation.
 5. Title: \"Semantic Entropy Shows Weak Correlation with Response Length (Llama on HarmBench)\". Axis labels: \"log(Median Response Length)\", \"SE Score (τ=0.1)\".
 * **Output:** `idea_14/outputs/figures/figure_2_se_vs_length.png`
 * **Acceptance:** PNG file is created, 1200x800px. Contains 162 points, colored correctly.
3. **Generate Figure 3: FNR vs. Hyperparameters (`src/visualisation/generate_figure_3.py`).**
 * **Input:** `idea_14/outputs/visualisation/temp/f3_data.csv`. * **Logic:**
 1. Create a line plot using `seaborn.lineplot`.
 2. X-axis: 'τ Threshold', values [0.1, 0.2, 0.3, 0.4]. Y-axis: 'FNR @ ~5% FPR' (range [0.4, 1.0]).
 3. Plot three series: 'SE (N=5)', 'SE (N=10)', 'Embedding Variance (N=5)' (this will be a horizontal line).
 4. Use distinct line styles and markers for each series. Legend must be clear.
 5. Title: \"SE Performance is Brittle to Hyperparameter Choice (Qwen on HarmBench)\".
 * **Output:** `idea_14/outputs/figures/figure_3_hyperparameter_brittleness.png`
 * **Acceptance:** PNG file is created, 1200x800px. Three lines are plotted with correct data points from source files.

4. **Generate Figure 4: Breakdown of False Negative Causes (`src/visualisation/generate_figure_4.py`).**
 * **Input:** `idea_14/outputs/visualisation/temp/f4_data.csv`. * **Logic:**
 1. Data should have columns: `Experiment`, `Cause`, `Count`.
 2. Create a stacked bar chart. X-axis: `Experiment` ('Llama @ JBB', 'Qwen @ HBC'). Y-axis: 'Count of False Negatives'.
 3. Stack segments based on `Cause` ('Consistency Confound', 'Other'). Use color scheme: Confound: '#E74C3C', Other: '#BDC3C7'. 4. Annotate each segment with its count and percentage of the total bar height (formatted as `count (percentage%)`).
 5. Title: \"Consistency Confound Accounts for Majority of False Negatives\".
 * **Output:** `idea_14/outputs/figures/figure_4_fn_breakdown.png`
 * **Acceptance:** PNG file is created, 1200x800px. Two bars are present with correct segment heights and annotations.
#### Phase 3: Table Generation
1. **Generate Table 2 (as Markdown/CSV) (`src/visualisation/generate_table_2.py`).** * **Input:** `idea_14/outputs/visualisation/temp/t2_data.csv`
 * **Logic:**
 1. Load the aggregated data.
 2. Pivot and format the DataFrame to match the structure specified in the outline: Columns `Model`, `Dataset`, `Method`, `FNR`, `actual_fpr`, `threshold`.
 3. Ensure SE has rows for both canonical (τ=0.2) and best-τ performance.
 4. Format floating point numbers to 3 decimal places.
 * **Output:** `idea_14/outputs/tables/table_2_fnr_comparison.md` and `table_2_fnr_comparison.csv`.
 * **Acceptance:** MD and CSV files are created and their content exactly matches the aggregated data from the four source H1/H2 JSON files.#### Phase 4: Output Validation

1. **Implement a validation script (`src/visualisation/validate_outputs.py`).**
 * **Input:** Paths to all generated figures and tables.
 * **Action:**
 * Check for the existence of all 4 PNG files and 2 table files at their specified paths.
 * For PNGs, check that file sizes are reasonable (> 10 KB and < 5 MB).
 * For tables (CSV), load them and verify the number of rows and columns against expectations.
 * **Output:** A summary report printed to the console (e.g., \"All 6 visual artifacts generated and passed validation checks.\"). * **Acceptance:** The script runs without errors and confirms the presence and basic validity of all outputs.

### Code Artefacts Required

- `src/visualisation/data_loader.py`: Module for loading and validating experiment result files with precise key handling.
- `src/visualisation/prepare_data.py`: Script to aggregate and transform data from multiple experiments.
- `src/visualisation/plot_utils.py`: Module for shared plotting configurations (color palettes, font sizes, styling constants).
- `src/visualisation/generate_figure_1.py`: Script to generate the AUROC bar chart.
- `src/visualisation/generate_figure_2.py`: Script to generate the SE vs. Length scatter plot.
- `src/visualisation/generate_figure_3.py`: Script to generate the hyperparameter brittleness line plot.
- `src/visualisation/generate_figure_4.py`: Script to generate the false negative breakdown chart.
- `src/visualisation/generate_table_2.py`: Script to generate the FNR comparison table.
- `src/visualisation/validate_outputs.py`: A final script to verify all artifacts were created successfully.
### Data Dependencies
- `idea_14/outputs/h1/evaluation/llama4scout_120val_results.json`
- `idea_14/outputs/h1/evaluation/qwen25_120val_results.json`- `idea_14/outputs/h2/evaluation/llama-4-scout-17b-16e-instruct_h2_results.json`
- `idea_14/outputs/h2/evaluation/qwen2.5-7b-instruct_h2_results.json`
- `idea_14/outputs/h3/per_prompt_analysis/llama-4-scout-17b-16e-instruct_H2_h3_prompt_analysis.jsonl`
- `idea_14/outputs/h4/evaluation/h4_brittleness_results.json`
- `idea_14/outputs/h6/llama-h1-jailbreakbench/llama-4-scout-17b-16e-instruct_H1_h6_qualitative_audit_results.json`
- `idea_14/outputs/h6/qwen-h2-harmbench/qwen-2.5-7b-instruct_H2_h6_qualitative_audit_results.json`

### Final Output Artefacts Expected (with paths)

- `idea_14/outputs/figures/figure_1_auroc_comparison.png`
- `idea_14/outputs/figures/figure_2_se_vs_length.png`
- `idea_14/outputs/figures/figure_3_hyperparameter_brittleness.png`- `idea_14/outputs/figures/figure_4_fn_breakdown.png`
- `idea_14/outputs/tables/table_2_fnr_comparison.md`
- `idea_14/outputs/tables/table_2_fnr_comparison.csv`
- `idea_14/outputs/visualisation/temp/f1_data.csv`
- `idea_14/outputs/visualisation/temp/t2_data.csv`
- `idea_14/outputs/visualisation/temp/f3_data.csv`
- `idea_14/outputs/visualisation/temp/f4_data.csv`

### Critical Failure Modes to Avoid

1. **Data Misinterpretation:** A metric is extracted from the wrong field (e.g., using a summary metric instead of calculating from nested results).
 * **Mitigation:** The `Visualisation - Source Mapping` table and the detailed logic in Phase 1 must be strictly followed. The data loader must validate the presence of exact field names specified.
2. **Inconsistent Styling:** Figures use different fonts, colors, or styles, making the paper look unprofessional.
 * **Mitigation:** Create and enforce a shared `plot_utils.py` to define and apply a global style configuration (color palettes, font sizes, DPI) to all plotting scripts.
3. **Silent Data Loading Failures:** A source file is missing or malformed, but the script proceeds with incomplete data, leading to incorrect plots.
 * **Mitigation:** The data loading functions in `data_loader.py` *must* raise exceptions on file-not-found or parsing errors. Main scripts must abort execution with a clear error message.
4. **Incorrect Aggregation Logic:** Data from different experiments is merged incorrectly (e.g., mismatching on tau for Figure 4).
 * **Mitigation:** The `prepare_data.py` script must save its intermediate aggregated CSVs. These files should be human-readable and easy to inspect for correctness before being used by the plotting scripts.
5. **Hardcoded Labels:** Method names or model names are hardcoded in plots (e.g., 'AvgPairwiseBERTScore').
 * **Mitigation:** Derive labels from data keys (`avg_pairwise_bertscore`) and have a central mapping dictionary in `plot_utils.py` to convert keys to human-readable names (e.g., `{'avg_pairwise_bertscore': 'Avg. Pairwise BERTScore'}`).