{
"claim": "On dm_control Cartpole-swingup (vision, 64×64), a Dreamer-style world model trained *only* with a frozen CLIP ViT-B/32 perceptual reconstruction loss (plus the usual KL dynamics term) supports model-predictive control that achieves an average episodic return no worse than 10 % below a pixel-reconstruction Dreamer baseline after identical training (40 k gradient steps).",
"dataset": "dm_control Cartpole-swingup vision mode (64×64 RGB), 100 k offline replay frames collected with a random policy",
"metric": "Mean episodic return over 10 evaluation episodes using 5-step CEM planning with 100 candidates",
"baseline": "DreamerV2 with pixel MSE reconstruction + KL dynamics, identical architecture and training hyper-parameters",
"success_threshold": "SALVO-mini world model is *validated* if its mean return ≥ 0.90 × (baseline mean return) OR ≥ 450 (whichever is lower); otherwise the hypothesis is falsified",
"budget": {
"compute": "1 GPU (RTX 4090 or A100)",
"hours": "≤ 3 (≈1.2 h per model training + 0.2 h evaluation)",
"memory": "≤ 16 GB"
}}
