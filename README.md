# PPE detection on construction imagery — YOLOv8

Course: MAICEN — Module 4, Unit 3 (Computer Vision) · FMP group assignment
:Team: Americo Nuno Caldas Teixeira, Jose Brito de Barros Aguiar, Manuel González Oliva, Natalia Lungu, Cesare Della Corte

> **Safety label / limitations declaration**
> This model is an assistive tool for preliminary screening only. It produces false negatives.
> It must **NOT** be used as the sole verifier for life-safety decisions. Every detection is
> reviewed by a site supervisor before any action is taken. Full limitations statement in
> [`docs/governance_checklist.md`](docs/governance_checklist.md).

---

## 1. Problem and success criteria

**AECO problem.** PPE compliance on site is checked by eye, intermittently, by supervisors who are
also doing several other things. A detector that pre-screens site photographs or CCTV snapshots for
people without a helmet or without a high-visibility vest turns an ad-hoc visual sweep into a
ranked worklist: the supervisor reviews the frames the model flags instead of scrolling through
everything.

**Deployment mode.** Near real-time — a fixed camera or a periodic photo drop, processed on a
schedule. Not edge, not instant: 9 ms inference on a T4 is far faster than the human review loop
around it.

**Error priority — recall-first ("paranoid").** A missed `no-helmet` is a potential injury; a false
alarm costs a supervisor thirty seconds. We accept false positives in order to reduce misses. This
is the most important decision in the project, and the current model does not meet it — see
section 5.

| Criterion | Target | Achieved | Verdict |
|---|---|---|---|
| mAP@50, all classes | ≥ 0.70 | **0.867** | Pass |
| Recall, `no-helmet` | ≥ 0.80 | **0.545** | **Fail** |
| Recall, all classes | ≥ 0.80 | 0.787 | Marginal |
| Precision, all classes | ≥ 0.70 | 0.863 | Pass |
| Inference speed | < 100 ms/image | 9.1 ms (T4) | Pass |

---

## 2. Classes and label rules

Five classes, inherited from the source dataset. Definitions and edge cases:
[`docs/class_definitions.md`](docs/class_definitions.md)

| # | Class | Train instances | Meaning |
|---|---|---|---|
| 0 | `helmet` | 6,330 | A hard hat, worn |
| 1 | `no-helmet` | **291** | A head with no hard hat — the safety-critical class |
| 2 | `no-vest` | 2,222 | A torso with no high-visibility vest |
| 3 | `person` | 7,075 | A person, regardless of PPE |
| 4 | `vest` | 3,221 | A high-visibility vest, worn |

The distribution is severely imbalanced: 24 `person` instances for every 1 `no-helmet`. This is the
root cause of the recall failure above and drives the whole iteration plan.

---

## 3. Dataset

