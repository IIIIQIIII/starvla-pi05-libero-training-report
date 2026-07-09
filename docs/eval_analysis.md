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

## Why Videos Matter Here

The official simulator predicate is the final binary score. However, binary success can miss partial progress, especially when a rollout reaches the target, makes contact, or moves the relevant object but does not satisfy the exact final predicate. For that reason, this repository includes rollout videos and visual proxy metrics alongside official JSON results.
