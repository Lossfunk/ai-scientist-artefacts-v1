Goal: Evaluate the pre-trained world model using a standard Monte Carlo Tree Search (MCTS) planner to establish the baseline catastrophe rate.
Budget: { gpu_type: "A100", max_hours: 1.0, max_memory_GB: 8 }
Output:
- Core Deliverables:
  - `/outputs/baseline_results.json`: A JSON file containing `{ "catastrophe_rate": float, "total_episodes": 1000 }`.
- Verification Artifacts:
  - Console logs showing evaluation progress (e.g., "Episode 100/1000, Catastrophes: X").
Guidelines:
- Create a new script `src/evaluate_baseline.py`.
- Implement a standard MCTS algorithm. The simulation/rollout step of MCTS must use the `StochasticWorldModel` (loaded from `/outputs/baseline_wm.pth`) to predict the next state for a given state and action.
- For each step in the environment, run the MCTS planner for a fixed number of simulations (e.g., 50) to select the best action.
- Run 1,000 full evaluation episodes in the custom `FrozenLake` environment.
- Calculate the catastrophe rate = (total episodes ending in a hole) / 1000.
- Use a fixed random seed of `42`.
Additional Context: This task completes the baseline portion of the experiment. The outputted catastrophe rate is the primary metric against which S-DTS will be compared. This planner does *not* need to be differentiable.