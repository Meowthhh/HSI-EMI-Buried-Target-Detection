# Hyperspectral vs. Electromagnetic Induction for Buried-Ordnance Detection

A depth- and material-resolved comparison of **VNIR hyperspectral imaging (HSI)** and **EM61-Lite electromagnetic induction (EMI)** on the **RIT/DIRS DRC Site 2 seeded field**.

The project asks a simple question:

> **As targets are buried more deeply, how much does each sensing modality still recover?**

The analysis combines hyperspectral target/anomaly detection, EM61 response analysis, burial-depth statistics, material comparisons, sensor-agreement analysis, positive controls, georeferencing checks, and an empty-control-hole experiment.

---

## Highlights

- Cleans the original **150-row** target table into:
  - **131 real seeded targets**
  - **8 empty control holes**
  - **11 blank/placeholder rows**
- Uses a **3123 × 6631 × 272** VNIR hyperspectral cube.
- Restricts HSI processing to wavelengths up to **900 nm** (**226 bands** in the notebook run).
- Evaluates HSI using **SAM**, **Matched Filter (MF)**, **ACE**, and **RX/local anomaly analysis**.
- Uses **EM61-Lite leveled Channel 1** measurements for EMI analysis.
- Matches targets to the strongest EMI reading within **1 metre**.
- Compares both sensors using:
  - surface / shallow / deep burial classes,
  - exact continuous burial depth,
  - surface vs. buried targets,
  - target material,
  - conductive/inert responsiveness,
  - empty control holes.
- Supports optional NVIDIA GPU acceleration through **CuPy**, with automatic NumPy CPU fallback.

---

## Main Result

The final report shows that the two sensors behave very differently.

| Metric | HSI (VNIR) | EMI (EM61) |
|---|---:|---:|
| Overall detection | **0.05 (6/130)** | **0.46 (60/130)** |
| Surface detection | 0.10 | 0.26 |
| Buried detection | 0.03 | 0.53 |
| Depth correlation | r = -0.06, p = 0.47 | r = 0.19, p = 0.03 |
| Depth relationship | Not significant | Significant positive trend |

When targets are grouped into three burial classes, HSI appears to fall with depth:

```text
Surface → Shallow → Deep
 0.10   →  0.04   → 0.00
```

However, this trend disappears when exact measured depth is used. The continuous HSI relationship is essentially flat and statistically non-significant.

EMI behaves differently. Its detection rate rises from the surface to buried targets, and the continuous-depth relationship remains statistically significant.

---

## Why This Project Is Interesting

A simple three-bin analysis initially suggests a clean "HSI gets worse with depth" story. The continuous-depth experiment shows that this apparent trend is largely a **binning artifact**.

At the same time, EMI performs most of the detection, but the empty-hole experiment suggests that its response on this field may be strongly influenced by **soil disturbance**, rather than target metal alone.

This makes the sensors complementary:

- **EMI:** strong localization / disturbed-location detection
- **HSI:** weak detection on this site, but potentially useful spectral/material information

The long-term motivation is therefore **HSI + EMI sensor fusion**, rather than choosing only one modality.

---

## Repository Structure

The local project directory used for the experiments is organized as follows:

```text
landmine_hsi/
│
├── final_1.ipynb
├── final_2.ipynb
│
├── elm_corrected_vnir
├── elm_corrected_vnir.hdr
├── elm_corrected_vnir.cff
│
├── EO_properties_and_Locations_field2.csv
├── GCP_points.csv
├── SVC_spectral_measurements_of_calibration_panels_in_the_scene.csv
│
├── Spectral_Signatures_of_Targets/
│   └── *.sig
│
├── Metal Detector EM61 Lite Data/
│   └── *.xlsx
│
├── figures/
│   ├── fig1_target_map.png
│   ├── fig2_degradation.png
│   ├── fig3_pd_far_by_burial.png
│   ├── fig4_score_overlay.png
│   ├── hsi_depth_continuous.png
│   ├── paper_hsi_by_burial.png
│   ├── paper_hsi_depth_continuous.png
│   ├── paper_sig_coverage.png
│   ├── fig_hsi_vs_emi.png
│   ├── emi_depth_continuous.png
│   ├── fig_control_holes.png
│   ├── report_summary_2x2.png
│   ├── report_emi_material.png
│   └── report_sensor_agreement.png
│
├── results_20260805_0719/
│   ├── targets_full.csv
│   ├── summary.json
│   └── generated figures
│
├── results_emi_20260805_0725/
│   ├── emi_per_target.csv
│   └── generated figures
│
└── .ipynb_checkpoints/
```

