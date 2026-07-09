# Checkpoint Progression

The 30k, 60k, and 90k checkpoints were probed on the same drawer-opening task:

`open the middle drawer of the cabinet`

Official one-episode probe results were all `0/1`, but the rollout videos show meaningful behavioral differences across checkpoints.

## Visual Proxy Summary

These metrics are image-space proxies computed from fixed regions of interest in 256 x 256 rollout videos. They are not official LIBERO simulator predicates.

| Rollout | Drawer/cabinet dark-area delta | Cabinet changed-pixel peak | Min gripper-to-handle proxy distance |
| --- | ---: | ---: | ---: |
| Reference success | +96.1% | 1,785 px | 55.1 px |
| PI0.5 30k probe | +5.9% | 188 px | 61.1 px |
| PI0.5 60k probe | +6.8% | 1,232 px | 57.8 px |
| PI0.5 90k probe | +107.3% | 2,042 px | 57.7 px |

## Interpretation

The 30k checkpoint mostly approaches but does not cause much task-relevant change.

The 60k checkpoint shows more scene change around the cabinet region, but still lacks reliable drawer opening.

The 90k checkpoint produces a drawer/cabinet visual change comparable to the reference success video, even though the official simulator predicate still marks the episode failed. This supports the qualitative observation that the policy learned a task-relevant drawer-pulling behavior before it learned robust final-state completion.

## Figures

- [Checkpoint progression plot](../figures/checkpoint_progression_plot.jpg)
- [Success vs failure comparison plot](../figures/comparison_plot.jpg)
- [Reference success contact sheet](../figures/success_qwen_oft_contact_sheet.jpg)
- [30k contact sheet](../figures/pi05_ckpt30000_fail_contact_sheet.jpg)
- [60k contact sheet](../figures/pi05_ckpt60000_fail_contact_sheet.jpg)
- [90k contact sheet](../figures/pi05_ckpt90000_fail_contact_sheet.jpg)
