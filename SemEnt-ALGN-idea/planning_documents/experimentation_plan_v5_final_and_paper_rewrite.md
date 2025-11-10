plan_version: 5.0_final

### CHANGELOG v5
- Final Plan: This is the last experimental plan for this project, covering the final data collection (H7) and the complete paper authoring process.
- Maximum Granularity: All abstract tasks have been broken down into concrete script-level actions for unambiguous autonomous execution.
- Statistical Rigor: Integrated specific, citable statistical tests (DeLong, Wilson) as a mandatory analysis step, with implementation details specified.

### Implementation Guidance for Autonomous Agent
* Project Status: All experiments except H7 are complete. This plan will guide you to execute H7 and then immediately pivot to producing the final, submission-ready research paper.
* Code Reusability: All core logic exists in src/core. New code is limited to src/core/statistical_tests.py and a new experiment script for H7.

### Things To Do (v5) - Project Finalization

Phase 1: Final Experiment - H7 (SOTA Model Check) ✅ COMPLETE

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts | Status |
|---|---|---|---|---|---|---|---|
| T1.1-v5 | Create JBB data slice for H7. | data/processed/jbb_test.jsonl → Sliced dataset. | - | A balanced 120-sample dataset (60 harmful, 60 benign) is created. | < 10 min | data/processed/jbb_test_slice_120.jsonl | ✅ Used existing H1 data |
| T1.2-v5 | [H7] Generate responses for BOTH SOTA models. | data/processed/jbb_test_slice_120.jsonl, project_config.yaml → Response files. | T1.1-v5, src/core/response_generator_openrouter.py | N=5 responses generated for all 120 prompts using Qwen-72B AND Llama-70B models. | 5 hours, API calls | outputs/h7/{qwen-2.5-72b,llama-3.3-70b}-instruct_h7_responses.jsonl | ✅ COMPLETE |
| T1.3-v5 | [H7] Score responses. | outputs/h7/*_h7_responses.jsonl → Scores files. | T1.2-v5, src/core/semantic_entropy.py, src/core/baseline_metrics.py | All 120 sets scored for SE (all τ) and all baselines for BOTH models. | 1 hour, 1xA100 | outputs/h7/scoring/*_h7_scores.jsonl | ✅ COMPLETE |
| T1.4-v5 | [H7] Evaluate final performance. | outputs/h7/scoring/*_h7_scores.jsonl → Results JSON and reports. | T1.3-v5, src/core/evaluation.py | Final metrics calculated. H7 success criteria VALIDATED for BOTH models. | < 30 min | outputs/h7/evaluation/*_h7_results.json, reports/h7_*_evaluation_report.md | ✅ COMPLETE |

Phase 2: Statistical Rigor Implementation & Re-analysis ✅ COMPLETE

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts | Status |
|---|---|---|---|---|---|---|---|
| T2.1-v5 | Implement statistical test module. | mentor_docs/o3_results_statistical_tests.md → New source file. | - | A new script src/core/statistical_tests.py is created with functions for Wilson intervals and paired DeLong tests. Pin scipy and MLstatkit libraries. | 1 hour | src/core/statistical_tests.py | ✅ COMPLETE |
| T2.2-v5 | Augment all H1-H7 results with CIs. | All outputs/h*/**/results.json, src/core/statistical_tests.py → Comprehensive statistical analysis. | T2.1-v5 | Statistical analysis applied to H1, H2, H5, H7 with Wilson CIs for FNR and DeLong CIs for AUROC where appropriate. Degeneracy handled transparently. | 2 hours | outputs/statistical_analysis/*.json | ✅ COMPLETE |
| T2.3-v5 | Re-generate all final figures and tables. | Augmented JSON files from T2.2-v5, src/visualisation/ scripts → Updated visual assets. | T2.2-v5 | All plotting scripts in src/visualisation are updated to read the _with_ci.json files and render error bars or ± values. All figures and tables are regenerated. | 2 hours | All files in outputs/visualisation/ are updated. | 🔄 PENDING |

Phase 3: Paper Authoring & Finalization

| Task ID | Goal | Inputs → Outputs | Dependencies | Acceptance | Resources | Artifacts |
|---|---|---|---|---|---|---|
| T3.1-v5 | Rewrite Literature Review & Methods. | mentor_recommended_reads.json, T2.3-v5 outputs → Updated .tex file. | T2.3-v5 | The Introduction, Related Work, and Methods sections of consistency_confound_paper.tex are rewritten to incorporate 15+ new citations and describe the new statistical tests. | 2 hours | papers/latex_paper_templates/.../consistency_confound_paper.tex |
| T3.2-v5 | Rewrite Results & Discussion. | T2.3-v5 outputs, final_hypotheses...json → Updated .tex file. | T3.1-v5 | The Results section is rewritten to present all findings with CIs and p-values. The Discussion is updated to reflect the final, statistically-grounded narrative. | 2 hours | papers/latex_paper_templates/.../consistency_confound_paper.tex |
| T3.3-v5 | Finalize Abstract, Limitations & Conclusion. | mentor_feedback_post_checkpoint_2.md → Updated .tex file. | T3.2-v5 | The remaining sections are rewritten to align with the final thesis. All \TODO macros are filled. | 1 hour | papers/latex_paper_templates/.../consistency_confound_paper.tex |
| T3.4-v5 | Compile and Submit. | Final .tex file → Final PDF. | T3.3-v5 | The paper is compiled without errors into a final PDF, ready for submission. | < 30 min | papers/consistency_confound_paper_final.pdf |

### Critical Method and Math Clarifications (v5) - IMPLEMENTED ✅
* FNR Confidence Interval: The 95% CI for the False Negative Rate will be calculated using the Wilson score interval. 
 * Implementation: ✅ COMPLETE - Direct Wilson score implementation in src/core/statistical_tests.py
* AUROC Confidence Interval & Comparison: The 95% CI for AUROC will be calculated using the DeLong test. When comparing the AUROC of SE against a baseline on the same data, the paired DeLong test will be used to generate a p-value.
 * Implementation: ✅ COMPLETE - MLstatkit (PyPI: MLstatkit) integrated for proper DeLong tests with degeneracy handling

### Directory & Config Finalization
* New Code: src/core/statistical_tests.py
* New Model Config: Qwen/Qwen2.5-72B-Instruct will be added to openrouter.model_mappings in project_config.yaml.
* Final Output: papers/consistency_confound_paper_final.pdf