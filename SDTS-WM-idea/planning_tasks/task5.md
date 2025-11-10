Goal: Implement the S-DTS agent, including the differentiable tree search and expected value backup mechanisms.
Budget: { gpu_type: "A100", max_hours: 0.5, max_memory_GB: 8 }
Output:
- Core Deliverables:
  - `src/s_dts.py`: Contains the `SDTSAgent` class which integrates the `StochasticWorldModel` (from Task 2) and a `DifferentiableTreeSearch` module.
- Verification Artifacts:
  - The `s_dts.py` script must include a `test_gradient_flow()` function that:
    1. Instantiates the `SDTSAgent`.
    2. Performs a small search (e.g., 5 expansions).
    3. Calculates a dummy loss based on the final leaf values.
    4. Calls `loss.backward()`.
    5. Checks that `agent.world_model.dynamics_predictor.mlp[0].weight.grad` is not `None` and prints a success message.
Guidelines:
- The `DifferentiableTreeSearch` module should manage tree expansion.
- When expanding a node, use the world model's `DynamicsPredictor` to get a distribution over the next latent states. Sample from this distribution. To ensure gradient flow for the categorical distribution, use the Gumbel-Softmax trick as mentioned in the paper (Section 5.2.a).
- The backup phase must implement the expected backup rule (Section 5.2.b). For simplicity, you can approximate the expectation by drawing `K=5` samples from the dynamics predictor and averaging their values.
- The agent's `forward` or `plan` method should take a state, run the search for a fixed number of trials (e.g., 50), and return the resulting Q-values for actions from the root.
Additional Context: This is the most complex implementation task, focusing on the core mechanism of the S-DTS paper. The self-test is crucial to ensure the end-to-end differentiability is working before attempting to train it.