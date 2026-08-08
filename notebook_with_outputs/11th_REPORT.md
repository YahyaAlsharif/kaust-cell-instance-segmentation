# Stage 11 Report — Density-Targeted Augmentation

## Model
- Encoder: ConvNeXt-Tiny, ImageNet-21k pretrained
- Architecture: U-Net + foreground/seed prediction + watershed
- Seed erosion fraction: 0.70
- Seed: 43
- Crop: 400x512
- Member: aug70

## Stage 11 Changes
Stage 11 keeps the Stage 10 e70 model/training setup and changes the training-data exposure:

- D4 / 90-degree rotation augmentation
- Full-image scaling before cropping
- Density-biased scale mixture:
  - 45%: 0.68-0.82
  - 35%: 0.85-1.00
  - 20%: 1.00-1.20
- Dense-image oversampling
  - dense threshold: >= 200 GT instances
  - repeat factor: 2
- Foreground downscaling uses INTER_AREA
- Seed/discrete targets use INTER_NEAREST

Dense images in training:
- 73 / 338
- boosted training list: 484
- approximate dense sampling share: 45.2%

## Stage 10 Density Audit

The Stage 10 e70 audit showed that undercounting is concentrated in dense images:

| Density | Images | Mean F1 | Pred / GT |
|---|---:|---:|---:|
| Sparse <50 | 41 | 0.8573 | 0.9484 |
| Medium 50-199 | 28 | 0.7569 | 0.9860 |
| Dense 200-399 | 11 | 0.7792 | 0.9036 |
| Very dense >=400 | 6 | 0.7518 | 0.8703 |

This motivated Stage 11's density-targeted augmentation strategy.

## Validation Result
- Peak validation F1: 0.8019
- Selected plateau F1: 0.8012
- Selected post-processing:
  - t_cell: 0.5
  - t_seed: 0.55
  - min_area: 90
  - score_thr: 0.0

## Test Prediction Statistics
- Mean predictions/image: 105.7
- Minimum: 14
- Maximum: 588
- Fallbacks: 0

## Reference
Stage 10 e70:
- local plateau F1: 0.8072
- local peak F1: 0.8086
- public score: 0.5295

Stage 11 tests whether deliberately exposing the model to larger portions of dense microscopy
fields improves dense-image recall beyond the Stage 10 seed-target optimum.

## Output
- Submission: `/kaggle/working/11th_submission.csv`
- Report: `/kaggle/working/11th_REPORT.md`
