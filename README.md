# Utrom

A general-purpose humanoid platform built on the Unitree G1, trained from scratch with PPO and a Transformer policy.

## Demo

<!-- Drag-drop your demo video into a GitHub issue or release, then paste the resulting URL here -->
<video src="DEMO_VIDEO_URL" controls width="100%"></video>

The robot above is v8 — 51M training steps, walking with friction domain randomization and push recovery on flat ground. Trained solo on a single Colab GPU.

## What this is

Utrom is an end-to-end humanoid locomotion stack: simulation environment, PPO trainer, Transformer policy, reward shaping, and domain randomization. The goal of the project is a general-purpose humanoid with onboard reasoning — a long-horizon research effort. v8 is the current locomotion baseline.

## Architecture

- **Simulator:** MuJoCo, Unitree G1 (23 DOF — locked wrists)
- **Policy:** Transformer encoder over 8-frame observation history. 256 embedding dim, 3 layers, 4 heads. ~2.3M parameters.
- **Algorithm:** PPO with GAE, separate MLP critic, KL-based brake on policy updates
- **Control:** Position actuators with per-joint PD gains, action delay, friction domain randomization (0.6–1.2× scale), random torso pushes (50–150N) for robustness
- **Training:** 32 parallel envs, 2048 steps per rollout, on a single A100/L4

## Results

| Version | Mean Reward | Steps | Key Changes |
|---------|-------------|-------|-------------|
| v1 | 4827 ± 266 | 39M | Baseline — PPO + Transformer from scratch |
| v2 | 4274 ± 478 | 30M from v1 | Foot impact penalty, torso wobble −58% |
| v3 | 5463 ± 131 | 1M from v2 | Arm swing reward, tighter velocity range |
| v4 | 4167 ± 487 | 13M from scratch | Hard hip yaw constraint, heading + drift penalties |
| v5 | 7008 ± 141 | 30M from scratch | Arm swing ×4, torso pitch penalty, phase-gated energy penalties |
| v6 | 7733 ± 79 | 63M from scratch | Torso yaw damping, angular momentum penalty, sharpened vx tracking |
| v7b | 7200 ± 524 | 34M from v6 | Push recovery, foot symmetry penalty (best episode: 7747) |
| **v8** | **7061 ± 406** | **51M from v7b** | **Friction domain randomization, decisive gait, tighter variance** |

Reward numbers are mean ± std over training-time evaluation episodes. Higher is better.

## Methodology highlights

**Reward shaping evolution.** The reward function went through eight major iterations covering velocity tracking, foot air time, gait alternation, jerk penalties, torso wobble, foot impact, arm swing, force spikes, energy efficiency, lateral drift, heading alignment, torso yaw damping, angular momentum, elbow extension, shoulder roll, push recovery, and foot symmetry. Each version's changes were targeted at a specific behavioral failure observed in the previous demo video.

**Domain randomization.** Friction is randomized 0.6–1.2× per episode. Random 50–150N pushes are applied to the torso every 100–300 steps with 30% probability during PHASE_FULL. Hip yaw joints are clamped to ±10° to enforce forward-facing locomotion.

**Phase curriculum.** Training proceeds through PHASE_STAND (zero command) → PHASE_SLOW (fixed slow walk) → PHASE_FULL (random commands across vx ∈ [0.3, 1.5], vy ∈ [-0.3, 0.3], yaw ∈ [-0.5, 0.5]). Phase transitions are triggered by reward thresholds.

**Action space.** 29-dim continuous action representing residual joint angle offsets from a default standing pose. Actions are scaled by 0.5 and added to the default pose to produce position actuator targets.

## About

Solo project. The full implementation (training code, environment, reward function, hyperparameters) is in a private repository. This public repo exists to document the work for those interested in humanoid RL.



---

Built on the [Unitree G1](https://www.unitree.com/g1) platform.
