# Results and evidence pack

Rule of thumb from the brief: opening the repository, a reader should see a successful detection
within two clicks. Nobody should have to run code to find out whether the model works.

## Curves (this folder)

| File | What it shows | Source |
|---|---|---|
| `results.png` | Loss curves down, mAP curves up across epochs | `runs/detect/<run>/results.png` |
| `confusion_matrix.png` | Where classes are confused, and what leaks to background | validation run |
| `PR_curve.png` | The precision/recall trade-off behind the threshold choice | validation run |

## Evidence (`evidence/`)

| Count | Naming | Content |
|---|---|---|
| 3–5 | `annotation_01.png` … | Screenshots of the annotation work in Roboflow — boxes on raw images, showing label rules applied |
| 10 | `val_pred_01.png` … `val_pred_10.png` | Model predictions on validation images |
| 5 | `new_pred_01.png` … `new_pred_05.png` | Predictions on images the model has never seen |
| 3 + 3 | `fp_01_*.png`, `fn_01_*.png`, … | The six failure cases discussed in `docs/error_analysis.md` |

Keep the failure screenshots captioned in `docs/error_analysis.md`, not in the filename — the
caption is where the marks are.

**Before committing:** check every screenshot for identifiable faces, name badges and site
identifiers. If any are present, blur them or replace the image (see
`docs/governance_checklist.md`, section 2).
