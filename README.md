# Patient-level and cross-site re-evaluation of breast histopathology classifiers

Code and run logs for the paper:

> Hossain MR. *Patient-level and cross-site re-evaluation of foundation-model classifiers for breast histopathology: data leakage, label conflicts and input resolution under domain shift.* Submitted to *Computers in Biology and Medicine*, 2026.

The study re-evaluates frozen and LoRA-adapted foundation-model classifiers (DINOv2 ViT-B/14, Phikon-v2 ViT-L/16) on **BreakHis** under patient-disjoint cross-validation, and tests every model without tuning on **BACH** (ICIAR 2018) under a pre-specified 4 × 2 design (input pipeline × post-hoc correction).

## Repository contents

| Folder | What it contains |
|---|---|
| `notebooks/01_breakhis_benchmark.ipynb` | Integrity audit, patient-disjoint 5-fold CV, all methods, BACH baseline, statistics, every BreakHis table and figure |
| `notebooks/02_bach_interventions.ipynb` | Pre-specified BACH experiment: resize / native-resolution tiles / stain normalisation / tiles + stain norm., with and without EM prior correction |
| `runs/` | The executed notebooks exactly as run on Kaggle (all printed outputs and tables) |
| `figures/` | All figures used in the paper; `Fig1_study_design.drawio` is the editable study-design diagram |

## Data

Neither dataset is redistributed here. Download them from the original sources:

- **BreakHis**: <https://web.inf.ufpr.br/vri/databases/breast-cancer-histopathological-database-breakhis/> (Kaggle mirror used in the notebooks: `ambarish/breakhis`)
- **BACH (ICIAR 2018, Part A, microscopy images)**: <https://iciar2018-challenge.grand-challenge.org/> (Kaggle copy used in the notebooks: `erenyeager00111/iciar2018-bach-images`)

### Cleaned BreakHis split

The notebook removes exact duplicates found by hashing the decoded RGB pixels:

| Type | Patient | Labels | Groups | Images | Action |
|---|---|---|---:|---:|---|
| Cross-label | 14-13412 | DC and LC | 123 | 246 | all removed |
| Within-patient | 14-12204 | LC | 2 | 4 | one copy kept |

This leaves **7,661 images from 80 patients**. Fold assignments are produced deterministically by `StratifiedGroupKFold(n_splits=5, shuffle=True, random_state=42)` grouped by patient and stratified by subtype; the notebook asserts that no patient appears in more than one fold.

## How to reproduce

The notebooks were run on Kaggle with one NVIDIA T4 GPU.

1. Create a Kaggle notebook from `notebooks/01_breakhis_benchmark.ipynb` and attach both datasets with **Add Input**.
2. Set `RUN_PART = "binary"` and run (about 7 h), then `RUN_PART = "subtype"` (about 7 h).
3. Attach the outputs of both runs, set `RUN_PART = "report"` and run to produce all tables, figures and the claims check.
4. Run `notebooks/02_bach_interventions.ipynb` with both datasets attached for the BACH experiment.

Set `QUICK_TEST = True` for a short smoke test before a full run.

## Main results (from the paper)

- Image-level splitting inflates 8-class balanced accuracy from 44.3% to 95.0% for identical models.
- Under patient-level CV, all Phikon-based methods reach 88.7–89.3% binary balanced accuracy; LoRA adaptation is never significantly worse, but not consistently better, than frozen heads.
- On BACH, the frozen Phikon-v2 probe collapses (50.0%); LoRA adaptation recovers +11.8 pp over it in all seeds.
- Native-resolution tiling raises LoRA-Phikon on BACH from 61.4% to 72.8% (all seeds); stain normalisation and EM prior correction do not help.

## Requirements

See `requirements.txt`. The notebooks install nothing beyond the standard Kaggle GPU image.

## License

Code: MIT (see `LICENSE`). The datasets keep their original licences and terms of use.

## Citation

See `CITATION.cff`. Please cite the paper once published.
