# Experiment Guide

This document summarizes the major experiment entry points used in the project.

Run all commands from the repository root.

## Environment

```bash
conda env create -f environment.yaml
conda activate marl
pip install -e .
```

The plotting and probing scripts require common analysis packages:

```bash
pip install numpy pandas matplotlib scikit-learn seaborn
```

## 1. Bandwidth Sweep

Purpose: test whether communication improves performance and whether larger communication vocabularies help.

Main script:

```bash
bash onpolicy/scripts/train_mpe_scripts/train_mpe_comm.sh
```

This trains:

```text
comm_dim in {2, 4, 8, 16, 32, 64}
seeds in {1, 2, 3}
comm_target = adversaries
```

No-communication checkpoints are stored as:

```text
onpolicy/scripts/results/MPE/simple_tag/mappo/base_reward_nocomm/
```

Communication checkpoints are stored as:

```text
onpolicy/scripts/results/MPE/simple_tag/mappo/base_reward_comm{d}/
```

Plot:

```bash
python -m onpolicy.scripts.eval.plot_mpe_bandwidth_sweep
```

Representative outputs:

```text
onpolicy/scripts/results/MPE/simple_tag/mappo/base_reward_bandwidth_sweep/
```

## 2. Communication Cutoff Intervention

Purpose: test whether trained policies causally depend on communication content at evaluation time.

Main evaluation script:

```bash
python -m onpolicy.scripts.eval.eval_cutoff_intervention \
  --env_name MPE \
  --scenario_name simple_tag \
  --algorithm_name mappo \
  --model_dir onpolicy/scripts/results/MPE/simple_tag/mappo/base_reward_comm8/run1/models \
  --num_good_agents 1 \
  --num_adversaries 3 \
  --num_landmarks 2 \
  --episode_length 100 \
  --eval_episodes 200 \
  --use_simple_comm \
  --comm_dim 8 \
  --comm_target adversaries \
  --fixed_opponent_policy all \
  --share_policy
```

Interventions:

```text
vanilla   no intervention
zero      replace communication vector with all zeros
random    replay random messages from a buffer
permuted  swap messages across predator identities
noise     add clipped Gaussian noise
```

Plot:

```bash
python -m onpolicy.scripts.eval.plot_mpe_cutoff_intervention
```

Representative outputs:

```text
onpolicy/scripts/results/MPE/simple_tag/mappo/base_reward_cutoff_intervention/
```

## 3. Per-Speak Cost Sweep

Purpose: test whether agents can reduce communication without losing task performance.

Main script:

```bash
bash onpolicy/scripts/train_mpe_scripts/train_mpe_comm_step1.sh
```

Default sweep:

```text
comm_dim = 8
lambda in {0, 0.5, 0.71, 1.0, 1.41, 2.0, 10.0}
seeds in {1, 2, 3}
```

The shaped training reward is:

```text
r'_i,t = r_i,t - lambda * 1[c_i,t != 0]
```

where `c_i,t` is the communication token selected by predator `i`. The penalty applies only to predators with communication enabled.

Plot:

```bash
python -m onpolicy.scripts.eval.plot_mpe_comm_activity_sweeps
```

## 4. Dimension x Penalty Ablation

Purpose: test how communication vocabulary size interacts with communication cost.

Main script:

```bash
bash onpolicy/scripts/train_mpe_scripts/train_mpe_comm_step2_dim_lambda_ablation.sh
```

Default grid:

```text
comm_dim in {2, 8, 32}
lambda in {0, 0.5, 1.0}
```

## 5. Communication Content Probing

Purpose: test whether communication tokens encode task-relevant state information.

Main script:

```bash
bash onpolicy/scripts/train_mpe_scripts/run_mpe_probe_suite.sh
```

Default tasks:

```text
per-predator:
  sheep_rel_xy
  quadrant
  is_self_closest

joint:
  closest_wolf_id
```

The probing pipeline collects message/state pairs, trains linear probes, summarizes scores, computes information-theoretic diagnostics, and plots figures.

## 6. Role Analysis

Purpose: test whether predators form behavioral roles and whether those roles depend on communication.

Representative scripts:

```bash
python -m onpolicy.scripts.eval.collect_role_trajectories
python -m onpolicy.scripts.analysis.role_analysis
python -m onpolicy.scripts.analysis.plot_role_occupancy_cross_model
```

Representative outputs:

```text
onpolicy/scripts/results/MPE/simple_tag/mappo/base_reward_role_analysis/
```

## Notes

- Training can be expensive because most scripts use `num_env_steps=10000000`.
- To run quick smoke tests, override script environment variables such as `NUM_ENV_STEPS`, `SEEDS`, or `FINAL_EVAL_EPISODES`.
- Fixed-opponent evaluations use both random and heuristic prey policies.
- Most project experiments use `--comm_target adversaries`, so only the three predator agents communicate.