> The generated result-folder timestamps depend on when the notebooks are executed, so a new run will create a new `results_YYYYMMDD_HHMM/` or `results_emi_YYYYMMDD_HHMM/` directory.

---

## Notebook Guide

### `final_1.ipynb` — HSI Burial-Depth Analysis

This notebook focuses on the hyperspectral side of the project.

It performs:

1. Environment and GPU-backend initialization.
2. ENVI hyperspectral cube memory mapping.
3. Ground-truth cleaning and target reconciliation.
4. Conversion of longitude/latitude coordinates to image pixels.
5. Field cropping around the seeded targets.
6. RGB target-map generation.
7. SVC `.sig` reference-spectrum loading and target matching.
8. Target/background spectral analysis.
9. Positive-control detection using the calibration panels.
10. GCP and local georeferencing checks.
11. Local-background high-pass filtering.
12. Object-level target detection with:
    - ACE
    - Matched Filter
    - SAM
13. Probability-of-Detection vs. False-Alarm-Rate analysis.
14. Per-target HSI scoring and continuous-depth statistics.
15. Publication figures and result export.

### `final_2.ipynb` — Final HSI + EMI Comparison

This is the main cross-sensor comparison notebook used to reproduce the report-level HSI-vs-EMI analysis.

It performs:

1. HSI loading and local-background high-pass filtering.
2. Per-target RX/local anomaly scoring.
3. EM61 Excel-file discovery and loading.
4. Preference for the combined/leveled EMI file when available.
5. Use of absolute leveled `Channel1_new` when present.
6. Target-to-EMI matching within a **1 m radius**.
7. HSI-vs-EMI detection comparison.
8. Material and responsiveness-class comparison.
9. Target-vs-background EMI AUC calculation.
10. Continuous burial-depth correlation.
11. Surface-vs-buried comparison.
12. Empty-control-hole disturbance analysis.
13. Publication-figure generation.
14. Per-target result export.

### Which notebook should I use?

Use:

```text
final_1.ipynb
```

for the detailed **HSI-only diagnostics, spectral references, SAM/MF/ACE analysis, positive controls, and georeferencing checks**.

Use:

```text
final_2.ipynb
```

for the final **HSI-vs-EMI comparison** and the numerical results reported in the final project report.

---

## Data Description

### Hyperspectral Cube

The VNIR cube used by the notebooks is:

```text
elm_corrected_vnir
elm_corrected_vnir.hdr
elm_corrected_vnir.cff
```

Important properties used by the code:

| Property | Value |
|---|---|
| Shape | 3123 × 6631 × 272 |
| Data type | float32 |
| Format | ENVI / BSQ |
| Approx. spectral range | 398–1002 nm |
| Analysis range | ≤ 900 nm |
| Bands used in notebook run | 226 |
| Coordinate system | Geographic latitude/longitude, WGS-84 |
| Approx. spatial resolution | 9 mm GSD |

The cube is opened with `spectral.open_image()` and memory-mapped to avoid loading the entire image into RAM immediately.

### Target Metadata

```text
EO_properties_and_Locations_field2.csv
```

The notebook uses fields including:

- target ID
- item/type
- longitude
- latitude
- burial depth
- ferrous/non-ferrous label
- class information

The CSV contains duplicate `ID` column names, so the notebook automatically de-duplicates column names before processing.

### Spectral Signatures

```text
Spectral_Signatures_of_Targets/
```

The `.sig` files contain two-column spectral measurements:

```text
Wavelength (nm), Reflectance
```

The notebook scales the reference spectra by **100** because the `.sig` measurements use approximately a 0–1 reflectance scale while the hyperspectral cube uses approximately a 0–100 scale.

