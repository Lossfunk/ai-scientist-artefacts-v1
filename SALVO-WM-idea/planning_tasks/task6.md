Goal: Implement the SALVO-mini agent by replacing the pixel reconstruction loss with a CLIP-based perceptual loss.
Budget: { gpu_type: "A100", max_hours: 0.5, max_memory_GB: 16 }
Output:
- Core Deliverables:
    - `src/salvo_agent.py`: A module defining a `SalvoAgent` class. It should reuse the `RSSM` and `ActorCritic` but replace the reconstruction loss.
- Verification Artifacts:
    - A `test()` function that instantiates the agent, loads a frozen CLIP ViT-B/32 model, computes the perceptual loss on a single batch, and prints the loss value. The test must pass without errors.
Inputs:
- Code: `src/world_model.py`, `src/baseline_agent.py`.
- Paper: OCR'd `RESEARCH_IDEA` document, specifically Sections 3 and 5 describing the SALVO method.
Guidelines:
1.  Create `src/salvo_agent.py`. The agent class can inherit from the baseline agent and override the loss calculation.
2.  Use the `transformers` library to load a frozen `openai/clip-vit-base-patch32` model and its processor. The model must be in `eval()` mode and its gradients disabled.
3.  The perceptual loss is the Mean Squared Error between the CLIP image embeddings of the ground-truth observation `xt` and the reconstructed observation `xt_hat`. `L_perceptual = ||CLIP_vision_model(xt) - CLIP_vision_model(xt_hat)||^2`.
4.  The world model loss for SALVO-mini is `L_perceptual + beta * L_kl`. Use `beta = 1.0`.
5.  The `test()` function must verify that this new loss can be calculated and backpropagated.
6.  Add a `@stub.function(...)` to `run.py` called `test_salvo_agent` to execute this self-test.
Additional Context: The key modification of the MVH is implemented here. Ensure the CLIP model is properly loaded and frozen to avoid training it.