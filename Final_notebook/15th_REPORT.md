# Stage 15 FINAL report — object-centric zoom-in + continuous distance field + distance-watershed

## Formulation
- encoder: convnext_tiny.fb_in22k_ft_in1k (ImageNet-22k -> 1k), U-Net decoder (256, 128, 64, 32)
- head: 3 channels — foreground, e70 seed, normalised per-instance distance field
- object-centric training probability: 70%
- global training instance pool: 42981 cells
- centre jitter: 0.18 of native context size
- EMA decay: 0.9985
- targets: seed DERIVED by thresholding the resized distance field at 0.7
- augmentation: ZOOM-IN mixture ((0.25, 1.15, 1.35), (0.5, 1.35, 1.65), (0.25, 1.65, 1.9)), D4=True, flips, brightness/contrast/gamma/blur/noise
- crop (400, 512), batch 8, 72 epochs x 42 steps
- inference: scales (1.35, 1.65) x 4 flips, maps averaged at native resolution

## Validation
- grouped split fold 0/5 | groups 421 | train/val 338/86 | max val->train corr 0.665
- checkpoint epoch 72 | quick F1 0.7984
- decode family probe:
    marker landscape  t_marker     f1
      seed        fg      0.65 0.8185
      dist        fg      0.55 0.8170
      seed      dist      0.65 0.8079
      dist      dist      0.55 0.8015
- chosen decode: {'marker': 'seed', 'landscape': 'fg', 't_cell': 0.5, 't_marker': 0.65, 'min_area': 90}
- sweep peak 0.8145 | plateau-centre F1 0.8144
- density-conditioned t_cell: gain +0.0011, bootstrap win-rate 0.81, applied = False
                 bin  images  t_cell   base  routed
          sparse <50      45    0.45 0.8514  0.8527
       medium 50-199      25    0.50 0.7544  0.7544
       dense 200-399      12    0.60 0.7963  0.7991
    very dense >=400       4    0.50 0.8272  0.8272
- **FINAL LOCAL F1 = 0.8144**
- references: Stage 10 0.8072 (public 0.5295), Stage 12 0.8119 (public 0.5335)

## Density validation
                      images  mean_f1   mean_gt  mean_pred   mean_fn  mean_fp  pred_over_gt
    bin                                                                                    
    sparse <50            41   0.8567   27.3902    27.6585    3.9512   4.2195        1.0098
    medium 50-199         28   0.7699   91.8929    94.8214   20.4286  23.3571        1.0319
    dense 200-399         11   0.7917  320.6364   293.3636   75.9091  48.6364        0.9149
    very dense >=400       6   0.7749  476.8333   409.1667  128.3333  60.6667        0.8581

## Near-miss census (share of GT cells by best achievable Dice)
- Stage 15: {'hit>=0.50': np.float64(76.5), 'near 0.35-0.50': np.float64(9.0), 'weak 0.10-0.35': np.float64(9.1), 'missed <0.10': np.float64(5.3)}
- Stage 12 reference: hit 77.0 | near 8.6 | weak 8.3 | missed 6.1

## Submission integrity
- RLE order C | parity delta 0.000000
- predictions/image mean 109.0 (min 15, max 592)
- fallbacks 0 | all 182 test IDs present
- runtime 0h 26m 02s

## Verdict
- SUBMIT AS A DIVERSITY SLOT — comparable to Stage 12, different failure modes

## Outputs
- /content/15th_submission.csv
- /content/15th_REPORT.md
- checkpoint /content/stage15/ckpt/stage15_final_objectcentric_zoomdist_best.pt
- convenience model copy /content/stage15_selected_model.pt
- maps /content/stage15/maps