In the executed HSI notebook:

- **160 `.sig` files** were found.
- **69 / 131 targets** had at least one matched SVC reference.
- **54 targets** had multiple reference measurements.

When multiple signatures exist for a target, the notebook prefers a clean/cleaned signature over dirt/area-contaminated measurements.

### Calibration Panel Data

```text
SVC_spectral_measurements_of_calibration_panels_in_the_scene.csv
```

This file is used as a positive spectral-control experiment. A SAM detector is applied to the light-gray in-scene panel to verify that the HSI pipeline can detect a strong spectral target.

### Ground-Control Points

```text
GCP_points.csv
```

These points are used to visually inspect georeferencing and to test whether weak HSI target response could be explained by systematic image-to-ground-truth misalignment.

### EM61 Data

```text
Metal Detector EM61 Lite Data/
```

The EMI loader recursively searches for `.xlsx` files and:

1. prefers a **combined** file when available,
2. prefers leveled `Channel1_new`,
3. otherwise uses the first available Channel 1 signal,
4. ignores macOS `._*` metadata files,
5. can stitch individual line files if a combined file is unavailable.

In the executed notebook, the loader selected:

```text
Copy of shift_combined_file.xlsx
```

with:

```text
Channel1_new
2042 EMI readings
```

---

## Target Reconciliation

The original metadata contains 150 rows.

The notebooks classify them as:

| Category | Count |
|---|---:|
| Real seeded targets | 131 |
| Empty control holes | 8 |
| Blank/placeholder rows | 11 |
| Total | 150 |

For the 131 real targets:

| Category | Count |
|---|---:|
| Surface | 31 |
| Shallow | 70 |
| Deep | 30 |
| Metal | 86 |
| Non-metal | 41 |
| Material unspecified | 4 |

The burial-class function in the notebook defines:

```text
surface = 0 cm
shallow = >0 cm and ≤6 cm
deep    = >6 cm
```

The **6 cm shallow/deep boundary is a modeling choice**, which is why the project also analyzes exact continuous depth.

For the direct sensor comparison, **130 / 131 targets** lie within 1 m of an EMI reading. One shallow target is therefore excluded from the paired comparison, leaving:

```text
Surface: 31
Shallow: 69
Deep:    30
Total:  130
```

---

## Hyperspectral Processing

### Local Background Removal

The notebooks use a **31-pixel boxcar local mean**.

Conceptually:

```text
High-pass spectrum = target-region spectrum - local mean spectrum
```

This removes large-scale soil/illumination variation and makes a location stand out relative to its immediate surroundings.

### HSI Detectors

The HSI notebook evaluates:

#### Spectral Angle Mapper (SAM)

Measures spectral-shape similarity based on the angle between spectra.

#### Matched Filter (MF)

Measures how strongly a target-like direction appears relative to the estimated background.

#### Adaptive Cosine Estimator (ACE)

A covariance-aware normalized target detector.

#### RX Anomaly Score

Measures how unusual a spectrum is relative to the local-background distribution.

---

## HSI Object-Level Detection

The HSI notebook generates score maps for:

```text
ACE
MF
SAM
```

Local maxima are matched to known target centers within a positional tolerance.

For all 131 real targets, the executed notebook reported:

| Detector | Pd @ FAR = 1e-3 | Pd @ FAR = 1e-2 |
|---|---:|---:|
| ACE | 0.25 | 0.93 |
| MF | 0.30 | 0.93 |
| SAM | 0.19 | 0.69 |

These results indicate that the pipeline is functioning, but target-level HSI detection is weak at strict false-alarm operating points.

---

## HSI Diagnostic Controls

### Calibration-Panel Positive Control

The light-gray calibration panels produce clear SAM responses.

This is important because it demonstrates that the HSI pipeline is capable of detecting a strong spectral target when sufficient spectral contrast exists.

### Georeferencing Check

For 17 surface targets with matched signatures, the local search produced approximately:

```text
median offset: (-1, 4) pixels
offset scatter: ~11 pixels
mean SAM match gain: 0.092
```

