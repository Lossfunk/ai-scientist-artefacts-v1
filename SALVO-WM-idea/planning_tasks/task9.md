Goal: Perform a comprehensive final analysis of the training and evaluation results to generate a final report that explicitly validates or falsifies the hypothesis.
Budget: { gpu_type: "none", max_hours: 0.2, max_memory_GB: 8 }
Output:
- Core Deliverables:
    - `/outputs/final_report.md`: A Markdown file summarizing the complete analysis. It must include:
        - **I. Executive Summary & Verdict**: A high-level summary of the findings and the final, unambiguous verdict: "HYPOTHESIS VALIDATED" or "HYPOTHESIS FALSIFIED", explicitly framed by the success criteria in the MVH spec.
        - **II. Hypothesis & Motivation Recap**: A brief summary of the core ideas from `minimum_viable_hypothesis_spec.md` and `SALVO_VLM_Semantic_Control_WM_Idea.pdf` to contextualize the results.
        - **III. Training Dynamics Analysis**: A comparison of the training logs, noting differences in loss convergence, reward learning curves, and overall stability between the Baseline and SALVO agents.
        - **IV. Quantitative Evaluation**:
            - A table with comprehensive statistics for episodic returns (Mean, Std Dev, Median, Min, Max).
            - The p-value from an independent t-test to assess the statistical significance of the performance difference.
        - **V. Planner Performance Analysis**:
            - A table comparing the Mean Absolute Error (MAE) between the planner's imagined rewards and the actual rewards received.
        - **VI. Empirical & Qualitative Observations**:
            - Bullet points summarizing key observations from the end-to-end process (e.g., model stability, planner creativity, failure modes, comparison of agent behaviors from videos).
- Verification Artifacts:
    - Console output printing the key metrics and the final verdict.
Inputs:
- **From Modal Volume**:
    - `/outputs/evaluation_results.json`: Contains detailed logs from the 10-episode evaluation of each agent.
    - `/outputs/baseline_training.log`: The training log for the baseline agent.
    - `/outputs/salvo_training.log`: The training log for the SALVO agent.
- **Conceptual Inputs**:
    - `tasks/minimum_viable_hypothesis_spec.md`
    - `tasks/SALVO_VLM_Semantic_Control_WM_Idea.pdf`
Guidelines:
1.  **Data Retrieval**: Use the `modal volume get mvh-salvo-mini-outputs <remote_path> <local_path>` command to download the required `.json` and `.log` files into the local `outputs/` directory.
2.  **Create `src/analyze.py`**: Implement a `generate_report()` function.
3.  **Analysis Logic**:
    - Load and parse all three input files.
    - Calculate all required metrics: training summary stats, evaluation return statistics (mean, std, median, min, max), t-test p-value, and imagined reward MAE.
    - Calculate the success threshold from the MVH spec.
4.  **Report Generation**:
    - Synthesize the quantitative results with the qualitative observations and the project's founding documents.
    - Format and write all sections of the report to `/outputs/final_report.md`.
5.  **Modal Integration**: Add a final `@stub.function(...)` to `run.py` called `analyze_results` that executes this analysis.
Additional Context: This is the final step that directly answers the research question. The analysis should be multi-faceted, combining training dynamics, statistical evaluation, planner accuracy, and qualitative observations to provide a robust conclusion.