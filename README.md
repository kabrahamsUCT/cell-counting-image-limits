# cell-counting-image-limits

Code, data and analysis supporting the study *Image-information limits to
automated cell counting across procedural and deep-learning workflows*.

The study compares three automated counting workflows against manual
haemocytometer counts over a ~1500 h suspension CHO culture: two procedural
(classical image-processing) variants built on scikit-image, and a deep-learning
workflow built on Cellpose.

## Contents

| Path | Description |
|---|---|
| [`procedural-workflows.ipynb`](procedural-workflows.ipynb) | The two scikit-image workflows, applied to the sample images |
| [`Dataset S1 - Image allocation and dataset roles.csv`](Dataset%20S1%20-%20Image%20allocation%20and%20dataset%20roles.csv) | Per-image allocation of dataset roles (Supplementary Data S1) |
| [`sample images/`](sample%20images/) | Six brightfield frames spanning 864–1352 h |

## Procedural workflows

`procedural-workflows.ipynb` **covers the scikit-image workflows only** — scikit
Bright and scikit Dim. Both share one pipeline and differ only in the contrast
limits and whether a minimum-area exclusion is applied:

| Workflow | Contrast limits | Smoothing | Threshold | Cleaning | Area exclusion |
|---|---|---|---|---|---|
| scikit Bright | 99th / 100th percentile | Gaussian, σ = 0.5 px | Otsu | closing + erosion, 1 px disk | components < 10 px removed |
| scikit Dim | 98th / 99th percentile | Gaussian, σ = 0.5 px | Otsu | closing + erosion, 1 px disk | components < 80 px removed |

Neither is selected per image — both are applied to every image with fixed
settings and reported as two separate workflows. The count is the number of
connected components surviving the area rule.

The notebook reproduces the study's published per-image counts for the six
sample images: scikit Bright matches all six exactly, and scikit Dim matches
five of six (the 864 h image is discussed in the notebook).

To run it:

```
pip install numpy pandas scikit-image matplotlib jupyter
jupyter lab procedural-workflows.ipynb
```

Run from the repository root, so that the `sample images/` folder is found. The
notebook writes no files.

## Cellpose workflow

**The Cellpose workflow was run through the Cellpose GUI, not through a script,
so no code for it is included here.** See
[mouseland/cellpose](https://github.com/mouseland/cellpose) for the software,
installation instructions and GUI documentation.

Reproducing it means repeating that interactive procedure. As reported in the
paper's Methods:

- Pretrained models from the Cellpose model zoo were screened, shortlisting CP,
  cyto and cyto2.
- Inference settings: estimated cell diameter 10 px, flow threshold 0.7,
  cell-probability threshold 0.
- The shortlisted models were retrained by a human-in-the-loop procedure in the
  GUI — predicting masks, manually removing incorrect masks and adding masks
  around missed cells, then updating the model. Training used 100 epochs per
  iteration, a learning rate of 0.1 and a weight decay of 0.0001.
- The retrained CP model was selected for the final analysis. Its weights and
  inference parameters were fixed before application to the longitudinal
  dataset, and no image from the 72-image comparative dataset was used to update
  them.

The images used at each of these stages are identified in Dataset S1.

## Dataset S1 — image allocation and dataset roles

`Dataset S1 - Image allocation and dataset roles.csv` records, for every image,
its acquisition timepoint and its role at each stage of workflow development and
evaluation, as described in the supplementary materials. It documents which
images contributed to more than one stage, and permits reconstruction of the
72-image longitudinal dataset and of the sensitivity-analysis subsets.

| Column | Description |
|---|---|
| `folder_bin` | Timepoint folder label |
| `Time elapsed (hours)` | Elapsed culture time at acquisition |
| `filename`, `relative_path`, `extension` | Image identity and location within the image tree |
| `Manual Counts` | A manual haemocytometer reference count is available |
| `Artefact (bubble, glare, shadow)` | Image contains a bright artefact; basis of the artefact-excluded sensitivity subset |
| `Growth Curve Analysis` | Image is part of the 72-image longitudinal comparison |
| `Parameters` | Used for Cellpose inference-parameter selection |
| `Shortlist Creation` | Used for pretrained-model screening |
| `Training` | Used for human-in-the-loop retraining of the Cellpose model |
| `Validation and Detailed audit` | Used in the six-image descriptive segmentation audit |

Each role column is `Yes` / `No`. The `Training` and `Growth Curve Analysis`
columns are disjoint, confirming that no image used to update the model weights
entered the longitudinal evaluation.

## Sample images

Six brightfield frames from a single suspension CHO culture (line S1), spanning
864–1352 h of elapsed culture time — the six images of the descriptive
segmentation audit (`Validation and Detailed audit` in Dataset S1).

These are the cropped fields as analysed in the study, not raw frames, so the
notebook's counts are directly comparable with the study's published per-image
counts.

## Scope

This repository is a reference implementation and a data record. It reproduces
the published counts for the six sample images, but does not recompute the
paper's summary statistics. Parameter selection, sensitivity analysis, and
agreement against manual counts are reported in the paper and its supplementary
materials.

## License

MIT — see [LICENSE](LICENSE).
