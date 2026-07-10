# Run Summary

## Goal

Run StarVLA PI0.5 training end to end on the four LIBERO suites and evaluate the resulting policy, with special attention to visible behavior in rollout videos. This was a completed training-chain experiment, not a strict reproduction of the official PI0.5/OpenPI recipe.

## Training Configuration

| Field | Value |
| --- | --- |
| Run ID | `pi05_libero_full_4h100` |
| Model | StarVLA PI0.5 |
| Initial checkpoint | PI0.5 base weights converted for StarVLA |
| Actual data mix | `libero_all` |
| Actual data/robot config | `libero_franka` |
| Datasets | LIBERO Object, Goal, Spatial, and LIBERO-10 no-noops LeRobot datasets |
| Dataset lengths | 66,984 / 52,042 / 52,970 / 101,469 |
| Hardware | 4 x H100 |
| Distributed setup | 4 processes, DeepSpeed ZeRO-2, NCCL |
| Train precision | bf16 mixed precision |
| Total steps | 120,000 |
| Checkpoint interval | 30,000 steps |
| Per-device batch size | 1 |
| Gradient accumulation | 1 |
| Global batch size | 4 |
| Optimizer | AdamW |
| Base/action-head learning rate | `5e-5` / `5e-5` |
| Gradient clipping | 1.0 |
| Trainable parameters | about 3.617B |
| Wall-clock training time | about 9h 35m |
| Final train loss observed | 0.0466663 |

Training completed successfully at `120000/120000` with checkpoints at 30k, 60k, 90k, and 120k/final. Model checkpoints are not included in this repository because they are large and depend on upstream model licensing.

The final loss is a single logged action-DiT training-loss sample. It confirms that optimization ran, but it is not evidence of task success or recipe correctness.

## Critical Configuration Difference

The official `run_pi05_libero_8gpu.sh` uses `openpi_libero_all` / `openpi_libero_franka`. This run used `libero_all` / `libero_franka` after the nested LIBERO data registry was not discovered through the path used when the run was launched.

Although both mixes point to the same four underlying LIBERO datasets, their transforms are different:

| Field | This run | Official PI0.5/OpenPI recipe |
| --- | --- | --- |
| Data mix | `libero_all` | `openpi_libero_all` |
| Data config | `libero_franka` | `openpi_libero_franka` |
| State transform | No equivalent `q99` normalization | `q99` on all state fields |
| Action transform | `min_max` on x/y/z/roll/pitch/yaw | `q99` on all action fields, including gripper |
| Global batch | 4 | 64 by launcher default |

This is a control-semantics difference, not merely a naming difference. The evaluation path used PI0.5/OpenPI `q99` normalization and unnormalization, so the model's training input/target distribution did not match the evaluation convention. That mismatch is the leading explanation for the very low final success rate. It is plausible and consistent with the rollouts, but only an official-config rerun or a controlled ablation can establish its exact numerical impact.

## Evaluation Configuration

The post-training automatic evaluation attempted to run with `bfloat16`, but failed with a matrix dtype mismatch. Final evaluation was rerun in `float32`.

The final report covers four LIBERO suites:

- `libero_goal`
- `libero_spatial`
- `libero_object`
- `libero_10`

Each suite was evaluated for 50 episodes: 10 tasks x 5 episodes. The official StarVLA LIBERO reporting protocol uses 500 episodes per suite: 10 tasks x 50 episodes. The smaller sample makes our rates less precise, but it cannot plausibly explain the observed gap by itself.

## Artifacts

Final result JSONs are in `results/final_eval/`.

The final success videos are in `videos/success/`. These are intentionally small and included directly in git so the repository remains immediately useful when shared.

Checkpoint probe videos are in `videos/checkpoint_probe/`.

Visual analysis outputs are in `analysis/` and `figures/`.
