# Error analysis

Method: `best.pt` (epoch 9) evaluated on the 121-image validation split at IoU 0.5, and inspected
visually at confidence 0.35 on a random sample of validation images (`01_Training.ipynb` §6).
Per-class precision and recall come from §5 of the same notebook. Screenshots for each case below
go in [`../results/evidence/`](../results/evidence/).

Vocabulary: **FP** = detected something that is not there · **FN** = missed something that is
there · **class confusion** = right object, wrong label · **localisation error** = right object,
sloppy box.

## Headline finding

The model is accurate overall (mAP@50 0.867) and unreliable exactly where it matters. Recall by
class, ordered:

| Class | Recall | Precision | Val instances |
|---|---|---|---|
| person | 0.919 | 0.864 | 246 |
| helmet | 0.888 | 0.921 | 237 |
| vest | 0.837 | 0.818 | 145 |
| no-vest | 0.746 | 0.764 | 91 |
| **no-helmet** | **0.545** | 0.947 | **11** |

Recall tracks training frequency almost perfectly (6,330 helmet instances → 0.888; 291 no-helmet →
0.545). This is not a modelling failure, it is a dataset failure, and it is fixable with data
rather than with hyperparameters.

---

## False positives

### FP-1 — `no-vest` on a clothed torso that is not a worker
- **Evidence:** `results/evidence/fp_01.png` — [FILL: filename from your prediction run]
- **Observed:** `no-vest` has the lowest precision of any class (0.764), meaning roughly a quarter
  of its detections are wrong.
- **Hypothesis:** `no-vest` is defined by an **absence**, so the model has to learn a negative.
  Any torso-shaped region without high-saturation yellow or orange satisfies it. Bystanders,
  people in the background and dark clothing all qualify visually, and at 2,222 instances there is
  not enough variety for the model to learn the surrounding context that distinguishes "worker
  without a vest" from "person who is not a worker".
- **Cost:** a supervisor opens a frame showing a visitor in a coat. Cheap individually, corrosive
  to trust if frequent.

### FP-2 — `helmet` on helmet-coloured objects
- **Evidence:** `results/evidence/fp_02.png` — [FILL]
- **Observed:** `helmet` precision 0.921 — about 1 in 13 helmet detections is wrong.
- **Hypothesis:** the classic look-alike failure from Session 2. A hard hat at distance is a
  smooth, saturated, roughly hemispherical blob. Buckets, traffic cones, drums and machinery
  housings share that signature, and the dataset contains no labelled negatives for them.
- **Cost:** low. A spurious helmet does not trigger an alert in a recall-first configuration.

### FP-3 — Duplicate and overlapping boxes on the same person
- **Evidence:** `results/evidence/fp_03.png` — [FILL]
- **Observed:** mAP@50–95 (0.474) is far below mAP@50 (0.867), which is the signature of loose,
  drifting boxes rather than of missed objects.
- **Hypothesis:** the schema places three overlapping labels on one compliant worker (`person`,
  `helmet`, `vest`). In crowded frames NMS has to separate boxes that genuinely overlap, and the
  looser the fit the more duplicates survive.
- **Cost:** inflated counts. Any use of this model for headcount would be wrong.

---

## False negatives

### FN-1 — Unhelmeted worker not flagged (the critical one)
- **Evidence:** `results/evidence/fn_01.png` — [FILL]
- **Observed:** `no-helmet` recall 0.545 — about **5 of the 11** unhelmeted people in the
  validation set were missed entirely.
- **Hypothesis:** **class imbalance.** With 291 training instances against 6,330 for `helmet`, the
  loss is dominated by the helmet class; the cheapest thing the model can do with an ambiguous head
  is stay silent or call it `helmet`. Precision of 0.947 on the same class confirms the shape of
  the problem: the model has learned what a bare head looks like, it has just learned to require
  very strong evidence before saying so.
- **Cost:** **this is the failure the system exists to prevent.** A worker without a hard hat
  passes unflagged in roughly half of cases. Stated plainly in the governance checklist.

### FN-2 — Distant and small instances missed
- **Evidence:** `results/evidence/fn_02.png` — [FILL]
- **Observed:** [FILL: confirm on your own prediction images — look for people in the background
  detected as `person` but with no PPE class attached].
- **Hypothesis:** *scale*. At `imgsz=640`, a head 20 m from the camera is a handful of pixels. The
  `person` box survives because the body is large; the helmet region does not carry enough signal,
  so the PPE classes go undetected while `person` succeeds. This matches `person` recall (0.919)
  being far above every PPE class.
- **Cost:** compliance cannot be assessed beyond a certain distance — a hard operating limit that
  belongs in the limitations statement.

### FN-3 — Occluded workers
- **Evidence:** `results/evidence/fn_03.png` — [FILL]
- **Observed:** [FILL: look for workers behind formwork, machinery or other people].
- **Hypothesis:** *occlusion*. When only a head and shoulders are visible, the PPE classes lose the
  context that normally supports them. Rule 2 in `class_definitions.md` says partial people are
  labelled, but partial instances are a small fraction of the training data.
- **Cost:** sites are cluttered; this is the normal viewing condition, not the exception.

---

## Prioritised next data improvements

Each is a dataset action. The evidence above says capacity is not the binding constraint — the
model stopped improving at epoch 9 and early-stopped at 24 — so more epochs or a larger backbone
would not fix any of this.

1. **Rebalance `no-helmet`: add 300–500 instances, roughly doubling to tripling the class.**
   Targets FN-1, the only failure that matters for the stated use case. Source from other Roboflow
   Universe PPE datasets with a compatible schema, or by mining frames where our model predicts
   `person` with no `helmet` and no `no-helmet` — the ambiguous cases are exactly the ones worth
   labelling. Expected effect: `no-helmet` recall from 0.55 toward 0.80. Effort: ~3 h.
   *Cheap partial fix available immediately: lower the confidence threshold for `no-helmet` only.
   At precision 0.947 there is a lot of headroom to trade precision for recall without drowning the
   supervisor. Measure it on the PR curve before committing.*

2. **Expand the validation set for the rare classes.** 11 `no-helmet` instances in 6 images cannot
   support a claim about recall. Move images from the 84-image test split, or add held-out images,
   until `no-helmet` has 50+ validation instances. This produces no accuracy gain — it makes the
   measurement trustworthy, which is a prerequisite for showing improvement #1 actually worked.
   Effort: ~1 h.

3. **Add hard negatives for `no-vest` and `helmet`.** Targets FP-1 and FP-2: frames containing
   buckets, cones, drums and non-worker bystanders, labelled as background. Expected effect:
   `no-vest` precision up from 0.764; fewer helmet look-alikes. Effort: ~2 h.

Explicitly **not** on the list: more epochs, `yolov8s`, heavier augmentation. Validation mAP
plateaued at epoch 9 of 24 — the ceiling here is data coverage, not training.

## Iteration plan

| Version | Change | Metric to watch | Status |
|---|---|---|---|
| v1 | Baseline, `yolov8n`, 50 epochs configured / 24 run | mAP@50 = 0.867 | Done |
| v2 | Improvement 2 (validation support), then 1 (rebalance) | `no-helmet` recall | Planned |
| v3 | Improvement 3 (hard negatives) | `no-vest` precision | Planned |
