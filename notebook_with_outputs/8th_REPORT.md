# Stage 8 report - non-SSL ConvNeXt-UNet pair (Colab)
- run tag: stage8_nonssl | encoder: convnext_tiny.fb_in22k_ft_in1k (plain ImageNet-21k, NO SimSiam)
- members: [('n1', 0.45), ('n2', 0.58)]
- fold 0/5 | train/val 338/86 | groups 421 | max val-to-train corr 0.665
- split matches Stage 7: True
- per-source local F1: n1 0.7962, n2 0.8055, ens_n1_n2 0.8073
- best source: ens_n1_n2 | post-processing {'t_cell': 0.5, 't_seed': 0.75, 'min_area': 70, 'score_thr': 0.0}
- RLE parity delta: 0.00000
- test predictions/image: mean 111.3 (min 12, max 639), fallbacks 0
- total elapsed: 0h 54m 54s

## What Stage 9 does with this
These maps are the NON-SSL family. Stage 9 (Kaggle, inside the Stage 7 notebook) blends
them against m1/m2/m3 and sweeps the blend weight. Both families were scored on the same
86 validation images - verify with val_ids.json before trusting any blend number.

## Reading the local number
Stages 2/4/5 fall on public = 0.773 * local - 0.111. Stage 7 (SSL) landed +0.016 above it,
half of Stage 6's +0.033, so the SSL bonus is shrinking as local F1 rises. Treat this
non-SSL model as sitting ON the line: local 0.82 projects to roughly 0.523.
Public differences under ~0.008 are inside the noise of a 91-image split.