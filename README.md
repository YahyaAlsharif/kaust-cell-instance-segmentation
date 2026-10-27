# Cell Instance Segmentation — KAUST Academy Summer School 2026, Challenge 2

**3rd place / 24 teams** on the final private leaderboard of *KA SS 26: Challenge 2* (detect and segment individual neuronal cells in microscopy images).

| | |
|---|---|
| Final rank | **3rd of 24 teams** (52 entrants, 234 submissions) |
| Team private LB | **0.5472** |
| This notebook (Stage 15) | private 0.5463, public 0.5345 |
| Local grouped-validation F1 | **0.8144** |
| Final model | Single ConvNeXt-Tiny U-Net, no ensemble |
| Final run time | ~26 minutes end to end on one T4 |

Public Kaggle notebook: https://www.kaggle.com/code/ghostylicious/3rd-place-object-centric-convnext-unet-distance

---

## Overview

This repository is the full record of a 30-hour instance-segmentation competition, run overnight from Thursday to Friday: not only the notebook that placed 3rd, but the fifteen numbered stages that led to it, each with the report it produced.

The final entry was submitted by a two-person team. This repository documents the solution line I designed, ran and evaluated, which is the line that produced both of our selected submissions.

The short version of the story: a Mask R-CNN baseline got us to ~0.50 on the public leaderboard, a ConvNeXt-U-Net with marker-controlled watershed decoding got us to ~0.53, and roughly ten stages of measurement, failed ideas and one target reformulation moved that to 0.5345 public / 0.5472 private. The single biggest gain in the second half did not come from a bigger model or an ensemble. It came from changing *what the network was asked to look at*.

---

## The task

Given grayscale fluorescence microscopy images of neuronal cells, produce a pixel-accurate mask for **every individual cell**, not a single foreground mask. Cells touch, overlap, and are thin and branched.

**Data**

- 424 labelled training images, 520 × 704 grayscale
- 182 test images
- 42,981 labelled cell instances in the training pool
- 1,972 additional **unlabelled** images provided for optional self-supervision
- Density varies enormously: from ~11 to ~609 cells in a single image

**Metric**

Per-image instance F1 at an IoU-style Dice threshold: predicted and ground-truth instances are matched greedily one-to-one, a match counts as a true positive only if Dice ≥ 0.50, and the final score is `2TP / (2TP + FP + FN)` averaged over images. This is a *detection* metric wearing a segmentation costume. A cell segmented at Dice 0.49 scores exactly the same as a cell that was never found at all, which shapes almost every decision below.

**Rules**

No external data, generic ImageNet-style pretrained weights only, teams up to five, top three must publish their solutions.

**Reference points published by the organisers**

| Reference | Public LB |
|---|---|
| Mask R-CNN + ResNet-50 starter baseline | 0.2623 |
| Previous year's 1st place solution | 0.5136 |

---

## Result

Final private leaderboard (top 4):

| # | Team | Private score |
|---|---|---|
| 1 | Abdallah Mohamud | 0.5598 |
| 2 | yo - Oxford | 0.5478 |
| **3** | **KAUST مكة (our team)** | **0.5472** |
| 4 | Abdulrahman Haj Mousa | 0.5443 |

Public-to-private movement was small across the board, which is consistent with the test split being stratified over cell density rather than drawn at random.

Two submissions were selected. The Stage 12 probability-map blend scored 0.5472 and set the team's final position; the Stage 15 single model scored 0.5463. They are effectively tied. See the postscript for what the rest of the private scores revealed.

---

## Approach

The final pipeline is a **semantic-to-instance** design rather than a proposal-based detector:

1. A U-Net with a ConvNeXt-Tiny encoder predicts three dense maps: **foreground**, an eroded **seed** map, and a **continuous normalised per-instance distance field**.
2. Training samples are **object-centric**: most crops are taken around a randomly chosen labelled cell and upscaled, so the network sees cells at a larger effective resolution than the native image provides.
3. At inference, predictions are averaged over **two scales × four flips** at native resolution.
4. Instances are recovered by **marker-controlled watershed**: thresholded seeds become markers, the foreground map becomes the landscape, and small components are dropped.

