Goal: Train the `SDTSAgent` using the joint optimization procedure and evaluate its final performance.
Budget: { gpu_type: "A100", max_hours: 3.0, max_memory_GB: 8 }
Output:
- Core Deliverables:
  - `/outputs/s_dts_agent.pth`: The saved `state_dict` of the trained `SDTSAgent`.
  - `/outputs/s_dts_results.json`: A JSON file containing `{ "catastrophe_rate": float, "total_episodes": 1000 }`.
- Verification Artifacts:
  - `/outputs/s_dts_training_log.json`: A JSON file logging the `LQ` loss and world model loss per training step.
  - `/outputs/s_dts_loss.png`: A plot of the `LQ` loss over training steps.
Guidelines:
- Create a new script `src/train_s_dts.py`.
- Instantiate the `SDTSAgent` from Task 5. The world model inside should be initialized from scratch.
- Implement the main training loop for 50,000 steps.
- In each step, the agent interacts with the environment. For each action selection, it runs the differentiable search.
- The primary loss is the planning loss `LQ` (e.g., MSE between search output Q-values and target Q-values from a replay buffer, similar to MuZero or DTS). The world model is also trained with its own loss `L_WM` as in Task 3. The total loss is `L_total = LQ + L_WM`.
- The key is that `LQ`'s gradients flow back through the search and into the world model parameters, achieving joint optimization.
- After 50,000 training steps, perform the final evaluation: run 1,000 episodes using the trained agent's policy and calculate the catastrophe rate.
- Use Adam optimizer with learning rate `1e-4` and a fixed random seed of `42`.
Additional Context: This task executes the experimental condition of the MVH. Success here means creating an agent that learns to plan and model the world simultaneously to avoid risk. This is the most computationally intensive task.