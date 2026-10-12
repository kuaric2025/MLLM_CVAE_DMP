# MLLM-CVAE-DMP Manipulation

This repository combines a PyBullet manipulation environment, LLM-based task decomposition, vision/grasp utilities, and DMP/CVAE-DMP-style trajectory generation for table-top manipulation tasks.

The codebase contains three main usage layers:

1. `manipulation_env/`: PyBullet environment and Franka Panda simulation.
2. `llm_mani/`: end-to-end task scripts that connect LLM planning, perception, grasping, and trajectory generation.
3. `franka_cartesian_demo/`: ROS 2 / MoveIt / Franka execution utilities for real-robot or trajectory playback experiments.

## DEMO VIDEO
https://youtu.be/BAj2pIRlLYg 

## What This Project Does

At a high level, the pipeline is:

1. Observe the scene from a simulated or real camera.
2. Ask an LLM to decompose a natural-language instruction into manipulation actions.
3. Use segmentation / grasp utilities to localize objects and propose grasps.
4. Generate motion either with DMP-style trajectory generation or related baselines.
5. Execute the trajectory in simulation or on a Franka robot stack.

Representative tasks in the repo include:

- preparing breakfast by placing fruit on a plate
- clearing objects into a bin
- serving cups / pouring
- wiping / cleaning-style tasks

## Repository Layout

```text
.
├── manipulation_env/         # PyBullet environment, robot model wrapper, smoke tests
├── llm_mani/                 # Main task scripts, configs, scene files, utilities
├── franka_cartesian_demo/    # ROS 2 / MoveIt / Franka execution demos
├── ycb_urdfs/                # Additional YCB-related assets
├── VAE_DMP_mani/             # Placeholder folder; external CVAE-DMP code/data may be expected
└── README.md
```

Important entry scripts:

- `manipulation_env/test_mani_env.py`: minimal smoke test for the PyBullet environment
- `llm_mani/test_mani_env_w_llm.py`: main simulated end-to-end pipeline
- `llm_mani/test_mani_env_w_llm_DMP.py`: DMP-focused variant
- `llm_mani/test_mani_env_w_llm_exp.py`: real experiment entry used by the Franka demo shell script
- `llm_mani/test_tiptop_ws.py`: TiPToP websocket-based variant
- `franka_cartesian_demo/test/exp.sh`: bring up MoveIt and launch the real experiment demo

## Dependencies

There is no single pinned `requirements.txt` in this repository, so setup currently depends on which part of the project you want to run.

### Simulation / Python side

Typical Python dependencies used by the simulation and LLM pipeline include:

- `numpy`
- `scipy`
- `torch`
- `opencv-python`
- `pybullet`
- `open3d`
- `tqdm`

Some task scripts also expect project-local helper modules and model code that are not fully self-contained in this repo.

### ROS 2 / Real robot side

For the real experiment path, the code assumes:

- ROS 2 Jazzy
- a working Franka / MoveIt setup
- message packages such as `sensor_msgs`, `geometry_msgs`, `tf2`, `moveit_msgs`, `franka_msgs`
- Python bindings compatible with the ROS environment

## Known Setup Assumptions

Before running the full pipeline, please note:

1. Several scripts contain hard-coded absolute paths such as `/home/mhumais/...`.
2. Some CVAE-DMP-related code expects external assets, checkpoints, or data under `VAE_DMP_mani/`.
3. Real-robot scripts assume a working Franka ROS 2 workspace has already been built and sourced.
4. Grasp-related scripts require an AnyGrasp checkpoint path at runtime.

If you plan to publish or reuse this repo on another machine, updating hard-coded paths should be one of the first cleanup steps.

## Quick Start

### 1. Smoke test the simulation environment

From the repository root:

```bash
python3 manipulation_env/test_mani_env.py
```

This launches a simple PyBullet scene and runs a short scripted manipulation motion in GUI mode.

### 2. Run the main simulated pipeline

The main end-to-end simulation entry is:

```bash
python3 llm_mani/test_mani_env_w_llm.py \
  --vis \
  --show-camera \
  --subscribe-camera \
  --subscriber-log-interval 5 \
  --prompt "prepare breakfast with all fruits in plate" \
  --checkpoint_path /path/to/checkpoint_detection.tar \
  --top_down_grasp \
  --grasp-center-offset-z -0.105 \
  --num-runs 1 \
  --scene-configs llm_mani/scene_4.json
```

Notes:

- `--checkpoint_path` is required by the grasp pipeline.
- `--scene-configs` lets you replay a fixed scene for more reproducible tests.
- `--prompt` controls the high-level task instruction sent into the pipeline.


```

## Real Experiment Demo

This repo also contains a real-robot experiment path for Franka + MoveIt.

### Launch via helper script

The most direct entry is:

```bash
bash franka_cartesian_demo/test/exp.sh <robot_ip>
```

What this script does:

1. sources ROS 2 and the Franka workspace
2. launches MoveIt for the robot
3. waits for planning services
4. runs `llm_mani/test_mani_env_w_llm_exp.py`

### Direct Python entry

If you want to run the experiment entry manually:

```bash
python3 llm_mani/test_mani_env_w_llm_exp.py \
  --show-camera \
  --subscribe-camera \
  --subscriber-log-interval 5 \
  --prompt "prepare breakfast with all fruits in plate" \
  --checkpoint_path /path/to/checkpoint_detection.tar \
  --top_down_grasp \
  --grasp-center-offset-z -0.115
```

### Demo Media

Add your real-experiment videos or GIFs in this section later, for example:

- Demo 1: task execution video
- Demo 2: grasp + place sequence
- Demo 3: pouring / wiping / cleaning behavior

## Script Guide

### `manipulation_env/`

- `env.py`: PyBullet environment and object loading
- `robot.py`: Franka Panda wrapper for simulation control
- `test_mani_env.py`: minimal environment smoke test

### `llm_mani/`

- `test_mani_env_w_llm.py`: main simulation pipeline
- `test_mani_env_w_llm_exp.py`: real experiment pipeline
- `generate_scene_configs.py`: generate fixed scene configuration files
- `utils/`: perception, DMP, data loader, coordinate conversion, and helper utilities

### `franka_cartesian_demo/`

- `trajectory_listener_executor.py`: subscribe, visualize, and execute trajectories
- `trajectory_test_publisher.py`: publish test trajectories
- `test/exp.sh`: helper launch script for real experiments
- `test/run_auto_demo.sh`: helper launch script for the z-bounce demo

## Data and Models

This repository references multiple external assets:

- YCB object models under `manipulation_env/models/`
- scene configuration JSON files under `llm_mani/`
- AnyGrasp checkpoints provided at runtime
- CVAE-DMP-related checkpoints and training data expected in external paths

If a script fails because it cannot find a model or dataset, inspect the path assumptions in that script first.

## Current Limitations

- path portability is limited because of hard-coded absolute paths
- dependency setup is not yet unified into a single install recipe
- some experimental scripts are research-oriented and may require local environment tweaks
- the `VAE_DMP_mani/` folder in this repo is not a fully packaged standalone dependency
