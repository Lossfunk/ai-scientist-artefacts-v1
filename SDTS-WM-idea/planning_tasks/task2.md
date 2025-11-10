Goal: Implement the `FrozenLake-v1` environment wrapper and the core Stochastic World Model (VS-WM) components.
Budget: { gpu_type: "A100", max_hours: 0.2, max_memory_GB: 8 }
Output:
- Core Deliverables:
  - `src/environment.py`: Contains a wrapper for `gymnasium.make("FrozenLake-v1", is_slippery=True)` that applies the custom rewards: +1 for goal, -10 for hole, 0 otherwise.
  - `src/world_model.py`: Contains the `StochasticWorldModel` class, an `nn.Module`, which includes:
    - An `Encoder` (1-layer MLP: `state_dim` -> `32-d latent`).
    - A `Decoder` (1-layer MLP: `32-d latent` -> `state_dim`).
    - A `DynamicsPredictor` (1-layer MLP: `32-d latent` + `action` -> `32-d latent`).
- Verification Artifacts:
  - The `world_model.py` script must include a `test_world_model()` function that, when run, instantiates the model, performs a forward pass with a dummy input tensor, and prints the output shapes to confirm correctness.
Guidelines:
- The environment has 16 discrete states and 4 discrete actions. Use embeddings for these inputs.
- The `DynamicsPredictor` should output the parameters for a categorical distribution over the next latent state.
- The `StochasticWorldModel` class should have a method that takes `(latent_state, action)` and returns a `torch.distributions.Categorical` object for the next latent state.
- Use a fixed random seed of `42` for all initializations.
- Assume the state observation is a single integer. The encoder should handle this, perhaps via an embedding layer.
Additional Context: This world model is the core component used in both the baseline (Task 3 & 4) and the experimental agent (Task 5 & 6). The architecture is intentionally simple as per the MVH_SPEC.