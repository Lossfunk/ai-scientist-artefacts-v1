Goal: Analyze the results from the baseline and S-DTS evaluations and generate a final report concluding whether the hypothesis is supported.
Budget: { gpu_type: "null", max_hours: 0.1, max_memory_GB: 2 }
Output:
- Core Deliverables:
  - `/outputs/final_report.md`: A markdown file summarizing the experiment.
- Verification Artifacts:
  - Console output printing the results of the statistical test.
Guidelines:
- Create a new script `src/analyze_results.py`.
- Load the results from `/outputs/baseline_results.json` and `/outputs/s_dts_results.json`.
- Perform a two-sided proportion z-test using `scipy.stats.proportions_ztest` to compare the two catastrophe rates. The number of trials for each is 1,000.
- The markdown report must contain:
  - A clear statement of the MVH.
  - A table with the results: `| Agent | Catastrophe Rate |`.
  - The results of the z-test: Z-statistic and p-value.
  - A conclusion: "The hypothesis is [SUPPORTED/REJECTED]" based on whether the S-DTS catastrophe rate is ≤ 50% of the baseline rate and the p-value is < 0.05.
Additional Context: This final task synthesizes all previous work into a single, verifiable conclusion that directly addresses the claim made in the MVH_SPEC.