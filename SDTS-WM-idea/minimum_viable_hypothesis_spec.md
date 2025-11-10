{
"claim": "After 50k training steps on FrozenLake-v1 (4×4, slippery), an agent using Stochastic-DTS (jointly-optimised stochastic world model + differentiable tree search) will exhibit a catastrophe rate (episodes ending in a hole) at least 50% lower than an agent using a separately pre-trained stochastic world model with the same architecture but standard MCTS planning, evaluated over 1,000 episodes.",
"dataset": "OpenAI Gym FrozenLake-v1 (4×4, is_slippery=True) with rewards +1 (goal), −10 (hole), 0 otherwise",
"metric": "Catastrophe rate = (holes / 1,000 evaluation episodes)",
"baseline": "Separately trained stochastic world model (same 32-d latent, 1-layer MLP architecture) optimized on prediction loss for 50k steps, then used with standard MCTS planning",
"success_threshold": "S-DTS catastrophe rate ≤ 0.5 × (baseline catastrophe rate), with the difference statistically significant (two-sided proportion z-test, p < 0.05)",
"budget": {
"compute": "1 GPU (RTX 3090/4090 or A100)",
"hours": "< 1",
"memory": "≤ 8 GB"
}
}
