# Multi-Agent Pursuit-Evasion with Communication

This repository contains our course project on multi-agent cooperation in a pursuit-evasion game. We build on the MAPPO codebase and focus on the `simple_tag` environment from the Multi-Agent Particle Environment (MPE), where three predator agents coordinate to capture one prey agent in a continuous 2D world with static obstacles.

Our project studies two questions:

1. How should reward be redesigned so each predator receives a clearer credit signal?
2. When does explicit communication actually improve coordination, and when is it redundant?

## Project Summary

We modify the original `simple_tag` task in three main ways:

- **Credit assignment reward redesign**: predator rewards distinguish direct capture from teammate capture, add per-agent distance shaping, and penalize obstacle collisions.
- **Discrete communication channel**: predators can send one-hot communication tokens that are observed by other predators on the next step.
- **Communication diagnostics**: we evaluate bandwidth, communication cutoff interventions, per-speak cost, probing analysis, and role-emergence behavior.

The main conclusion is that communication can improve return over a no-communication baseline, but its benefit quickly saturates. In this near-fully observable environment, learned messages encode some prey-spatial information but are not strictly necessary for successful capture.

## Project Report

[Multi-Agent Pursuit-Evasion Report](Project_Report.pdf)


## Repository Map

```text
.
├── onpolicy/
│   ├── config.py                              # MAPPO and communication arguments
│   ├── envs/mpe/scenarios/simple_tag.py       # modified pursuit-evasion environment and rewards
│   ├── envs/mpe/environment.py                # communication action-space handling
│   ├── runner/shared/mpe_runner.py            # communication penalty and logging
│   ├── algorithms/r_mappo/                    # MAPPO policy and actor/critic code
│   └── scripts/
│       ├── train_mpe_scripts/                 # training and experiment shell scripts
│       ├── eval/                              # evaluation, intervention, plotting, and probing scripts
│       ├── analysis/                          # role clustering / behavioral analysis
│       └── results/MPE/simple_tag/mappo/      # saved experiment outputs
├── docs/
│   ├── EXPERIMENTS.md                         # how to rerun major experiments
│   ├── PROJECT_ORGANIZATION.md                # what is core code vs generated output
│   └── upstream_MAPPO_README.md               # original MAPPO README
├── environment.yaml
├── requirements.txt
└── setup.py
```

## Core Code Changes

The most important project-specific files are:

- `onpolicy/envs/mpe/scenarios/simple_tag.py`
  - adds communication settings for `simple_tag`
  - modifies predator and prey rewards
  - adds communication observations
- `onpolicy/envs/mpe/environment.py`
  - extends discrete action spaces with communication actions
  - supports a null communication token when per-speak cost is enabled
- `onpolicy/runner/shared/mpe_runner.py`
  - subtracts communication cost from rewards during training
  - logs communication activity, token usage, and active-token entropy
- `onpolicy/algorithms/r_mappo/algorithm/r_actor_critic.py`
  - adds action-probability access used for communication diagnostics
- `onpolicy/scripts/eval/`
  - contains fixed-opponent evaluation, communication cutoff intervention, probing, and plotting scripts

## Installation

Create an environment and install the package in editable mode:

```bash
conda env create -f environment.yaml
conda activate marl
pip install -e .
```

If your local environment already has PyTorch, NumPy, pandas, scikit-learn, matplotlib, seaborn, and gym installed, you can also use that environment directly.

## Reproducing Main Experiments

Run commands from the repository root.

Bandwidth sweep:

```bash
bash onpolicy/scripts/train_mpe_scripts/train_mpe_comm.sh
python -m onpolicy.scripts.eval.plot_mpe_bandwidth_sweep
```

Per-speak cost sweep:

```bash
bash onpolicy/scripts/train_mpe_scripts/train_mpe_comm_step1.sh
python -m onpolicy.scripts.eval.plot_mpe_comm_activity_sweeps
```

Communication cutoff intervention:

```bash
python -m onpolicy.scripts.eval.plot_mpe_cutoff_intervention
```

Communication probing:

```bash
bash onpolicy/scripts/train_mpe_scripts/run_mpe_probe_suite.sh
```

Role analysis:

```bash
python -m onpolicy.scripts.eval.collect_role_trajectories
python -m onpolicy.scripts.analysis.role_analysis
python -m onpolicy.scripts.analysis.plot_role_occupancy_cross_model
```

More details are in [`docs/EXPERIMENTS.md`](docs/EXPERIMENTS.md).

## Main Result Artifacts

Curated lightweight figures and tables are collected under:

```text
results_summary/figures/
results_summary/tables/
```

The original generated outputs are stored under:

```text
onpolicy/scripts/results/MPE/simple_tag/mappo/base_reward_bandwidth_sweep/
onpolicy/scripts/results/MPE/simple_tag/mappo/base_reward_cutoff_intervention/
onpolicy/scripts/results/MPE/simple_tag/mappo/base_reward_role_analysis/
```

The full `results/` directory also contains model checkpoints and intermediate evaluation files. These are useful for reproducibility, but they make the repository large. See [`docs/PROJECT_ORGANIZATION.md`](docs/PROJECT_ORGANIZATION.md) for recommended cleanup options before final submission.

## Acknowledgement

This project is built on the open-source MAPPO implementation from Yu et al., "The Surprising Effectiveness of PPO in Cooperative Multi-Agent Games." The original upstream README is preserved in [`docs/upstream_MAPPO_README.md`](docs/upstream_MAPPO_README.md).