The offset is scattered rather than strongly systematic, so the weak HSI target signal cannot be explained simply by applying one global image shift.

### Target-vs-Background Contrast

The target and local-background spectra are very similar.

The executed HSI diagnostics reported an overall relative target/background difference of approximately:

```text
0.076
```

Only a small fraction of target pixels exceed the background anomaly floor, supporting the interpretation that **low spectral contrast is a major reason for weak VNIR detection**.

---

## EMI Processing

For every target, the code finds the strongest EMI value within:

```text
MATCH_RADIUS_M = 1.0
```

EMI detection uses the **90th percentile of the survey response** as the background-relative operating threshold.

The final comparison is restricted to the 130 targets that are covered by the EMI survey.

---

## Experiment 1 — Three Burial Classes

The report-aligned HSI/EMI comparison gives:

| Burial Class | n | HSI Detection | EMI Detection |
|---|---:|---:|---:|
| Surface | 31 | 0.10 | 0.26 |
| Shallow | 69 | 0.04 | 0.52 |
| Deep | 30 | 0.00 | 0.53 |

This initially looks like opposite depth trends:

- HSI decreases.
- EMI increases and then plateaus.

But the continuous-depth experiment is necessary to determine whether those trends are real.

![HSI vs EMI burial comparison](figures/fig_hsi_vs_emi.png)

---

## Experiment 2 — Continuous Burial Depth

Instead of assigning targets to arbitrary depth bins, the second analysis compares each target's score directly with its exact measured depth.

### HSI

```text
Pearson r = -0.064
p = 0.47
```

There is no statistically meaningful relationship between HSI anomaly score and burial depth.

### EMI

```text
Pearson r = 0.188
p = 0.0324
Spearman rho = 0.144
```

The EMI response has a small but statistically significant positive relationship with burial depth.

![EMI continuous depth analysis](figures/emi_depth_continuous.png)

---

## EMI Target-vs-Background Separability

The project evaluates whether EMI separates known target locations from background survey points.

The executed notebook reported:

```text
All targets vs background AUC      = 0.996
Conductive targets vs background  = 0.996
Inert targets vs background       = 0.993
```

This is extremely strong localization performance.

However, a high target-vs-background AUC does **not** mean that EMI reliably identifies target material.

---

## Material Analysis

Mean EMI response:

```text
Metal      ≈ 2391
Non-metal  ≈ 2353
```

The values are very similar.

Detection by the notebook's EMI-responsiveness grouping was:

```text
Conductive: 0.48
Inert:      0.33
```

Therefore, EMI performs much better at answering:

> **"Is this location anomalous?"**

than:

> **"What material is the buried object made of?"**

---

## Empty Control-Hole Experiment

Eight locations in the metadata are empty holes that were dug and refilled without placing ordnance inside.

They provide a direct soil-disturbance control.

| Group | n | Mean EMI Response | Detection Rate |
|---|---:|---:|---:|
| Empty control holes | 8 | 5204 | 0.88 |
| Real buried targets | 99 | 2601 | 0.53 |

Statistical comparison:

```text
Mann–Whitney p = 0.006
```

The empty holes respond more strongly than the real buried targets.

This suggests that on this field the EMI system may be responding strongly to **soil disturbance associated with digging**, not only to metal or ordnance.

Because only eight control holes are available, this result should be interpreted as **strongly suggestive, but not conclusive**.

![Control-hole EMI experiment](figures/fig_control_holes.png)

---

## Sensor Agreement

At the report operating point:

| Outcome | Targets |
|---|---:|
| HSI only | 4 |
| EMI only | 58 |
| Both | 2 |
| Neither | 66 |

This limited overlap reinforces the idea that the two sensing systems capture different information.

---

## Installation

### 1. Create the Conda Environment

```bash
conda create -n mines python=3.11 -y
conda activate mines
```

### 2. Install CPU/Common Dependencies

```bash
pip install numpy scipy scikit-learn pandas matplotlib spectral rasterio tqdm ipykernel openpyxl
```

### 3. Optional NVIDIA GPU Support

The notebooks use CuPy rather than PyTorch for GPU acceleration.