### Established techniques vs. our choices

Nothing in the list above was invented here. U-Net, ConvNeXt, distance-transform targets, marker-controlled watershed, EMA, flip/scale TTA and grouped cross-validation are all standard, and the watershed-from-seeds formulation for touching cells is the well-known recipe from nuclei and cell segmentation literature.

What was ours was the experimental design and the choices between them:

- deciding that a semantic + watershed pipeline was worth abandoning Mask R-CNN for, and measuring it rather than assuming it
- building a **near-duplicate-aware grouped split** after discovering the public/local gap was partly a leakage artefact
- the **object-centric zoom-in sampler** and the specific scale mixture, as a cheap approximation of "detect, crop, upscale" that fit the remaining compute budget
- switching the seed target from a **resampled binary core** to a **continuous normalised distance field**, after diagnosing a silent resampling bug
- the calibrated seed erosion level (0.70), found by a controlled sweep rather than guessed
- the decode-family probe and plateau-centre threshold selection, instead of taking the single best sweep cell
- the statistical gates that *rejected* two of our own ideas (see below)

---

## How the solution evolved

Fifteen stages, grouped into the five phases that actually mattered.

### Phase 1 — Detector baseline, and a validation split that was lying (Stages 1–4)

Started from the organisers' Mask R-CNN + ResNet-50 starter. A tuned Mask R-CNN with COCO initialisation reached **local 0.7909 / public 0.4967**, already well ahead of the baseline and of the previous year's winner.

That 0.79 vs 0.4967 gap was the first real problem. Random splitting was leaking: the dataset contains near-duplicate images, so validation images had close relatives in training. Rebuilding the split with a **near-duplicate detector** (pairwise image correlation, threshold 0.72, grouped 5-fold, 421 groups from 424 images) dropped local scores by several points and made them honest. A higher-resolution Mask R-CNN on the honest split scored local 0.7507 / public 0.4696.

**Lesson:** the offset between local and public settled at roughly 0.29 and stayed there for every subsequent stage. The absolute local number was never trustworthy, but the *ranking* of stages was, and that is what a validation set is actually for.

### Phase 2 — Switching families: ConvNeXt U-Net + watershed (Stage 5)

Replaced the detector with a ConvNeXt-Tiny U-Net predicting foreground + eroded seeds, decoded by marker-controlled watershed. Trained in 54 minutes on a T4 and scored **public 0.5293**, ahead of the previous year's winning solution and enough for 1st place at that point in the competition.

**Lesson:** for dense, touching, non-convex objects with a Dice ≥ 0.50 matching metric, a dense-prediction pipeline beat the box-proposal detector at a fraction of the training cost.

### Phase 3 — Self-supervision on the unlabelled set (Stages 6–7)

The competition provided 1,972 unlabelled images, so we tried to use them: SimSiam pretraining of the encoder, then Mean Teacher semi-supervised fine-tuning, then a three-member ensemble of SSL-adapted models.

- SimSiam + Mean Teacher: **0.5209** (Mean Teacher contributed nothing)
- SSL ensemble of 3: **0.5259**

Both were *below* the plain supervised Stage 5 model.

**Lesson:** self-supervision is not free. Under a few-hours budget, on ~2k unlabelled images of the same distribution, an ImageNet-22k encoder was already a better starting point than anything we could pretrain ourselves. We stopped spending time here, which was the useful outcome of the experiment.

### Phase 4 — Systematic tuning, ensembles, and one instructive bug (Stages 8–12)

This phase was mostly measurement:

| Stage | Idea | Public |
|---|---|---|
| 8 | Two non-SSL models with different seed erosion, map-averaged | 0.5294 |
| 9 | Cross-family blend (SSL × non-SSL), blend-weight sweep | 0.5286 |
| 10 | Controlled seed-erosion probe: 0.45 / 0.65 / 0.70 / 0.80 | **0.5295** |
| 11 | Density-targeted augmentation, dense-image oversampling | 0.5290 |
| 12 | Probability-map blend of the two best models (w = 0.35) | **0.5335** |

