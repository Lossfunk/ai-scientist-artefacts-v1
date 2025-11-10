Goal: Train the experimental SALVO-mini agent for 40,000 gradient steps.
Budget: { gpu_type: "A100", max_hours: 1.5, max_memory_GB: 16 }
Output:
- Core Deliverables:
    - `/outputs/salvo_model.pth`: The saved state dictionary of the fully trained agent.
- Verification Artifacts:
    - `/outputs/salvo_training.log`: A log file with metrics (total loss, perceptual_loss, kl_loss, actor_loss, critic_loss) recorded every 500 steps.
    - `/outputs/salvo_training_curves.png`: Plots for all key losses over the 40k training steps.
Inputs:
- Code: `src/salvo_agent.py`, `src/train.py`.
- Data: `/outputs/cartpole_swingup_random_100k.npz`.
Guidelines:
1.  Reuse the `src/train.py` script from Task 5.
2.  Create a new `@stub.function(...)` in `run.py` called `train_salvo`.
3.  This function will call `src.train.train_agent`, passing the `SalvoAgent` class and a different save path (`/outputs/salvo_model.pth`).
4.  Use the same hyperparameters and random seed (`42`) as the baseline training run for a fair comparison.
Additional Context: This run generates the experimental results that will be compared against the baseline.