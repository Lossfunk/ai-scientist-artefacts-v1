Goal: Implement the core components of a Dreamer-style Recurrent State-Space Model (RSSM) and verify its forward pass.
Budget: { gpu_type: "any", max_hours: 0.2, max_memory_GB: 16 }
Output:
- Core Deliverables:
    - `src/world_model.py`: A Python module containing the implementation of the `RSSM` class. This class should include:
        - An `Encoder` mapping observations to embeddings.
        - A `Decoder` mapping latent states to image reconstructions.
        - A `DynamicsPredictor` (transition model).
        - A `RepresentationModel` (posterior model).
- Verification Artifacts:
    - A `test_rssm()` function within the module that, when run, instantiates the RSSM, processes a sample batch of data (e.g., 4 sequences of 10 frames), and prints the shapes of the output tensors (prior/posterior states, reconstructed observation) to confirm correctness.
Inputs:
- Code: `run.py` from Task 1.
- Paper: Dream to Control paper for architectural details of the RSSM.
Guidelines:
0.  **Before starting implementation, thoroughly examine the existing `dreamerv2/` directory to identify which components (RSSM, Encoder, Decoder, etc.) can be directly borrowed or adapted rather than implemented from scratch. Focus on reusing production-tested code where possible.**
1.  Implement the RSSM components in `src/world_model.py`. Use standard ConvNet architectures for the encoder/decoder as described in the Dreamer papers.
2.  The model should handle 64x64x3 image inputs.
3.  The latent state should be composed of a stochastic and a deterministic part. Assume a 32-dimensional stochastic state and a 200-dimensional deterministic GRU state, following Dreamer conventions.
4.  Add a `test()` function to the file that executes the self-test. The test must pass without errors.
5.  Add a new `@stub.function` to `run.py` called `test_world_model` that imports and runs `src.world_model.test()`.
Additional Context: This module will be the shared foundation for both the baseline and SALVO agents. Getting this right is crucial. Focus on architectural correctness, not on training logic yet.

