# Cell counting: image-information and workflow limits

Supporting material for *Image-information limits to automated cell counting across procedural and deep-learning workflows*.

The study compares two fixed procedural segmentation variants (scikit Bright and scikit Dim) with a retrained Cellpose CP model for brightfield haemocytometer images of suspension CHO cells. The primary count-agreement analysis comprises **72 quadrant images from 18 sampling timepoints** (864–1465 h) in one longitudinal experiment. The six sample images are a separate **descriptive audit**, not an independent validation dataset.

## Repository contents

| Path | Contents |
| --- | --- |
| [`Image Counts.csv`](Image%20Counts.csv) | Manual reference and three automated cell counts for all 72 images, with descriptive error columns and analysis-subset flags |
| [`Dataset S1 - Image allocation and dataset roles.csv`](Dataset%20S1%20-%20Image%20allocation%20and%20dataset%20roles.csv) | Image identities, development roles, and evaluation allocations |
| [`procedural-workflows.ipynb`](procedural-workflows.ipynb) | Demonstration of both fixed scikit-image variants on the six sample images |
| [`sample images/`](sample%20images/) | Six previously cropped brightfield images used in the descriptive segmentation audit |
| [`LICENSE`](LICENSE) | Licence for the authors' released source code; see restrictions below |

*If the uploaded count file has a different name, update the first link to match its exact path.*

## Complete 72-image count dataset

`Image Counts.csv` contains one record for each of the four quadrant images at each of 18 timepoints. The columns include image name/path, elapsed time, artefact and development-image flags, manual haemocytometer counts, and the scikit Bright, scikit Dim and retrained Cellpose CP counts. The file also includes CCRE and absolute-count-error values.

The current CSV uses a **two-row heading**: the workflow names are in the first row and the headings `Count`, `CCRE` and `Absolute Err` appear in the second. Readers should treat the second row as a header, **not as an observation**. There are 72 image records after these headings. For statistical calculations, use the count columns; the reported errors can then be recomputed from the counts.

For each image and workflow, the manuscript defines:

- Signed count error: `manual count - automated count` (positive means undercounting).
- Absolute count error: `abs(manual count - automated count)`.
- Cell count relative error (CCRE): `100 × absolute count error / manual count`.
- Bland–Altman bias: mean signed count error.
- Approximate 95% limits of agreement: `bias ± 1.96 × sample standard deviation of signed count error`.

The published tables report summary statistics for all 72 observations and for sensitivity subsets that exclude the 4 parameter-selection images, 10 model-screening images, or 6 artefact-classified images, as indicated in the accompanying metadata. The CSV enables **count-level** recalculation; it is not a substitute for the original images, segmentation masks or model weights.

**Time precision:** the `Time elapsed (hours)` labels in the count CSV are rounded at some half-hour sampling points. For growth-curve and kinetic calculations, use the exact elapsed times and dilution factors below rather than treating the rounded labels as exact timestamps.

## Cell concentration and growth-curve metadata

The original experimental records used the mean of four quadrant counts and the corresponding Trypan dilution factor:

`cell concentration (cells/mL) = (Q1 + Q2 + Q3 + Q4) / 4 × dilution factor × 10,000`

| Exact elapsed time (h) | Dilution factor | Exact elapsed time (h) | Dilution factor |
| ---: | ---: | ---: | ---: |
| 864 | 2 | 1180.5 | 4 |
| 941 | 2 | 1202.5 | 4 |
| 988.5 | 4 | 1229 | 2 |
| 1013.5 | 4 | 1277 | 2 |
| 1031.5 | 4 | 1298.5 | 4 |
| 1037 | 4 | 1324.5 | 4 |
| 1105.5 | 2 | 1351.5 | 4 |
| 1132.5 | 4 | 1370.5 | 8 |
| 1157.5 | 4 | 1465 | 4 |

