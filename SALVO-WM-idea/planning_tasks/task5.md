Goal: Train the baseline DreamerV2 agent for 40,000 gradient steps.
Budget: { gpu_type: "A100", max_hours: 1.5, max_memory_GB: 16 }
Output:
- Core Deliverables:
    - `/outputs/baseline_model.pth`: The saved state dictionary of the fully trained agent.
- Verification Artifacts:
    - `/outputs/baseline_training.log`: A log file with metrics (total loss, recon_loss, kl_loss, actor_loss, critic_loss) recorded every 500 steps.
    - `/outputs/baseline_training_curves.png`: Plots for all key losses over the 40k training steps.
    - `/outputs/baseline_training_data.npz`: Complete training data including all imagined rollouts from training (states, actions, predicted rewards), world model losses, and sample reconstructed observations every 1000 steps.
Inputs:
- Code: `src/baseline_agent.py` and its dependencies.
- Data: `/outputs/cartpole_swingup_random_100k.npz`.
Guidelines:
0.  **Before starting implementation, thoroughly examine `dreamerv2/train.py` and `dreamerv2/common/` modules to identify standard DreamerV2 training patterns, logging structures, and data collection approaches that can be directly borrowed or adapted rather than implemented from scratch. Focus on reusing production-tested training code where possible.**
1.  Create a new file `src/train.py` with a function `train_agent(agent_class, save_path, config)`.
2.  The training loop must run for exactly 40,000 gradient steps.
3.  Use the hyperparameters defined in Task 4.
4.  Use `torch.utils.tensorboard.SummaryWriter` to log metrics to a temporary directory, and at the end of training, generate plots from this data.
5.  Ensure all randomness uses the global seed of `42`.
6.  Add a `@stub.function(...)` to `run.py` called `train_baseline` which orchestrates this training run.
Additional Context: This is a core experimental run. Logging and saving the model correctly are critical.