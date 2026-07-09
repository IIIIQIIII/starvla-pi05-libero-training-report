# StarVLA PI0.5 LIBERO Training Report

This repository packages a completed StarVLA PI0.5 fine-tuning and evaluation run on LIBERO. It is meant as a compact, shareable record of the run: configuration, final metrics, rollout videos, checkpoint probes, and a small visual proxy analysis.

Model weights, cloud credentials, access tokens, SSH keys, full logs, and raw training datasets are intentionally not included.

## Quick Result

| Suite | Successes | Episodes | Success rate |
| --- | ---: | ---: | ---: |
| LIBERO Goal | 11 | 50 | 22.0% |
| LIBERO Spatial | 0 | 50 | 0.0% |
| LIBERO Object | 0 | 50 | 0.0% |
| LIBERO 10 | 0 | 50 | 0.0% |
| **Overall** | **11** | **200** | **5.5%** |

The final policy learned useful coarse grounding and target-reaching behavior. It performed best on contact-oriented tasks such as opening a drawer, pushing a plate, and turning on a stove, but remained weak on precise grasping, stable placement, and long-horizon multi-object manipulation.

## Run Setup

| Item | Value |
| --- | --- |
| Model family | StarVLA PI0.5 |
| Dataset mix | LIBERO all-suite mix |
| Hardware | 4 x H100 |
| Training length | 120,000 steps |
| Checkpoints inspected | 30k, 60k, 90k, 120k/final |
| Final evaluation dtype | float32 |

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

## Main Takeaway

The trained PI0.5 policy is not yet a strong general LIBERO policy, but the behavior is not random. On the drawer task, the 90k checkpoint produced visually correct target approach and drawer-pulling motion even when the official binary task predicate marked the episode as failed. This is why the repository includes both official success rates and video-derived proxy metrics.

See:

- [Run summary](docs/run_summary.md)
- [Evaluation analysis](docs/eval_analysis.md)
- [Checkpoint progression](docs/checkpoint_progression.md)

## Notes

This repository is a report artifact, not a full reproduction package. To reproduce training, use the upstream StarVLA codebase, obtain the required model and dataset access, and adapt the configuration summarized in `docs/run_summary.md`.
