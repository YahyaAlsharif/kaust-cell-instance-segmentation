# Stage 9 report - cross-family blend (SSL x non-SSL)
- SSL members (Stage 7): ['m1', 'm2', 'm3'] | non-SSL members (Stage 8): ['n1', 'n2']
- validation identity verified against Stage 8 val_ids.json: 86 images, identical
- blend weight w = 0.50  (w = non-SSL share; 0.0 = pure Stage 7, 1.0 = pure Stage 8)
- blend-weight table:
      w       f1  t_cell  t_seed  min_area  mean_pred
    0.5 0.814394    0.50    0.65        90 116.800000
    0.6 0.814305    0.50    0.70        90 116.911111
    0.7 0.814153    0.55    0.70        90 115.977778
    0.4 0.813390    0.50    0.65        90 116.733333
    0.3 0.812483    0.45    0.65        90 117.311111
    1.0 0.810416    0.55    0.70        90 116.622222
    0.0 0.810157    0.45    0.65        90 117.688889
- chosen post-processing: {'t_cell': 0.5, 't_seed': 0.7, 'min_area': 70, 'score_thr': 0.0}
- local F1 (global thresholds): 0.8072
- local F1 (density-adaptive): 0.8121 -> rejected (+0.005 gate)
- RLE parity delta: 0.00000
- test predictions/image: mean 111.1 (min 11, max 629), fallbacks 0

## Prior points
| stage | local | public |
|---|---|---|
| 5 (ImageNet ConvNeXt-UNet) | 0.8279 | 0.5293 |
| 6 (SSL, starved schedule)  | 0.7740 | 0.5209 |
| 7 (SSL x3 ensemble)        | 0.8035 | 0.5259 |
| 9 (blend, w=0.50)        | 0.8072 | ? |

Line through stages 2/4/5: public = 0.773 * local - 0.111. Stage 7 sat +0.016 above it,
Stage 6 +0.033 - the SSL residual shrinks as local F1 rises, so do not credit this blend
with a bonus it has not earned. Read it as sitting on or just above the line.

## Submission order
1. /kaggle/working/submission.csv - the blend at global thresholds.
2. submission_stage9_density.csv only if the +0.005 gate accepted it.

## Final selection
Keep 5th_submission.csv (0.5293) as one of the two slots throughout - it lives on Kaggle's
side and nothing lost locally affects it. The second slot goes to this blend even if it
measures within ~0.005 of 0.5293: at 91 public images that gap is under one standard error,
and a two-family blend has the better expected private score than either family alone.