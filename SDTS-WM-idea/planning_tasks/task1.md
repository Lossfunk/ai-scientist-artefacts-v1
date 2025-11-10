Goal: Create the complete project scaffolding, including the Modal app orchestrator, directory structure, and a container image with all necessary Python libraries.
Budget: { gpu_type: "null", max_hours: 0.1, max_memory_GB: 2 }
Output:
- Core Deliverables:
  - `/project/run.py`: Modal stub file defining the app, image, and shared volume.
  - `/project/src/`: Empty directory for source code.
  - `/project/outputs/`: Empty directory mapped to a `modal.SharedVolume`.
Guidelines:
- Create a `run.py` file with a `modal.Stub` named "s-dts-mvh".
- Define a `modal.Image` that installs the following libraries: `torch==2.3.1`, `gymnasium==0.29.1`, `numpy==1.26.4`, `scipy==1.13.1`, `modal`.
- Define a `modal.SharedVolume` that will be mounted at `/outputs` in the container.
- The `run.py` file should include placeholder `@stub.function` definitions for all 7 tasks, each with the correct `shared_volume` mapping.
Additional Context: This task is the foundation for all subsequent tasks. The directory structure and Modal app defined here are mandatory and must not be altered by other tasks.