Two results were worth more than the scores:

**The erosion probe (Stage 10)** bracketed the seed erosion optimum at 0.70 (local F1 0.8086 at e70 vs 0.7933 at e45), and its per-density audit showed where the model was actually losing: sparse images scored 0.857 while very dense images scored 0.752, and on dense images the model predicted only ~0.87 cells for every ground-truth cell. The failure mode was *under-segmentation of crowded cells*, not bad masks.

**Stage 11 was a regression, and finding out why mattered.** Density-targeted augmentation *should* have helped. It did not, because the binary seed cores occupy ~1.5% of pixels, and resizing them with nearest-neighbour interpolation silently deleted the cores of small cells. The pipeline ran without error and just quietly got worse. That bug is the direct reason the final architecture regresses a **continuous distance field** and derives the binary seed *after* resizing: a normalised distance field is scale-invariant and interpolates cleanly.

### Phase 5 — Alternative backbones, then the final single model (Stages 13–15)

Stage 13 tried a DINOv3 ViT-S/16 backbone with a dual DINO + CNN detail branch (local 0.7956, not submitted because the pretrained-weights rules were unconfirmed). Stage 14 tried a density-gated mixture of experts. Neither beat the Stage 12 blend.

With one submission and about two hours of T4 time left, Stage 15 abandoned ensembling and went back to the diagnostic from Stage 10: the model was losing cells in dense images because individual cells occupied too few pixels. The reported leading approach detected cells, cropped around each one and upscaled the crop. The transferable principle there is not the detector, it is **object-centric effective resolution**, and that can be approximated during training instead of at inference:

> Sample a labelled instance uniformly from the pool of 42,981 cells, crop a jittered context window around it at native resolution, and upscale that window into the fixed network crop.

Because dense images contain more cells, uniform sampling over the *instance* pool automatically oversamples dense images without any hand-tuned density weighting. Stage 15 scored **local 0.8144 / public 0.5345** as a single model, beating every ensemble we had built on the public leaderboard. On the private leaderboard it scored 0.5463, marginally below the Stage 12 blend's 0.5472. See the postscript.

**Lesson:** the last real gain came from changing the sampling and target formulation, not from adding parameters or members.

---

## Final architecture

```
input 3 × H × W  (grayscale replicated to 3 channels)
   │
   ├── stem: 3×3 conv → BN → ReLU (full resolution, 32 ch)
   │
   └── encoder: convnext_tiny.fb_in22k_ft_in1k (timm, features_only)
                feature channels [96, 192, 384, 768]
                            │
        U-Net decoder: 256 → 128 → 64 → 32, bilinear upsampling,
        skip connections at every stage, final fusion with the stem
                            │
                    1×1 conv → 3 channels
                            │
     ┌──────────────────────┼──────────────────────┐
 foreground              seed (e70)        normalised distance
   (BCE + Dice)          (BCE + Dice)     (weighted smooth-L1 on sigmoid)
```

- **Encoder:** ConvNeXt-Tiny pretrained on ImageNet-22k and fine-tuned on 1k, via `timm`. A hard assertion prevents a silent fallback to a different encoder.
- **Targets:** foreground (binary), a per-instance Euclidean distance transform normalised to its own maximum, and a contact zone (dilated instance overlap) used only to weight the loss (×4 near contacts).
- **Seeds are never stored as a resized binary map.** They are obtained by thresholding the distance field at 0.70 *after* any resize.
- **Loss:** weighted BCE + Dice on foreground and seed, plus a foreground-weighted smooth-L1 regression on the distance field (weight 1.5; foreground pixels weighted 3.0 vs 0.10 for background, with an extra boost inside contact zones).

---

## Training strategy

