# Stage 6 report — SSL + Mean Teacher ConvNeXt-UNet watershed
- encoder: convnext_tiny.fb_in22k_ft_in1k
- unlabeled images used: 1972
- selected checkpoint source: supervised
- supervised quick grouped F1: 0.7539
- best semi-supervised quick grouped F1: 0.7539
- final TTA + tuned watershed grouped F1: 0.7740
- post-processing: {'t_cell': 0.5, 't_seed': 0.65, 'marker_mode': 'cc', 'min_distance': 0, 'min_area': 71}
- train/val images: 339/85 | groups: 424
- max val-to-train thumbnail correlation: 0.553
- test predictions/image: mean 102.9 (min 6, max 569)
- fallbacks: 0
- total elapsed: 1h 07m 52s

## Interpretation
The selected checkpoint is chosen only from grouped validation. The unlabeled data affect the model through
self-supervised domain adaptation and conservative EMA teacher consistency. Hard pseudo-label polishing is
disabled by default because it is higher risk and should only be accepted after a demonstrated validation gain.