# Stage 10 report - seed-erosion probe
- run tag: stage10_erosion | encoder: convnext_tiny.fb_in22k_ft_in1k (plain ImageNet-21k)
- all probes at seed 43, crop (400, 512) - erosion is the only variable
- fold 0/5 | train/val 338/86 | groups 421 | max val-to-train corr 0.665

## Probes
    member  erosion  seed_pct  quick_f1   tta_f1       mean  fallbacks
       e70     0.70  1.561275  0.792441 0.806190 110.032967          0
       e65     0.65  1.968103  0.788588 0.804938 108.961538          0
       e80     0.80  0.778643  0.790789 0.791371 102.510989          0
       e45     0.45  4.169578  0.774600 0.790773 113.747253          0

## Erosion curve (coarse sweep)
    member  erosion  coarse_f1  t_cell  t_seed  min_area  mean_pred  controlled
       e45     0.45   0.793267    0.55    0.70        90 111.139535        True
        n1     0.45   0.795356    0.45    0.70        90 113.453488       False
        n2     0.58   0.803469    0.50    0.60        90 108.372093       False
       e65     0.65   0.804873    0.50    0.55        90 108.255814        True
       e70     0.70   0.808552    0.45    0.65        90 108.395349        True
       e80     0.80   0.803889    0.55    0.55        90 103.639535        True

- selected members: ['e70']
- chosen post-processing: {'t_cell': 0.5, 't_seed': 0.65, 'min_area': 90, 'score_thr': 0.0}
- local F1: 0.8072
- RLE parity delta: 0.00000
- test predictions/image: mean 107.4 (min 13, max 605), fallbacks 0
- GT mean instances/image: 125.2
- total elapsed: 1h 27m 33s

## Prior points
| stage | local | public |
|---|---|---|
| 5  single ImageNet U-Net | 0.8279 | 0.5293 |
| 7  SSL x3 ensemble       | 0.8035 | 0.5259 |
| 8  non-SSL n1+n2         | 0.8073 | 0.5294 |
| 9  cross-family blend    | 0.8072 | 0.5286 |
| 10 erosion probe        | 0.8072 | ? |

Those four public scores span 0.0035 across four genuinely different models - that is the
noise floor of a 91-image split, not a ranking. Judge this run on the erosion curve's
shape, not on a single public score.

## Selection
Two slots. Keep the best measured file in one and something UNCORRELATED with it in the
other - the slots are scored separately and the better private one counts, so two shots
that fail differently beat two that fail together.