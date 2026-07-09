# Run Summary

## Goal

Fine-tune StarVLA PI0.5 on the LIBERO task distribution and evaluate whether the resulting policy can perform simulated manipulation tasks, with special attention to visible behavior in rollout videos.

## Training Configuration

| Field | Value |
| --- | --- |
| Run ID | `pi05_libero_full_4h100` |
| Model | StarVLA PI0.5 |
| Initial checkpoint | PI0.5 base weights converted for StarVLA |
| Dataset | LIBERO all-suite mix |
| Hardware | 4 x H100 |
| Total steps | 120,000 |
| Checkpoint interval | 30,000 steps |
| Per-device batch size | 1 |
| Final train loss observed | 0.0466663 |

Training completed successfully at `120000/120000` with checkpoints at 30k, 60k, 90k, and 120k/final. Model checkpoints are not included in this repository because they are large and depend on upstream model licensing.

## Evaluation Configuration

The post-training automatic evaluation attempted to run with `bfloat16`, but failed with a matrix dtype mismatch. Final evaluation was rerun in `float32`.

The final report covers four LIBERO suites:

- `libero_goal`
- `libero_spatial`
- `libero_object`
- `libero_10`

Each suite was evaluated for 50 episodes.

## Artifacts

Final result JSONs are in `results/final_eval/`.

The final success videos are in `videos/success/`. These are intentionally small and included directly in git so the repository remains immediately useful when shared.

Checkpoint probe videos are in `videos/checkpoint_probe/`.

Visual analysis outputs are in `analysis/` and `figures/`.
