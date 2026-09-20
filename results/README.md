# Results and evidence pack

Rule of thumb from the brief: opening the repository, a reader should see a successful detection
within two clicks. Nobody should have to run code to find out whether the model works.

## Curves (this folder)

| File | What it shows | Source |
|---|---|---|
| `BoxPR_curve.png` | The precision/recall trade-off behind the threshold choice | validation run |
| `confusion_matrix.png` | Where classes are confused, and what leaks to background | validation run |
| `BoxF1_curve.png` | F1 against confidence, for picking an operating point | validation run |

## Evidence (`evidence/`)

| Count | Naming | Content |
|---|---|---|
| 3–5 | `annotation_01.png` … | Screenshots of the annotation work in Roboflow — boxes on raw images, showing label rules applied |
| 10 | `val_01.jpg` … `val_10.jpg` | Model predictions on validation images |
| 5 | `new_01.jpg` … `new_05.jpg` | Predictions on images the model has never seen |
| 3 + 3 | `fp_01_*.png`, `fn_01_*.png`, … | The six failure cases discussed in `docs/error_analysis.md` |

Note: none of the 15 prediction images contains an unhelmeted worker, so `no-helmet` is not
exercised there. That class is illustrated in the three FN cases.

Keep the failure screenshots captioned in `docs/error_analysis.md`, not in the filename — the
caption is where the marks are.

**On faces:** the evidence images contain identifiable faces, unblurred. See
`docs/governance_checklist.md` §2 for why, and what would change outside coursework.