```bash
pip install cupy-cuda12x
```

The code automatically falls back to NumPy if CuPy is unavailable or incompatible.

### 4. Register the Jupyter Kernel

```bash
python -m ipykernel install --user --name mines --display-name "Python (mines)"
```

---

## Hardware / Memory Notes

The HSI crop used by the notebooks has shape:

```text
1337 × 2787 × 272
```

and occupies approximately:

```text
4.1 GB
```

before additional processing arrays are considered.

`final_2.ipynb` performs the local high-pass operation **in place** to reduce memory use.

GPU acceleration is optional. The notebooks were successfully executed with CuPy on an **NVIDIA GeForce RTX 5070 Laptop GPU**, but CPU execution is also supported and will be slower.

---

## How to Run

### Step 1 — Place the Data

Keep the required files/folders under one project directory matching the repository structure shown above.

### Step 2 — Set `DATA_DIR`

In both notebooks, update:

```python
DATA_DIR = r"D:/landmine_hsi"
```

to the location of your cloned project/data directory.

For example on Linux:

```python
DATA_DIR = "/home/your_username/landmine_hsi"
```

### Step 3 — Run HSI Diagnostics

Open:

```text
final_1.ipynb
```

and run the cells from top to bottom.

This generates the detailed HSI diagnostics, spectral-reference analysis, object-level detector curves, georeferencing checks, and HSI result files.

### Step 4 — Run the HSI + EMI Comparison

Open:

```text
final_2.ipynb
```

and run the cells from top to bottom.

This generates the report-level HSI-vs-EMI analysis, continuous-depth results, material comparison, control-hole experiment, and EMI result files.

---

## Important Reproducibility Note

The notebooks contain multiple HSI diagnostics intended for different purposes.

For reproducing the **final report's sensor-to-sensor detection table**, use the background-relative HSI threshold calculated from the **95th percentile of background RX anomaly scores** in `final_2.ipynb`.

The detailed `final_1.ipynb` ACE/MF/SAM experiments answer a different question: object-level HSI detector behavior and Pd-vs-FAR performance.

Keeping these analyses conceptually separate avoids mixing different HSI scoring definitions.

---

## Generated Outputs

### `final_1.ipynb`

The HSI notebook writes figures and tables including:

```text
figures/
├── fig1_target_map.png
├── fig2_degradation.png
├── fig3_pd_far_by_burial.png
├── fig4_score_overlay.png
├── hsi_depth_continuous.png
├── hsi_per_target_scores.csv
├── results_object_level.csv
├── paper_hsi_by_burial.png
├── paper_hsi_depth_continuous.png
└── paper_sig_coverage.png
```

It also creates:

```text
results_YYYYMMDD_HHMM/
├── targets_full.csv
├── summary.json
└── copies of generated PNG figures
```

### `final_2.ipynb`

The HSI + EMI notebook writes files including:

```text
figures/
├── fig_hsi_vs_emi.png
├── hsi_vs_emi_per_target.csv
├── emi_depth_continuous.png
├── emi_per_target_scores.csv
├── fig_control_holes.png
├── report_summary_2x2.png
├── report_emi_material.png
├── report_sensor_agreement.png
└── hsi_vs_emi_final.csv
```

It also creates:

```text
results_emi_YYYYMMDD_HHMM/
├── emi_per_target.csv
└── copies of generated PNG figures
```

---

## Reproducing the Report-Level Numbers

For the final project conclusions, the most important outputs are:

```text
figures/fig_hsi_vs_emi.png
figures/hsi_vs_emi_per_target.csv
figures/emi_depth_continuous.png
figures/emi_per_target_scores.csv
figures/fig_control_holes.png
results_emi_*/emi_per_target.csv
```

The main paired comparison uses the **130 EMI-covered targets**, not all 131 real seeded targets.

---

## GitHub Data Policy

The hyperspectral cube and raw EM61 files can be large and may also be subject to dataset redistribution terms.

For a public GitHub repository, it is generally better to commit:

```text
README.md
final_1.ipynb
final_2.ipynb
selected figures/
small derived result tables
requirements.txt or environment.yml
LICENSE
```

