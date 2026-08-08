# Stage 2 Run Report

- Architecture: v2
- Weights: coco
- Mask loss: bce+dice
- Official metric: per-image instance F1 at Dice ≥ 0.5
- Train/validation images: 338/86
- Detection capacity: 1000
- Anchors: ((20,), (24,), (32,), (44,), (96,))
- Anchor scale correction: 1.631
- Best checkpoint F1 during training: 0.7313824256247006
- Best global sweep F1: 0.750732
- Global post-processing: {'mask_thr': 0.6, 'min_area': 103, 'allow_overlap': False, 'score_thr': 0.26}
- Cluster post-processing used: False
- Cluster validation F1: 0.7520512883576381
- Test predictions/image: {'mean': 98.94505494505495, 'min': 12, 'max': 529}
- Fallback images: 0
- Total elapsed: 2h 17m

## Submit

`/kaggle/working/submission.csv`
## Stage 3/4 additions
- run tag: stage4_imagenet_r50v2_hires
- weights: coco | arch: v2 | seed: 777
- grouped split: True | groups: 421 | fold 0/5
- max val-to-train thumbnail correlation: 0.665 (mean 0.076)
- train/val images: 338/86
- detections per image: 1000 | instance cap: None
- HONEST validation F1: 0.7507   <-- compare with Stage 2's leaky 0.7909
- chosen post-processing: {'mask_thr': 0.6, 'min_area': 103, 'allow_overlap': False, 'score_thr': 0.26}

### Probe submissions
                                               file  score_thr  local_val_f1  mean_pred
    submission_stage4_imagenet_r50v2_hires_main.csv       0.26      0.750732       98.9
    submission_stage4_imagenet_r50v2_hires_s014.csv       0.14      0.733799      116.8
    submission_stage4_imagenet_r50v2_hires_s020.csv       0.20      0.747239      106.9
    submission_stage4_imagenet_r50v2_hires_s032.csv       0.32      0.749189       92.3
    submission_stage4_imagenet_r50v2_hires_s038.csv       0.38      0.742937       85.8
submission_stage4_imagenet_r50v2_hires_minarea0.csv       0.26           NaN      131.6

Submit the ladder in ascending score_thr order, record the public score of each,
then keep the threshold at the centre of the best region rather than its peak.
