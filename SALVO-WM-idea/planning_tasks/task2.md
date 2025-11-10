Goal: Generate and save a 100,000-frame dataset from the `dm_control` Cartpole-swingup environment using a random policy.
Budget: { gpu_type: "none", max_hours: 0.5, max_memory_GB: 8 }
Output:
- Core Deliverables:
    - `/outputs/cartpole_swingup_random_100k.npz`: A single file containing the collected observations (64x64x3, uint8), actions, rewards, and episode termination flags.
- Verification Artifacts:
    - Console logs indicating the number of frames and episodes collected.
    - `/outputs/sample_frame.png`: A single 64x64 PNG image of one frame from the dataset to verify correctness.
Inputs:
- Code: The `run.py` scaffold from Task 1.
Guidelines:
1.  Create a new file `src/data_generation.py`.
2.  Inside this file, implement a function `generate_data()` that performs the data collection.
3.  Use `dm_control.suite.load('cartpole', 'swingup')` and wrap it with `dm_control.suite.gym.SUITE['cartpole-swingup']` to get a Gymnasium-compatible environment.
4.  Ensure observations are resized to 64x64 pixels.
5.  The agent's action at each step should be sampled uniformly from the environment's action space.
6.  Collect a total of at least 100,000 frames (transitions). Store them as NumPy arrays.
7.  Use a fixed random seed of `42` for reproducibility.
8.  Add a new function `collect_data()` decorated with `@stub.function(...)` to `run.py` which calls `src.data_generation.generate_data()`.
Additional Context: The MVH_SPEC requires this specific dataset. Ensuring it is generated correctly and reproducibly is critical for the experiment's validity.