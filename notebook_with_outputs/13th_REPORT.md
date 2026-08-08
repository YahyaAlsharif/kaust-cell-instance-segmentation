# Stage 13 report — DINOv3 dense cell segmentation

- active encoder: vit_small_patch16_dinov3.lvd1689m
- intended primary: DINOv3 ViT-S/16 pretrained on generic LVD-1689M web imagery
- architecture: DINO intermediate fusion + shallow CNN detail branch + progressive decoder
- output: foreground + relative e70 seed maps -> marker-controlled watershed
- grouped split: fold 0/5 | groups 424 | train/val 339/85
- max val-to-train thumbnail correlation: 0.666
- checkpoint epoch: 50 | quick official F1: 0.7935
- final TTA swept F1: 0.7956 (peak 0.7982)
- chosen post-processing: {'t_cell': 0.5, 't_seed': 0.65, 'min_area': 90, 'score_thr': 0.0}
- test predictions/image: mean 106.6 (min 8, max 630)
- fallbacks: 0
- RLE parity delta: 0.000000
- runtime: 0h 25m 36s

## Density validation
                      images  mean_f1   mean_gt  mean_pred  mean_fn  mean_fp  pred_over_gt
    bin                                                                                   
    sparse <50            49   0.8266   27.8163    25.9184   5.4082   3.5102        0.9318
    medium 50-199         23   0.7380   97.0000   102.6957  22.9565  28.6522        1.0587
    dense 200-399          7   0.7508  295.0000   306.1429  68.2857  79.4286        1.0378
    very dense >=400       6   0.8158  499.1667   481.0000  98.8333  80.6667        0.9636

## Stage 10 reference
- local plateau F1: 0.8072
- public: 0.5295
- pred/GT: sparse 0.9484, medium 0.9860, dense 0.9036, very dense 0.8703

## Rules
Generic pretrained weights are allowed per the known guidance, but DINOv3 LVD-1689M was not specifically TA-confirmed like ImageNet/COCO.
SUBMISSION_RULES_CONFIRMED = False
Do not submit this candidate until the TA confirms that this generic DINOv3 pretrained source is acceptable.

## Outputs
- /content/13th_submission.csv
- /content/13th_REPORT.md
- Drive artifacts: /content/drive/MyDrive/ka_ss26/stage13_dinov3/artifacts