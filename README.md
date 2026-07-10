# StarVLA PI0.5 LIBERO Training Report

This repository packages a completed StarVLA PI0.5 training and evaluation run on LIBERO. It is meant as a compact, shareable record of the run: configuration, final metrics, rollout videos, checkpoint probes, and a small visual proxy analysis.

> [!IMPORTANT]
> This run completed all 120,000 training steps, but it is **not a strict reproduction of the official StarVLA PI0.5/OpenPI LIBERO recipe**. The actual run used `libero_all` / `libero_franka`; the official PI0.5 launcher uses `openpi_libero_all` / `openpi_libero_franka`. Those configurations apply materially different state/action normalization. The 5.5% result below therefore describes this adapted run and must not be presented as official StarVLA PI0.5 performance.

Model weights, cloud credentials, access tokens, SSH keys, full logs, and raw training datasets are intentionally not included.

## Quick Result

| Suite | Successes | Episodes | Success rate |
| --- | ---: | ---: | ---: |
| LIBERO Goal | 11 | 50 | 22.0% |
| LIBERO Spatial | 0 | 50 | 0.0% |
| LIBERO Object | 0 | 50 | 0.0% |
| LIBERO 10 | 0 | 50 | 0.0% |
| **Overall** | **11** | **200** | **5.5%** |

The adapted policy learned useful coarse grounding and target-reaching behavior. It performed best on contact-oriented tasks such as opening a drawer, pushing a plate, and turning on a stove, but remained weak on precise grasping, stable placement, and long-horizon multi-object manipulation.

## Run Setup

| Item | Value |
| --- | --- |
| Model family | StarVLA PI0.5 |
| Actual data mix | `libero_all` |
| Actual robot/data config | `libero_franka` |
| Hardware | 4 x H100 |
| Training length | 120,000 steps |
| Global batch size | 4 |
| Checkpoints inspected | 30k, 60k, 90k, 120k/final |
| Final evaluation dtype | float32 |
| Evaluation scale | 50 episodes/suite |

The original post-training automatic evaluation path failed under `bfloat16` because of a dtype mismatch. The final reported metrics come from a replacement `float32` evaluation.

## Videos

Representative final successes:

- [Open middle drawer, episode 02](videos/success/libero_goal_task00_ep02_success_open_the_middle_drawer_of_the_cabinet.mp4)
- [Push plate to front of stove, episode 00](videos/success/libero_goal_task05_ep00_success_push_the_plate_to_the_front_of_the_stove.mp4)
- [Turn on stove, episode 00](videos/success/libero_goal_task07_ep00_success_turn_on_the_stove.mp4)
- [All final success videos](videos/success/)

Checkpoint probe videos:

- [30k drawer probe](videos/checkpoint_probe/pi05_ckpt30000_libero_goal_task00_ep00_fail_open_middle_drawer.mp4)
- [60k drawer probe](videos/checkpoint_probe/pi05_ckpt60000_libero_goal_task00_ep00_fail_open_middle_drawer.mp4)
- [90k drawer probe](videos/checkpoint_probe/pi05_ckpt90000_libero_goal_task00_ep00_fail_open_middle_drawer.mp4)
- [90k drawer probe with simulator-instrumented logging](videos/checkpoint_probe/pi05_ckpt90000_libero_goal_task00_ep00_quant_open_middle_drawer.mp4)

Reference baseline artifact:

- [Earlier successful drawer rollout](videos/baseline/rollout_open_the_middle_drawer_of_the_cabinet_episode0_success.mp4)

## What Is Included

- `results/final_eval/`: final LIBERO evaluation JSONs.
- `results/checkpoint_eval/`: 30k/60k/90k single-task probe JSONs and simulator-instrumented probe output.
- `videos/success/`: final successful LIBERO Goal rollouts.
- `videos/checkpoint_probe/`: checkpoint progression probe videos.
- `figures/`: contact sheets and summary plots.
- `analysis/`: visual proxy metrics used to compare success and failure behavior.
- `docs/`: detailed run notes and evaluation interpretation.

## Official Recipe Comparison

The upstream PI0.5 launcher defaults to `openpi_libero_all`, 8 processes, and a per-device batch size of 8, for a global batch size of 64. Its OpenPI data config applies `q99` normalization to all state and action fields, including the gripper. This run used a global batch size of 4 and the legacy `libero_franka` transform, which applies `min_max` normalization only to the six Cartesian/rotation action fields and does not apply the same `q99` state transform.

The upstream README reports 96.25% average success for its reproduced PI0.5 train-and-eval result. That number was measured with the official OpenPI configuration and 500 trials per suite. Our 50 trials per suite produce a noisier estimate, but sample count alone cannot explain a gap from roughly 96% to 5.5%. The state/action normalization mismatch is the strongest diagnosis, followed by the much smaller global batch. A strict rerun or controlled ablation is still required to measure each factor independently.

See [Official recipe gap](docs/official_recipe_gap.md) for the exact comparison and source links.

## Main Takeaway

This adapted policy is not a strong general LIBERO policy, but its behavior is not random. Visual/language grounding and coarse target-reaching can survive a control-space mismatch, while precise action magnitude, orientation, and gripper timing degrade. That pattern is visible in the task distribution: contact-oriented tasks produced some successes, while grasping, placement, and multi-step tasks largely failed. On the drawer task, the 90k checkpoint produced a visible target approach and pulling motion even when the official binary task predicate marked the episode as failed.

See:

- [Run summary](docs/run_summary.md)
- [Evaluation analysis](docs/eval_analysis.md)
- [Official recipe gap](docs/official_recipe_gap.md)
- [Checkpoint progression](docs/checkpoint_progression.md)

## Notes

This repository is a report artifact, not a full reproduction package or an official benchmark reproduction. A future strict rerun should use the upstream OpenPI data config and match the official effective batch and evaluation protocol as closely as the available hardware allows.
