# Stage 2 Run Report

- Architecture: v2
- Weights: coco
- Mask loss: bce+dice
- Official metric: per-image instance F1 at Dice ≥ 0.5
- Train/validation images: 360/64
- Detection capacity: 850
- Anchors: ((20,), (24,), (28,), (40,), (92,))
- Anchor scale correction: 1.523
- Best checkpoint F1 during training: 0.7796296060583032
- Best global sweep F1: 0.790872
- Global post-processing: {'mask_thr': 0.5, 'min_area': 103, 'allow_overlap': False, 'score_thr': 0.26}
- Cluster post-processing used: False
- Cluster validation F1: 0.7916145402701842
- Test predictions/image: {'mean': 101.46153846153847, 'min': 16, 'max': 476}
- Fallback images: 0
- Total elapsed: 1h 22m

## Submit

`/kaggle/working/submission.csv`