Goal: Create the complete project scaffold, including the Modal app orchestrator, directory structure, and a container image with all necessary dependencies.
Budget: { gpu_type: "any", max_hours: 0.1, max_memory_GB: 8 }
Output:
- Core Deliverables:
    - `/project/run.py`: A Modal stub file defining the main app, a shared volume for `/outputs`, and the Python environment image.
    - `/project/src/`: An empty directory for task logic modules.
    - `/project/outputs/`: An empty directory mapped to a `modal.SharedVolume`.
- Verification Artifacts:
    - Console output confirming the Modal app is initialized and the file structure is created.
Inputs: None
Guidelines:
1.  Create a file `run.py` that initializes a `modal.App` named "mvh-salvo-mini".
2.  Define a `modal.SharedVolume` that will be mounted at `/outputs` in all subsequent tasks.
3.  Define a `modal.Image` that installs the following Python libraries with the specified versions:
    - `torch==2.3.1`
    - `torchvision==0.18.1`
    - `dm_control==1.0.14`
    - `gymnasium==1.0.0a2`
    - `transformers==4.41.2`
    - `accelerate==0.30.1`
    - `tensorboard==2.16.2`
    - `pandas==2.2.2`
    - `moviepy==1.0.3`
    - `scikit-image==0.23.2`
4.  The `run.py` file should include a placeholder function `scaffold()` decorated with `@stub.local_entrypoint()` that simply prints "Project scaffold created."
5.  The directory structure must be exactly as specified in the Output section.
Additional Context: This task is foundational. All subsequent tasks will add new functions to `run.py` and new logic files into `src/`.