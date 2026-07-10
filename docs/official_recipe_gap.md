# Official PI0.5 Recipe Gap

## Bottom Line

The run in this repository completed 120,000 steps on four H100 GPUs, but it did not use the exact data semantics of the official StarVLA PI0.5/OpenPI LIBERO recipe. Its 5.5% evaluation result belongs to this adapted configuration and is not an estimate of official StarVLA PI0.5 performance.

## Side-by-Side Comparison

| Item | This run | Official StarVLA PI0.5 launcher |
| --- | --- | --- |
| Framework | `PI05` | `PI05` |
| Base checkpoint | Converted PI0.5 base | Converted PI0.5 base |
| Training steps | 120,000 | 120,000 |
| Save interval | 30,000 | 30,000 |
| Learning rate | `5e-5` | `5e-5` |
| Hardware/processes | 4 H100 / 4 processes | 8 processes by default |
| Per-device batch | 1 | 8 |
| Global batch | 4 | 64 |
| Data mix | `libero_all` | `openpi_libero_all` |
| Robot/data config | `libero_franka` | `openpi_libero_franka` |
| State normalization | No OpenPI `q99` state transform | `q99`, all state fields |
| Action normalization | `min_max`, six pose fields | `q99`, all fields including gripper |
| Evaluation episodes | 50 per suite | 500 per suite |

## Why the Data Config Matters

The two data mixes reference the same four LIBERO datasets, but they do not define the same learning problem. `libero_franka` converts and normalizes only x/y/z/roll/pitch/yaw actions with `min_max`. `openpi_libero_franka` converts both state and action, applies `q99` normalization to every state field, and applies `q99` normalization to every action field including the gripper.

The PI0.5 evaluation path uses OpenPI-style quantile normalization for state input and action unnormalization. A model trained against the legacy transform can therefore receive state values and emit action values under a different convention at evaluation time. Rough action direction may remain useful, while magnitude, orientation, and gripper timing become unreliable.

## Expected Impact

The likely impact, from largest to smallest, is:

1. State/action preprocessing and normalization mismatch.
2. Global batch 4 versus the launcher's default global batch 64.
3. Evaluation sample size 50 versus 500 episodes per suite.
4. Float32 evaluation versus the documented mixed-precision setup.

The first item is sufficient to plausibly cause a severe control failure and matches the observed behavior. The second can affect optimization stability and comparability. The third mainly widens statistical uncertainty and cannot explain a drop of roughly 90 percentage points. The fourth is unlikely to explain the gap on its own.

## Evidence and Limits

The official StarVLA OpenPI README reports a 96.25% average for its reproduced PI0.5 train-and-eval result. Under a hypothetical true rate of 96.25%, a 50-episode suite would be expected to yield roughly 48 successes; its approximate 95% sampling margin is about 5.3 percentage points. This run instead produced 11/50 on Goal and 0/50 on each other suite.

This makes evaluation sampling an inadequate explanation. The configuration mismatch is the strongest code-level diagnosis, but exact attribution requires a controlled experiment.

## Required Strict Rerun

A comparable rerun should:

1. Use `openpi_libero_all` and verify that every dataset resolves to `openpi_libero_franka` before step 1.
2. Verify `q99` state/action statistics, including gripper, in saved configuration and a sampled batch.
3. Match global batch 64 with per-device batch size and/or gradient accumulation if memory allows.
4. Keep the official PI0.5 base checkpoint, 120,000 steps, 10,000 warmup steps, and 30,000-step save interval.
5. Run a short rollout sanity check before committing to the full train.
6. Evaluate 10 tasks x 50 episodes for each suite for final reporting.

## Upstream Sources

- [Official PI0.5 LIBERO launcher](https://github.com/starVLA/starVLA/blob/starVLA_dev/examples/simBenchmarks/LIBERO/train_files/openpi/run_pi05_libero_8gpu.sh)
- [Official OpenPI/PI0.5 training notes and result table](https://github.com/starVLA/starVLA/blob/starVLA_dev/examples/simBenchmarks/LIBERO/train_files/openpi/README.md)
- [LIBERO data registry and transforms](https://github.com/starVLA/starVLA/blob/starVLA_dev/examples/simBenchmarks/LIBERO/train_files/data_registry/data_config.py)
- [LIBERO evaluation protocol](https://github.com/starVLA/starVLA/blob/starVLA_dev/examples/simBenchmarks/LIBERO/README.md)
