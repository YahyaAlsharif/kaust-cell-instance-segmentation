# Run report
- smoke_test: True | arch: v2 | weights: coco | mask_loss: bce+dice
- csv columns: ['Unnamed: 0', 'id', 'annotation'] | cell_type present: False
- rle order: C | GT overlap images: 91.7%
- images: 424 train / 182 test / 1972 unlabeled
- instances/img p50/p95/p99/max: 48/504/593/782
- area p1/p5/p50/p95: 53/82/215/920
- box side p5/p50/p95: 11/19/54
- anchors: ((12,), (16,), (20,), (28,), (60,)) ratios (0.62, 1.0, 1.58) | det/img 850
- dataset mean/std: 0.5018/0.0548
- clusters k=2 silhouette {2: 0.4203592958718911, 3: 0.3648181272799571, 4: 0.358939458950008, 5: 0.33039122105720714}
- best val sum_over_max: 0.0004810721634692943
- best post-processing: {'mask_thr': 0.4, 'min_area': 82, 'score_thr': 0.3}
- val at best: sum_over_max=0.0017 pq_like=0.0009 f1=0.0034
- test predictions/img: {'mean': 110.5989010989011, 'min': 0, 'max': 341}

## Open questions
1. Official Dice@0.5 aggregation (paste the reference implementation).
2. Is the COCO-pretrained detector permitted, or ImageNet backbone only?
3. Submission column schema — confirmed against the leaderboard?