The underlying experiment also recorded **post-re-inoculation starting concentrations** of 250,000 cells/mL at 1037 h, 327,750 cells/mL at 1202.5 h, and 272,500 cells/mL at 1370.5 h for plotting cycle transitions. These are distinct from concentrations calculated from the four quadrant images at those same times. Refer to the manuscript for the specific exponential-growth windows and regression definitions.

## Procedural scikit-image workflows

The notebook implements two closely related workflows that were used as fixed procedural baselines. Both perform grayscale conversion, percentile-based contrast stretching, Gaussian smoothing (`σ = 0.5 px`), Otsu thresholding, binary closing and erosion with a one-pixel disk, connected-component labelling, and area exclusion.

| Workflow | Contrast stretching | Minimum retained component area |
| --- | --- | ---: |
| scikit Bright | 99th–100th intensity percentiles | 10 pixels |
| scikit Dim | 98th–99th intensity percentiles | 80 pixels |

Both variants were applied to every image, rather than selecting a variant based on the image. These implementations use connected-component labelling and **do not have a dedicated splitting stage** for touching foreground objects, such as marker-controlled watershed. The notebook demonstrates the methods on the six cropped audit images; it does not rerun all 72 proprietary source images.

To use the demonstration notebook, from the repository root:

```bash
pip install numpy pandas scikit-image matplotlib jupyter
jupyter lab procedural-workflows.ipynb
```

Software versions in any saved notebook output document that notebook's execution environment and should not be presumed to identify every package used in the original 2023 investigation.

## Cellpose model development

The original model-development exercise took place in **2023** using **Cellpose 2.0** in its graphical user interface (GUI):

- Ten images were used to screen pretrained models; CP, cyto and cyto2 were shortlisted.
- Four images were used for inference-parameter investigation. The **initial CP settings** selected at this stage were a diameter of 10 pixels, flow threshold 0.7, and cell-probability threshold 0.
- **Five** separate images acquired at or before 840 h, containing approximately 200 cell regions, were used for sequential human-in-the-loop retraining of the **final CP model**. Training used 100 epochs per update, learning rate 0.1 and weight decay 0.0001.
- After each CP training update, the recommended cell diameter was used for the following training image. The final **model-recommended** diameter was **19.99 pixels** after the fifth image. This value should not be confused with the initial 10-pixel setting; the original records do not conclusively identify how diameter selection was applied to every inference image.
- Two **additional** images were used to investigate further training of cyto and cyto2, but **did not update the final CP model**. Those models were not selected for the 72-image evaluation.
- No image used to update the final CP weights appears among the 72 evaluation images.

The `Training` flag in Dataset S1 historically covers the broader seven-image model-development exercise. It should **not** be interpreted as seven images having trained the selected CP model. The corresponding CP-specific allocation is five images.

The Cellpose workflow used the GUI; there is no separate custom inference script. **The trained CP weights and corrected training masks are not publicly included** because these materials are subject to third-party restrictions. Without the proprietary source images and model artefacts, the Cellpose segmentation itself cannot be re-executed exactly; count-level results can be examined using the released numerical data.

## Sample images and data restrictions

The six public microscopy images are the cropped fields used for the descriptive audit, **not the uncropped acquisition frames**. Crop coordinates relative to the originals are not provided in this repository.

The full microscopy dataset and restricted Cellpose artefacts were supplied or derived in collaboration with Newform Foods. The repository does not grant permission to redistribute any restricted material. The MIT licence applies to the authors' released **code** only; it does not imply an MIT licence for third-party images, cell-culture data or model artefacts. Contact the respective rights holder regarding access to restricted material.

## Scope of reproducibility

The released image-level counts and the equations in the manuscript support independent recalculation of count-level error and agreement statistics. The notebook permits inspection of the published procedural demonstration on six sample images. Reproducing all segmentation masks would additionally require the complete original microscopy dataset, the Cellpose model weights and the recorded inference configuration, which are not publicly deposited.