and avoid committing large/raw data unless redistribution is explicitly permitted.

In particular, the raw hyperspectral binary should normally remain outside Git history.

A suitable `.gitignore` would include entries such as:

```gitignore
.ipynb_checkpoints/

# Large hyperspectral files
elm_corrected_vnir
elm_corrected_vnir.cff

# Raw sensor data
Metal Detector EM61 Lite Data/

# Optional: keep source signatures/data local if redistribution is restricted
Spectral_Signatures_of_Targets/

# Generated timestamped outputs
results_*/
results_emi_*/

# Python
__pycache__/
*.pyc
```

You may choose to keep selected publication-ready files from `figures/` tracked in Git so the README images render directly on GitHub.

---

## Methodological Takeaway

One of the most important lessons from this project is that **categorical depth bins can create a visually convincing trend that is not supported by continuous data**.

For HSI:

```text
Binned result:
0.10 → 0.04 → 0.00
```

looks like a strong depth decline.

But:

```text
Continuous result:
r = -0.06, p = 0.47
```

shows no meaningful depth relationship.

For EMI, both categorical and continuous analyses support an increasing response with burial depth.

---

## Limitations

- SVC reference spectra cover only **69 / 131 targets**.
- SVC coverage is not uniform across burial depths.
- Burial depths are concentrated toward shallow values.
- Only **8 empty control holes** are available.
- HSI target/background spectral contrast is very weak.
- The 6 cm shallow/deep boundary is user-defined rather than a physical sensor threshold.
- Ferrous/non-ferrous labels do not necessarily describe actual EMI responsiveness perfectly.
- The control-hole disturbance interpretation requires confirmation on additional sites or datasets.
- Exact reproduction depends on using the same EM61 leveled/combined input product.

---

## Future Work

Potential extensions include:

- multimodal HSI + EMI sensor fusion,
- SWIR hyperspectral data,
- disturbed-soil spectral detection,
- better target-reference spectra,
- improved conductivity/material ground truth,
- testing on additional seeded fields,
- temporal analysis of soil disturbance after burial,
- learned multimodal fusion models,
- more robust depth-balanced evaluation.

---

## Final Conclusion

The results do not support a simple "HSI works at the surface and EMI takes over with depth" interpretation.

Instead:

- VNIR HSI is weak at almost all depths on this site.
- The apparent HSI depth decline from the three-bin experiment does not survive continuous-depth analysis.
- EMI provides most of the actual detections.
- EMI response increases with depth in the available data.
- EMI is highly effective at separating target locations from background.
- EMI shows weak material discrimination.
- Empty control holes indicate that disturbed soil may be an important part of the EMI signal.
- HSI and EMI therefore appear more useful as **complementary modalities** than as competing alternatives.

---

## Citation

If you use this repository, please cite the associated project report and the original dataset/publications as appropriate.

```bibtex
@misc{hsi_emi_buried_ordnance,
  title = {Hyperspectral vs. Electromagnetic Induction for Buried-Ordnance Detection},
  subtitle = {A Depth and Material-Resolved Comparison on the DRC Site 2 Seeded Field},
  note = {Project report and accompanying analysis code}
}
```

Update the BibTeX entry with the final author, year, institution, repository URL, and publication information before release.

---

## Related Work

The final report identifies the following work as inspiration:

- Lekhak et al. (2025) — hyperspectral buried-target benchmark work.
- Lekhak et al. (2024) — airborne EMI work.

See the final report for the complete citations and context.

---

## License and Data Attribution

The notebooks note that the dataset license should be verified before publication or redistribution.

Before making the repository public:

1. confirm the license of the RIT/DIRS data,
2. confirm whether the raw HSI/EMI files may be redistributed,
3. add the appropriate dataset attribution,
4. choose a separate license for your own source code.

---

## Acknowledgements

This project analyzes co-registered VNIR hyperspectral and EM61-Lite measurements from the RIT/DIRS DRC Site 2 seeded field.

Add the appropriate dataset creators, laboratory, supervisor, collaborators, institution, and funding acknowledgements before public release.
