# Evaluation Analysis

## Final Suite-Level Results

| Suite | Successes | Episodes | Success rate |
| --- | ---: | ---: | ---: |
| LIBERO Goal | 11 | 50 | 22.0% |
| LIBERO Spatial | 0 | 50 | 0.0% |
| LIBERO Object | 0 | 50 | 0.0% |
| LIBERO 10 | 0 | 50 | 0.0% |
| **Overall** | **11** | **200** | **5.5%** |

## LIBERO Goal Per-Task Results

| Task | Successes | Episodes | Success rate |
| --- | ---: | ---: | ---: |
| Open the middle drawer of the cabinet | 2 | 5 | 40% |
| Put the bowl on the stove | 1 | 5 | 20% |
| Put the wine bottle on top of the cabinet | 0 | 5 | 0% |
| Open the top drawer and put the bowl inside | 0 | 5 | 0% |
| Put the bowl on top of the cabinet | 0 | 5 | 0% |
| Push the plate to the front of the stove | 3 | 5 | 60% |
| Put the cream cheese in the bowl | 0 | 5 | 0% |
| Turn on the stove | 5 | 5 | 100% |
| Put the bowl on the plate | 0 | 5 | 0% |
| Put the wine bottle on the rack | 0 | 5 | 0% |

## Behavioral Interpretation

The policy shows clear target-conditioned movement and useful coarse grounding. It can often move the gripper toward the correct area and perform simple contact behaviors. The strongest final results are on tasks where the success condition can be reached through direct contact or simple object interaction:

- turning on the stove;
- pushing the plate;
- opening the middle drawer;
- placing the bowl on the stove in one successful episode.

The weak areas are consistent across the failed suites:

- grasp stability;
- object pose control after contact;
- precise final placement;
- multi-object and longer-horizon sequencing.

This explains the observed pattern: the policy often gets near the target but fails when success requires reliable manipulation after arrival.

## Why This Is Not Comparable to the Official PI0.5 Result

The upstream StarVLA README reports 96.25% average success for its reproduced PI0.5 train-and-eval run. Our 5.5% result is not a contradictory reproduction because the training semantics differ:

- this run trained with `libero_all` / `libero_franka`;
- the official PI0.5 launcher uses `openpi_libero_all` / `openpi_libero_franka`;
- this run's transform uses `min_max` only for six action fields, while the OpenPI transform uses `q99` for all state and action fields, including gripper;
- the evaluation path interpreted PI0.5 state/actions using the OpenPI `q99` convention;
- this run used global batch 4 instead of the official launcher's default global batch 64.

The evaluation sample count also differs: 50 episodes per suite here versus 500 in the official protocol. If the true success rate were 96.25%, 50 episodes would have a standard error of about 2.7 percentage points, compared with about 0.85 points for 500 episodes. That changes confidence by a few points; it does not explain 11 successes out of 200.

## Why Some Tasks Still Succeeded

The mismatch does not erase all pretrained visual and language capability. The policy can still identify the instructed object or region and produce roughly correct motion directions. Fine control is more sensitive to action scale, orientation, and gripper timing, so coarse target-reaching can survive while grasping and placement fail.

This matches the successes observed in this run:

- turn on the stove: 5/5;
- push the plate: 3/5;
- open the middle drawer: 2/5;
- put the bowl on the stove: 1/5.

These tasks tolerate direct contact or a wider completion region. Tasks requiring a stable grasp, precise pose, or multiple stages remained at 0/5. The distribution supports the normalization-mismatch diagnosis, but does not by itself prove causality.

## Why Videos Matter Here

The official simulator predicate is the final binary score. However, binary success can miss partial progress, especially when a rollout reaches the target, makes contact, or moves the relevant object but does not satisfy the exact final predicate. For that reason, this repository includes rollout videos and visual proxy metrics alongside official JSON results.

The video-derived metrics are diagnostic only. They should not be added to the official success count or presented as task success.