- **Source:** [construction-safety-gsnvb-oz6um v1](https://universe.roboflow.com/caprijopi-hotmail-com/construction-safety-gsnvb-oz6um/dataset/1),
  forked from the Roboflow-100 `construction-safety-gsnvb` benchmark into workspace
  `caprijopi-hotmail-com`
- **Licence:** CC BY 4.0 · **Export format:** YOLOv8
- **Split:** 2,983 train / 121 valid / 84 test. Train is 1,001 source images augmented 3×
  (horizontal flip, brightness ±15%); 1,206 source images in total.
- **Note on the split.** This is the dataset's native RF100 split (≈83/10/7), kept so results stay
  comparable with the published baseline. The brief asks for 80/20 — see
  [`docs/class_definitions.md`](docs/class_definitions.md) §6 for the decision and the fallback.
- **Published RF100 baseline:** mAP@50 0.882, precision 0.928, recall 0.774.

---

## 4. Quick start (Colab, no local install)

1. Open [`notebooks/02_Inference.ipynb`](notebooks/02_Inference.ipynb) in Colab:
 `https://colab.research.google.com/github/caprijopi-alt/ppe-detection-yolov8/blob/main/notebooks/02_Inference.ipynb`
2. Runtime → Change runtime type → **T4 GPU** (CPU also works, just slower).
3. Runtime → **Run all**. It downloads the released weights and runs inference on validation and
   new images.
4. Expected runtime: ~2–3 min.

Nothing is read from a local drive.

### Full reproduction (training)

1. Open [`notebooks/01_Training.ipynb`](notebooks/01_Training.ipynb) in Colab (T4 GPU).
2. Runtime → **Run all**. It prompts for a Roboflow API key (never hardcoded), downloads dataset
   v1, audits it, trains, validates, and writes curves and predictions.
3. Expected runtime: **~18 min** of training (24 epochs on a T4) plus ~5 min setup and download.

---

## 5. Results

`yolov8n` fine-tuned from COCO weights. Validation split: 121 images, 730 instances.

| Class | Precision | Recall | mAP@50 | mAP@50–95 | Val instances |
|---|---|---|---|---|---|
| **all** | **0.863** | **0.787** | **0.867** | **0.474** | 730 |
| person | 0.864 | 0.919 | 0.945 | 0.611 | 246 |
| helmet | 0.921 | 0.888 | 0.921 | 0.522 | 237 |
| vest | 0.818 | 0.837 | 0.884 | 0.495 | 145 |
| no-vest | 0.764 | 0.746 | 0.797 | 0.388 | 91 |
| no-helmet | 0.947 | **0.545** | 0.789 | 0.353 | 11 |

**Key takeaways**

1. **mAP@50 of 0.867 is solid.** Above the 0.7 "useful as a site-monitoring assistant" band and
   within 1.5 points of the published RF100 baseline (0.882) — reached with a nano model in 18
   minutes. As a screening aid, the model works.
2. **The safety-critical class is the weakest, which inverts that verdict.** `no-helmet` recall is
   0.545: roughly 5 of the 11 unhelmeted people in the validation set were missed. For a
   recall-first safety case this is the number that matters, and it fails. Precision on the same
   class is 0.947 — when it fires it is almost always right, but it fires too rarely. The model
   behaves like a "skeptic" where the use case demands a "paranoid".
3. **mAP@50–95 of 0.474 against mAP@50 of 0.867** means objects are found but boxes fit loosely.
   Acceptable for "someone in this frame has no helmet"; not acceptable for anything that measures
   or counts precisely.
4. **The `no-helmet` figures rest on 11 instances across 6 images.** The uncertainty around a
   recall of 0.545 at n=11 is very wide. Read it as a signal that the class is undertrained and
   under-validated, not as a precise measurement — fixing that support is improvement #1.

**Evidence:** [`results/`](results/) — `results.png`, `confusion_matrix.png`, `BoxPR_curve.png`,
`class_distribution.png`, plus [`results/evidence/`](results/evidence/).

**Error analysis:** [`docs/error_analysis.md`](docs/error_analysis.md)

**Weights:** [`best.pt`](https://github.com/caprijopi-alt/ppe-detection-yolov8/releases/download/v1.0/best.pt) (5.96 MB, saved from epoch 9) — GitHub Release `v1.0`

---

## 6. Reproducibility checklist

- [x] **Dataset:** [v1 link above](https://universe.roboflow.com/caprijopi-hotmail-com/construction-safety-gsnvb-oz6um/dataset/1), CC BY 4.0, YOLOv8 format
- [x] **Model variant:** `yolov8n.pt` (COCO-pretrained)
- [x] **Hyperparameters:** `epochs=50` (stopped at 24), `batch=16`, `imgsz=640`, `seed=0`,
      `patience=15`
- [x] **Ultralytics:** 8.4.155 · **torch:** 2.11.0+cu128 · **torchvision:** 0.26.0+cu128 ·
      **roboflow:** 1.5.0 · **numpy:** 2.1.3 · **Python:** 3.13.15
- [x] **Weights:** [`best.pt`](https://github.com/caprijopi-alt/ppe-detection-yolov8/releases/download/v1.0/best.pt) from epoch 9, published as a GitHub Release asset (`v1.0`)
- [x] **API key:** prompted via `getpass`, never committed
- [x] **Environment snapshot:** full `pip freeze` from the training runtime in
      [`docs/pip_freeze.txt`](docs/pip_freeze.txt) (Colab, 2026-09-19)
- [x] **Randomness:** `seed=0`; GPU non-determinism still moves metrics by roughly ±0.01 mAP
      between identical runs

## 7. Reproducibility proof

- **Last successful end-to-end run:** 2026-09-19, 11:15–11:57 UTC
- **Hardware:** Google Colab, NVIDIA Tesla T4 (15 GB), driver 580.82.07, CUDA 13.0
- **Runtime:** 24 epochs in 0.292 h (~17.5 min), plus ~5 min dataset download and setup
- **Outputs produced:** per-class metrics, `results.png`, confusion matrix, PR curve, class
  distribution chart, 6 annotated validation predictions, `best.pt`
- **Early stopping:** 50 epochs were configured with `patience=15`; training stopped at epoch 24
  after 15 without improvement, and best weights are from **epoch 9**. Early stopping is a
  deliberate anti-overfitting measure rather than a shortened run — the loss curves in
  `results/results.png` show validation loss flat from epoch 9 onward, so the remaining 26 epochs
  would have added compute without improving the model.

---

## 8. Repository structure

```
.
├── README.md
├── LICENSE
├── notebooks/
│   ├── 01_Training.ipynb        dataset → audit → train → validate → predict (outputs kept)
│   ├── 02_Inference.ipynb       load released weights → predict on val + new images
│   └── 03_Failure_Cases.ipynb   auto-find FP/FN against ground truth, annotate, export
├── docs/
│   ├── class_definitions.md     classes, label rules, imbalance, split decision
│   ├── sam_exploration.md       SAM 3 as a labelling aid — what helped, what failed
│   ├── error_analysis.md        3 FP + 3 FN with hypotheses, prioritised data fixes
│   ├── governance_checklist.md  provenance, PII, risk, human-in-the-loop, licence
│   └── pip_freeze.txt           environment snapshot
└── results/
    ├── results.png              loss and mAP curves
    ├── confusion_matrix.png
    ├── BoxPR_curve.png
    ├── class_distribution.png
    └── evidence/                annotations, validation preds, new-image preds
```

---

## 9. Licence and data rights

- **Code:** MIT — see [`LICENSE`](LICENSE)
- **Dataset:** `construction-safety-gsnvb`, Roboflow-100 benchmark, **CC BY 4.0**. Attribution
  retained; forked into `caprijopi-hotmail-com` and used unmodified at v1. No images of our own and
  no client site photography are included.
- **Weights:** fine-tuned from Ultralytics YOLOv8, licensed **AGPL-3.0**. Distribution or networked
  deployment of `best.pt`, or of code importing `ultralytics`, carries AGPL-3.0 obligations unless
  an Ultralytics Enterprise Licence is obtained.
- **Evidence screenshots:** derived from the CC BY 4.0 dataset. Faces are visible in some source
  images — see the PII section of [`docs/governance_checklist.md`](docs/governance_checklist.md).