| Setting | Value |
|---|---|
| Crop | 400 × 512 |
| Batch size | 8 |
| Schedule | 72 epochs × 42 steps |
| Optimiser | AdamW, lr 3e-4, encoder lr × 0.25, weight decay 1e-4 |
| Warmup / freeze | 150 warmup steps, encoder frozen for 2 epochs |
| Grad clip | 5.0 |
| Precision | Mixed (AMP) |
| Weight averaging | EMA, decay 0.9985, used for checkpoint selection |
| Hardware | Single T4 (Colab) |
| Training time | ~21 minutes |

**Object-centric zoom-in sampling**

- 70% of samples are centred on a labelled instance sampled uniformly from the 42,981-instance pool, with 18% centre jitter
- 30% are random context crops, so the network does not learn a "there is always a cell in the middle" prior
- Zoom mixture: 25% at 1.15–1.35×, 50% at 1.35–1.65×, 25% at 1.65–1.90×

**Augmentation:** full D4 (90° rotations plus flips, valid because these microscopy images have no canonical orientation), brightness/contrast jitter at 0.15σ, gamma 0.12, Gaussian noise 0.02, blur with p = 0.15.

The D4 rotation is chosen *before* the pre-rotation crop size is decided, because the crop is rectangular and a naive 90° rotation of a finished 400 × 512 crop produces a 512 × 400 tensor. This was a real bug caught by the shape sanity check.

---

## Inference and instance decoding

**Inference:** 2 scales (1.35×, 1.65×) × 4 flips, probability maps resized back and averaged at native resolution, with OOM-safe tiling (768 tiles, 128 overlap) for large views. Tiling parity against whole-image inference was verified at a mean absolute difference of 0.00005.

The notebook contains a preflight cell that trains three steps, runs one full-TTA image and projects the total wall clock. On the final run it projected over budget and **automatically reduced the TTA from three scales to two**, which is why the shipped configuration uses (1.35, 1.65).

**Decoding** is a two-step selection rather than a fixed recipe:

1. **Family probe** on 40 validation images — four combinations of marker source (seed head vs distance head) and watershed landscape (foreground vs distance):

   | markers | landscape | F1 |
   |---|---|---|
   | seed | fg | **0.8185** |
   | dist | fg | 0.8170 |
   | seed | dist | 0.8079 |
   | dist | dist | 0.8015 |

2. **Fine sweep** over all 86 validation images: `t_cell ∈ {0.40…0.60}`, `t_marker ∈ {0.50…0.75}`, `min_area ∈ {50, 70, 90, 110}`, then selection of the **plateau centre** rather than the peak cell, to avoid fitting the sweep noise. Peak 0.8145, plateau size 56, chosen `t_cell 0.50 / t_marker 0.65 / min_area 90` at F1 0.8144.

**A rejected idea, kept in the notebook:** density-conditioned `t_cell` (a different cell threshold per density bin) was measured for the third time here. Gain +0.0011 with a bootstrap win-rate of 0.81, against a gate of +0.004 and 0.90. It was **not applied**. Choosing per-bin thresholds on the same 86 images is selection bias, and a gate that occasionally rejects your own idea is the only kind worth having.

---

## Validation and results

**Split:** grouped 5-fold, fold 0, 421 near-duplicate groups, 338 train / 86 validation, maximum validation-to-train image correlation 0.665.

**Headline:** local F1 **0.8144**, against Stage 10's 0.8072 (public 0.5295) and Stage 12's 0.8119 (public 0.5335).

**By density (86 validation images):**

| Density bin | Images | F1 | mean GT | mean pred | pred / GT |
|---|---|---|---|---|---|
| sparse < 50 | 41 | 0.8567 | 27.4 | 27.7 | 1.010 |
| medium 50–199 | 28 | 0.7699 | 91.9 | 94.8 | 1.032 |
| dense 200–399 | 11 | 0.7917 | 320.6 | 293.4 | 0.915 |
| very dense ≥ 400 | 6 | 0.7749 | 476.8 | 409.2 | 0.858 |

Compared with Stage 10 (0.8573 / 0.7569 / 0.7792 / 0.7518), the object-centric model gains most where it was designed to: +0.013 on dense and +0.023 on very dense images.

