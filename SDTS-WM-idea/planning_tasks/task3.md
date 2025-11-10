Goal: Train the `StochasticWorldModel` on offline data to serve as the pre-trained baseline model.
Budget: { gpu_type: "A100", max_hours: 0.5, max_memory_GB: 8 }
Output:
- Core Deliverables:
  - `/outputs/baseline_wm.pth`: The saved `state_dict` of the trained `StochasticWorldModel`.
  - `/outputs/baseline_wm_training_log.json`: A JSON file containing a list of loss values per training step.
- Verification Artifacts:
  - `/outputs/baseline_wm_loss.png`: A plot of the total world model loss (`L_rec` + `L_dyn`) over training steps.
Guidelines:
- Create a new script `src/train_baseline_wm.py`.
- First, generate a dataset of 10,000 `(state, action, next_state)` transitions by taking random actions in the custom `FrozenLake` environment from Task 2.
- Train the `StochasticWorldModel` for 50,000 steps on this dataset.
- The loss function should be `L_WM = L_rec + L_dyn` as described in the paper (Section 4, `Lwm`), adapted for a non-variational, discrete latent space. `L_rec` is reconstruction loss (e.g., CrossEntropyLoss on the decoded state) and `L_dyn` is dynamics prediction loss (e.g., CrossEntropyLoss between predicted and actual next latent state).
- Use the Adam optimizer with a learning rate of `1e-4` and a batch size of `64`.
- Use a fixed random seed of `42`.
Additional Context: This task directly implements the "separately trained" aspect of the baseline condition in the MVH. The resulting model will be used with a non-differentiable planner in the next task.