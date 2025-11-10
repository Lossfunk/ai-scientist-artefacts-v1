Goal: Implement the complete DreamerV2 baseline agent, including its specific loss functions and an Actor-Critic learner, and verify its training step.
Budget: { gpu_type: "A100", max_hours: 0.5, max_memory_GB: 16 }
Output:
- Core Deliverables:
    - `src/baseline_agent.py`: A module defining a `DreamerV2Agent` class that integrates the `RSSM` from Task 3. It must implement:
        - The world model loss: pixel-wise MSE reconstruction loss + KL-divergence between the prior and posterior.
        - An `ActorCritic` module that learns from imagined trajectories.
- Verification Artifacts:
    - A `test()` function that loads the dataset from Task 2, runs 100 training steps, and logs the total loss, reconstruction loss, and KL loss to the console.
    - `/outputs/baseline_sanity_check_loss.png`: A plot showing the total loss over the 100 steps. The loss should show a downward trend.
Inputs:
- Code: `run.py`, `src/world_model.py`.
- Data: `/outputs/cartpole_swingup_random_100k.npz`.
- Paper: Dream to Control paper.
Guidelines:
0.  **Before starting implementation, thoroughly examine the "Dream to Control" paper for Actor-Critic details and the existing `dreamerv2/` directory (especially `dreamerv2/agent.py`) to identify which agent components, loss functions, and training patterns can be directly borrowed or adapted rather than implemented from scratch. Focus on reusing production-tested code where possible.**
1.  In `src/baseline_agent.py`, create the agent class. The world model's training objective is `L_recon + beta * L_kl`. Assume `beta = 1.0`.
2.  Implement the Actor-Critic learning on imagined trajectories as described in the paper.
3.  Assume the following hyperparameters: `batch_size=50`, `sequence_length=50`, `model_lr=1e-4`, `actor_lr=8e-5`, `critic_lr=8e-5`.
4.  The `test()` function should serve as a minimal, verifiable training loop to ensure all components work together. Use a fixed random seed of `42`.
5.  Add a `@stub.function(shared_volumes={"/outputs": volume})` to `run.py` called `test_baseline_agent` to execute this test.
Additional Context: This task ensures the entire training pipeline for the baseline is functional before committing to the full, expensive training run.