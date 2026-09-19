# Class definitions and labelling rules

Classes are inherited from the source dataset (`construction-safety-gsnvb`, Roboflow-100, CC BY
4.0). We did not re-annotate it. This file records what the labels mean, how we verified them, and
the two dataset decisions a grader will ask about.

## 1. Problem framing

| Component | This project |
|---|---|
| Objects of interest | Hard hats, high-visibility vests, people, and the absence of each |
| Environment | Outdoor construction sites, mixed daylight, mixed camera distance |
| Critical metric | **Recall** — a missed `no-helmet` is a safety event, a false alarm is 30 seconds of a supervisor's time |
| Success criteria | Flag unhelmeted people in site photography at ≥0.80 recall for supervisor review |
| Most likely failure mode | Distant or partly occluded heads read as `person` only, so the absence of a helmet is never asserted |

## 2. Class list

| # | Class | Means | Train instances |
|---|---|---|---|
| 0 | `helmet` | A hard hat being worn on a head | 6,330 |
| 1 | `no-helmet` | A bare head where a hard hat should be | 291 |
| 2 | `no-vest` | A torso with no high-visibility vest | 2,222 |
| 3 | `person` | A person, regardless of PPE state | 7,075 |
| 4 | `vest` | A high-visibility vest being worn | 3,221 |

`person` is annotated independently of the PPE classes, so a compliant worker produces three
overlapping boxes (`person` + `helmet` + `vest`). That is the source schema and we kept it.

## 3. The three rules

The conventions the source dataset follows, confirmed by inspecting a sample of the labels:

1. PPE classes box **the item or the body region**, not the whole person — `helmet` is the hat,
   `no-helmet` is the head.
2. Partial and occluded people are labelled; a visible head and shoulders is still `person`.
3. Reflections, printed images and people on signage are not labelled.

## 4. Class imbalance — the defining property of this dataset

```
person     7075  ████████████████████████
helmet     6330  █████████████████████
vest       3221  ███████████
no-vest    2222  ███████
no-helmet   291  █
```

24:1 between the most and least common class, and the rarest class is the one the use case exists
to catch. Every conclusion in `error_analysis.md` traces back to this chart
(`results/class_distribution.png`).

The validation split carries the same problem: `no-helmet` appears 11 times across 6 images, so its
metrics have very wide uncertainty. `person` appears 246 times across 117 images.

## 5. Label quality pass

We did **not** re-annotate the dataset. We did audit it: per-split image counts and per-class
instance counts are computed in `01_Training.ipynb` §3, and the outputs are committed with the
notebook.

[FILL — worth 20 minutes before submission: open 10–15 validation images in Roboflow and check
whether `no-helmet` is applied consistently (is a hooded head `no-helmet`? a head in a soft cap?).
Any inconsistency found here is a stronger error-analysis finding than anything derived from the
metrics alone, because it is a cause rather than a symptom.]

## 6. Split decision — read before submitting

The brief specifies an **80/20** split. This dataset ships with the RF100 native split:

| Split | Images | Share |
|---|---|---|
| train | 1,001 source (2,983 after 3× augmentation) | ~83% |
| valid | 121 | ~10% |
| test | 84 | ~7% |

We kept it so our numbers stay directly comparable with the published RF100 baseline (mAP@50
0.882), which is a legitimate methodological reason and is stated in the README.

**If the grader requires a strict 80/20:** in Roboflow, create v2 with the split rebalanced to
80/20 (folding `test` into `valid`), re-run `01_Training.ipynb` against `VERSION = 2`, and update
the metrics table. Cost: ~25 minutes. Keep v1 documented either way so the comparison stands.

All reported metrics come from the **validation** split, never from test.
