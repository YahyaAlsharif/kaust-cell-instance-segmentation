# Stage 7 report - SSL-adapted ConvNeXt-UNet ensemble
- run tag: stage7_ssl_ensemble
- encoder: convnext_tiny.fb_in22k_ft_in1k (+ SimSiam adaptation, 25 epochs @ 288px on 2578 unlabeled images)
- members: ['m1', 'm2', 'm3'] | fold 0/5 | train/val 338/86 | groups 421
- max val-to-train correlation: 0.665
- per-member TTA val F1: m1 0.7999, m2 0.7938, m3 0.7866
- ensemble val F1 (global thresholds): 0.8035
- ensemble val F1 (per-cluster): 0.8036 -> rejected
- chosen post-processing: {'t_cell': 0.45, 't_seed': 0.65, 'min_area': 90, 'score_thr': 0.0}
- RLE parity delta: 0.00000
- test predictions/image: mean 107.8 (min 12, max 602), fallbacks 0
- cluster mixture train [0.46, 0.54] vs test [0.467, 0.533]
- total elapsed: 1h 39m 50s

## How to read the local number
Stages 2/4/5 fall on a line where public = 0.773 * local - 0.111. Stage 6, whose encoder was
SimSiam-adapted on the unlabeled set, scored 0.033 ABOVE that line. This notebook is also
SSL-adapted, so compare its local F1 against Stage 6's 0.7740 (which produced 0.5209), not
against Stage 5's 0.8279 (which produced 0.5293). A local score near 0.80 here is expected to
beat 0.5293; do not reject it for looking lower than Stage 5.

## Submit order
1. submission.csv (ensemble, global thresholds) - the primary candidate.
2. submission_ensemble_cluster.csv only if the gate above accepted it.
Keep the Stage 5 file at 0.5293 selected as one of the two final submissions until something
measurably beats it on the public leaderboard.