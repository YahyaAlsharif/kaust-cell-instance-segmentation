# Stage 5 report - ConvNeXt-UNet + watershed (Colab T4)
- encoder: convnext_tiny.fb_in22k_ft_in1k (ImageNet only; no cell/microscopy pretraining)
- paradigm: dense prediction + marker-controlled watershed (no detection cap)
- grouped split: fold 0/5 | groups 424 | max val-to-train correlation 0.572
- train/val images: 339/85
- epochs completed: 70 of 70
- best checkpoint F1: 0.8157
- swept validation F1: 0.8279
- post-processing: {'t_cell': 0.55, 't_seed': 0.65, 'min_area': 87, 'score_thr': 0.56}
- test predictions/image: mean 100.9 (min 6, max 533, fallbacks 0)
- total elapsed: 0h 54m

## Probe submissions
                                        file  score_thr  local_val_f1  mean_predictions
                              submission.csv       0.56        0.8279             100.9
submission_stage5_convnext_unet_ws_s0_46.csv       0.46        0.8279             100.9
submission_stage5_convnext_unet_ws_s0_66.csv       0.66        0.8279             100.8

Compare the honest F1 above with Stage 3's. If Stage 5 is close, it is a strong ensemble
partner because its errors are structurally different. If it is far below, submit it once
to see the public score and keep Stage 3 as the primary.