**Near-miss census** (share of ground-truth cells by best achievable Dice, which shows how much score is sitting just under the 0.50 matching threshold):

| | hit ≥ 0.50 | near 0.35–0.50 | weak 0.10–0.35 | missed < 0.10 |
|---|---|---|---|---|
| Final model | 76.5% | 9.0% | 9.1% | 5.3% |
| Stage 12 | 77.0% | 8.6% | 8.3% | 6.1% |

9% of cells are found but land just below the matching threshold, and a further 5.3% are never found. That census, not the aggregate F1, is what pointed at effective resolution as the thing to fix.

**Submission integrity checks** run before writing the CSV: RLE encode/decode round-trip parity (delta 0.000000), all 182 test IDs present, zero fallback predictions, 109.0 predictions per image on average (min 15, max 592).

---

## Repository structure

```
.
├── Final_notebook/
│   ├── 15th_kaust_challenge2_FINAL_slamdunk_colab_FIXED_outputs.ipynb   # the submitted solution
│   └── 15th_REPORT.md                                                   # auto-generated run report
│
├── notebook_with_outputs/          # the full experimental history, stages 2-14
│   ├── kaust-kaggle-challenge-2-2026*.ipynb    # Phase 1: Mask R-CNN line (Kaggle)
│   ├── 5th ... 14th notebooks                  # Phases 2-5: ConvNeXt U-Net line (Colab)
│   ├── 12th_kaust-kaggle-challenge-2-*.ipynb   # the blend that set the team's final score
│   └── *_REPORT.md                             # one report per stage
│
├── pics/                           # leaderboard and competition screenshots
└── README.md
```

The published Kaggle notebook is the Stage 15 single model, because it is the cleanest and most reproducible artefact of the run. The Stage 12 blend, which is the submission that actually set the team's final private score, is in `notebook_with_outputs/` alongside its report, and is also [public on Kaggle](https://www.kaggle.com/code/ghostylicious/kaust-kaggle-challenge-2-2026-12th) for anyone reconciling this repository against the leaderboard. That notebook covers several stages in one file and is correspondingly messier.

Every stage notebook writes its own `REPORT.md` at the end of the run: configuration, validation table, decode sweep, density audit and submission integrity. Those reports, not the notebooks, are the fastest way to read the progression.

---

## How to run the final notebook

The final notebook is written for **Google Colab with a T4 GPU** and runs unattended from a cold start.

1. Open `Final_notebook/15th_kaust_challenge2_FINAL_slamdunk_colab_FIXED_outputs.ipynb` in Colab.
2. Runtime → Change runtime type → **T4 GPU**.
3. Upload your `kaggle.json` API token to `/content/kaggle.json`. The notebook downloads the competition data itself.
4. **Run all.**

Notes:

- `CFG.SMOKE_TEST = True` gives an ~8 minute rehearsal that proves the pipeline runs end to end. The numbers it produces are meaningless.
- `CFG.RESUME = True` reuses cached targets, checkpoints and probability maps if a cell is re-run.
- Mounting Drive is optional; if present, the checkpoint and probability maps are mirrored there.
- A full run produces `15th_submission.csv`, `15th_REPORT.md`, the best checkpoint and the cached probability maps.
- Total run time on one T4 is about 26 minutes, of which ~21 is training and ~3 is TTA inference over 268 images.

---

## Lessons learned

