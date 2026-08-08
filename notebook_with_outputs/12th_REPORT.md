# Stage 12 Report — e70 × aug70 Diverse Map Ensemble

## Strategy

Stage 12 is a zero-GPU probability-map ensemble between two ConvNeXt-Tiny
U-Net models trained on different data distributions.

### Stage 10 e70
- Standard Stage 10 training distribution
- Relative seed erosion: 0.70
- Public score: 0.5295
- Selected local F1: 0.8072
- Peak local F1: 0.8086

### Stage 11 aug70
- D4 / 90-degree rotations
- Full-image scaling before cropping
- Density-biased scale augmentation
- Dense-image oversampling
- Relative seed erosion: 0.70
- Selected local F1: 0.8012
- Peak local F1: 0.8019

Stage 11 was weaker independently, but its different training distribution
provides complementary probability maps.

## Blend Search

Weight definition:

    blended = (1-w) * e70 + w * aug70

Best F1 by weight:

w
0.00    0.808552
0.25    0.809799
0.35    0.811945
0.50    0.810615
0.65    0.808394
0.75    0.807408
1.00    0.801852

The weight axis has an interior optimum rather than being flat.

## Selected Ensemble

- aug70 weight: 0.35
- e70 weight: 0.65
- t_cell: 0.5
- t_seed: 0.55
- min_area: 90
- score_thr: 0.0

### Validation

- Peak Stage 12 F1: 0.811945
- Selected plateau F1: 0.811945
- Stage 10 reference: 0.8072
- Improvement over Stage 10: +0.004745

The experiment passed the pre-declared Stage 12 acceptance gate of
more than +0.004 over Stage 10.

## Near-Miss Diagnostic

Using Stage 10 e70 predictions:

- overall hit >=0.50: 77.0%
- overall near 0.35-0.50: 8.6%
- overall weak 0.10-0.35: 8.3%
- overall missed <0.10: 6.1%

Near-miss share by density:

- sparse: 3.0%
- medium: 9.1%
- dense: 8.4%
- very dense: 10.7%

This indicates that part of the remaining error, especially in medium and
very-dense images, consists of detected cells whose mask Dice lies just
below the competition's 0.50 matching threshold.

## Test Prediction Statistics

- Mean predictions/image: 106.9
- Minimum: 14
- Maximum: 597
- Fallbacks: 0

## Submission Integrity

- RLE masks checked: 2412
- RLE mismatches: 0
- All 182 test IDs present
- Unique submission index
- No null RLE annotations

## Output

- `/kaggle/working/12th_submission.csv`
- `/kaggle/working/12th_REPORT.md`