1. **Fix the validation set before tuning anything.** Near-duplicate leakage made an early model look 0.30 better than it was. The grouped split cost points on paper and paid for itself for the rest of the competition.
2. **A stable local-to-public offset is as useful as a calibrated one.** Ours sat near 0.29 for fifteen stages. Once you know the ranking transfers, you can stop worrying that the absolute number is wrong.
3. **The metric is the specification.** With matching at Dice ≥ 0.50, splitting one merged blob into two correct cells is worth far more than polishing a boundary. Every decision that helped came from that observation.
4. **Diagnose per-slice, not in aggregate.** The density audit and the near-miss census told us *which* cells were being lost. Aggregate F1 only told us that something was.
5. **A silent bug is more expensive than a failed idea.** Nearest-neighbour resampling of a 1.5%-coverage binary seed map deleted small cells' cores and produced a plausible-looking regression. The fix (regress a continuous distance field, threshold after resizing) became a core part of the final design.
6. **Gate your own ideas.** Density-conditioned thresholds were tested three times and rejected three times by a bootstrap gate. Selecting per-bin thresholds on 86 images is how you buy local score and lose leaderboard score.
7. **Self-supervision is not automatic value.** ~2k unlabelled images and a few GPU-hours could not beat an ImageNet-22k encoder. Knowing that after two stages was worth more than the stages cost.
8. **Know where your measurement stops resolving.** On 86 validation and ~91 private test images, differences under ~0.005 are noise. Most of the second half of this competition was spent inside that band, and the private leaderboard confirmed it. See the postscript.
9. **Compute discipline changes what you can attempt.** Fixed deadlines, preflight wall-clock projection, automatic TTA downgrade, resumable caching and OOM-safe tiling are the reason the final experiment could be attempted at all with two hours left.

---

## Postscript: what the private leaderboard actually showed

Private scores for every scored submission, alongside their public scores:

| Stage | Approach | Public | Private |
|---|---|---|---|
| 5 | First ConvNeXt-UNet + watershed, 54 min of training | 0.5293 | **0.5477** |
| 12 | Two-model probability-map blend (selected) | 0.5335 | 0.5472 |
| 15 | Object-centric single model (selected) | 0.5345 | 0.5463 |
| 8 | Two-model non-SSL ensemble | 0.5294 | 0.5455 |
| 9 | Cross-family blend | 0.5286 | 0.5448 |

The entire spread across five submissions is 0.0029, and the public and private orderings are close to reversed. **Stage 5, the first model of this family and the cheapest to train, had the best private score of anything produced in the following ten stages.**

The honest reading is that after Stage 5 the measurement resolution ran out. With 182 test images split roughly in half, and 86 local validation images, differences below about 0.005 are not distinguishable from noise, and essentially every decision from Stage 8 onward was made inside that band. The stages that produced real, repeatable movement were the ones with effects an order of magnitude larger:

- the leakage-aware validation rebuild, which corrected a ~0.30 illusion
- the switch from Mask R-CNN to dense prediction plus watershed, worth ~0.033 on the public leaderboard
- ruling out self-supervision on the unlabelled set, which saved time rather than earning score

Everything after that was tuning inside the noise floor. Selection did not rescue it and did not damage it either: had the unselected Stage 5 submission been chosen instead, it would have scored 0.5477 and still finished 3rd, 0.0001 behind 2nd place.

This is recorded here rather than quietly dropped, because knowing where your measurement stops resolving is more useful than the extra 0.005 of local F1 that motivated most of these stages. In a 30-hour window there was no time to build a noise estimate and then act on it, but there was time to build one, and that would have redirected the last third of the competition.

---

## Links

- Public Kaggle notebook, Stage 15 single model: https://www.kaggle.com/code/ghostylicious/3rd-place-object-centric-convnext-unet-distance
- Public Kaggle notebook, Stage 12 blend (the scored submission, multi-stage and less tidy): https://www.kaggle.com/code/ghostylicious/kaust-kaggle-challenge-2-2026-12th
- LinkedIn post: https://www.linkedin.com/posts/yahya-alsharif-204103304_computervision-deeplearning-imagesegmentation-ugcPost-7491964300434030592-Fpdu/ 

## Credits and attribution

Submitted as a two-person team entry during the KAUST Academy AI Summer Programme. This repository documents the solution line I developed, ran and evaluated across the fifteen stages, which produced both of the team's final selected submissions.

Built on established work: ConvNeXt (Liu et al.), U-Net (Ronneberger et al.), the `timm` library, marker-controlled watershed via `scipy` / `scikit-image`, and the distance-transform-plus-watershed formulation widely used in the cell and nuclei segmentation literature.
