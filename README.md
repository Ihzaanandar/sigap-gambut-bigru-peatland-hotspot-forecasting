# SIGAP-GAMBUT: BiGRU-Based Next-Day Peatland Hotspot Forecasting

**A time-series deep-learning project exploring next-day peatland hotspot risk in Ogan Komering Ilir (OKI), South Sumatra, Indonesia.**

![Python](https://img.shields.io/badge/Python-Data%20Science-blue?logo=python&logoColor=white)
![TensorFlow](https://img.shields.io/badge/TensorFlow-Keras-orange?logo=tensorflow&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![GEMASTIK](https://img.shields.io/badge/GEMASTIK-XIX%202026-5B6DAA)

SIGAP-GAMBUT investigates whether historical environmental and hotspot-related signals can help identify the risk of a peatland hotspot event on the following day. The notebook documents data checks, temporal feature preparation, GRU/BiGRU experiments, recall-oriented optimization, final training, and a notebook-based inference prototype.

> **Scope note:** this is a research and portfolio prototype, not an operational fire-alert system. The model predicts next-day hotspot occurrence at the study-area level; it does not predict the precise coordinates of future hotspots.

## Table of Contents

- [Project Overview](#project-overview)
- [Documented Benchmark](#documented-benchmark)
- [Methodology](#methodology)
- [Repository Contents](#repository-contents)
- [Data Requirements](#data-requirements)
- [Model Artifacts](#model-artifacts)
- [Run the Notebook](#run-the-notebook)
- [Limitations](#limitations)
- [Data Sources and Acknowledgments](#data-sources-and-acknowledgments)

## Project Overview

| Item | Description |
|---|---|
| Project | SIGAP-GAMBUT |
| Context | GEMASTIK XIX 2026 Data Science project |
| Study area | Peatland area in Ogan Komering Ilir (OKI), South Sumatra, Indonesia |
| Prediction task | Binary classification of next-day hotspot occurrence |
| Target | `peat_hotspot_tomorrow` |
| Model | Bidirectional Gated Recurrent Unit (BiGRU) |
| Input window | Seven historical days to predict day H+1 |
| Input channels | 21 selected numerical features plus `calendar_observed` (22 channels per timestep) |
| Development data | 2018–2023 |
| Walk-forward evaluation | 2020–2023 |
| Execution environment | Google Colab and Google Drive, as configured in the notebook |

### Why this project?

Peatland fires can have substantial environmental, ecological, and public-health consequences. This project explores a time-series classification approach for estimating whether at least one VIIRS hotspot will be observed in the target peatland area on the next day. It focuses on temporal validation and recall-oriented model development rather than treating accuracy as the only measure of performance.

## Documented Benchmark

The notebook records the following results for a recall-optimized experiment evaluated with walk-forward development folds for 2020–2023.

| Metric | Documented benchmark |
|---|---:|
| Accuracy | **85.03%** |
| Precision | **31.14%** |
| Recall | **37.61%** |
| Balanced accuracy | **63.00%** |
| F1-score | **32.61%** |
| Matthews correlation coefficient (MCC) | **0.2459** |
| PR-AUC | **0.3694** |

The architecture-selection stage documented 85.13% raw development accuracy and 30.66% recall for BiGRU64. In the subsequent recall-optimization experiment, recall increased to 37.61% while accuracy was 85.03%.

**Interpretation:** these results should be assessed together. Accuracy can be misleading when hotspot events are imbalanced, so precision, recall, balanced accuracy, PR-AUC, MCC, and year-by-year stability also matter. The numbers above are recorded experiment results, not a guarantee that a fresh run will reproduce identical values across environments.

## Methodology

The notebook is organized around the following workflow:

1. **Data and provenance audit:** check available project artifacts, validate the daily-cache schema, and audit the historical grid-day panel where available.
2. **Calendar completion:** build a continuous daily timeline and track missing calendar days using `calendar_observed`.
3. **Feature screening:** document Chi-Square screening and use the selected `no_seasonal_21` feature set.
4. **Temporal sequence generation:** use seven historical days to predict the next day, without directly using target-day features.
5. **Model experiments:** compare GRU/BiGRU configurations and record recall-oriented optimization.
6. **Walk-forward evaluation:** evaluate chronologically across development years 2020–2023.
7. **Final training:** train the selected BiGRU64 configuration using development data from 2018–2023.
8. **Inference and visualization:** provide date-based predictions, a seven-day external-input interface, an interactive notebook widget, and historical hotspot maps.

### Main model configuration

| Component | Configuration |
|---|---|
| Architecture | BiGRU with 64 units, followed by a dense layer with 16 units |
| Dropout | 0.20 |
| Loss | Binary focal cross-entropy |
| Focal-loss gamma | 4.0 |
| Learning rate | 0.00075 |
| Sample weighting | `class_year_balanced` |
| Input window | 7 days |
| Input channels | 22 per timestep |
| Feature set | `no_seasonal_21` plus `calendar_observed` |

## Repository Contents

The project folder currently uses this layout:

```text
sigap-gambut-bigru-peatland-hotspot-forecasting/
├── README.md
├── sample_dataset_gambut.xlsx
├── sigap_gambut_bigru_peatland_hotspot_forecasting.ipynb
└── models/
```

- **`README.md`**: project overview, documented results, data requirements, and instructions.
- **`sample_dataset_gambut.xlsx`**: sample workbook included with the project. It is provided as a sample/reference file; the notebook's main training pipeline does not load this Excel workbook directly.
- **`sigap_gambut_bigru_peatland_hotspot_forecasting.ipynb`**: the main notebook containing the workflow, experiments, evaluation, training, and inference prototype.
- **`models/`**: directory for exported model artifacts, if you choose to include them in the repository. See [Model Artifacts](#model-artifacts) below.

> **File extension check:** Windows may hide known file extensions in File Explorer. The README assumes the sample workbook is named `sample_dataset_gambut.xlsx`. If the actual extension is different, update the filename in this section to match it exactly.

## Data Requirements

The sample Excel workbook listed above does **not** replace the source data required by the training pipeline. The notebook is configured for Google Colab and looks for the main project data in Google Drive under `MyDrive/gemastik`.

### Primary daily cache

The notebook checks these cache files in order and uses the first one that exists:

```text
MyDrive/gemastik/data/model/training_simple_daily/cache/daily_dataset_simple.parquet
MyDrive/gemastik/data/model/training_v3_daily/cache/daily_dataset_v3.parquet
```

At least one of these daily caches is required to continue the modeling pipeline. If neither exists, the notebook raises a `FileNotFoundError`.

### Supporting artifacts

The notebook also checks for the following files for provenance audits and/or historical map functionality:

```text
MyDrive/gemastik/data/model/final/train_dataset_oki_hotspot_next_day_2018_2025.csv
MyDrive/gemastik/spatial/grid_oki_10km.geojson
```

The grid geometry may alternatively be located at:

```text
MyDrive/gemastik/scrapping data/spatial/grid_oki_10km.geojson
```

Some audit, map, and reporting sections may be unavailable if the corresponding supporting files are missing. Upstream manifests are optional.

## Model Artifacts

The notebook saves final-training artifacts to the following Google Drive directory:

```text
MyDrive/gemastik/data/model/gru_gemastik_2026_full_recall_optimized/models/
```

The expected filenames are:

```text
bigru64_focal_g4_lr000075_recall_optimized.keras
bigru64_recall_optimized_preprocessor.joblib
bigru64_recall_optimized_manifest.json
```

The local repository also has a `models/` directory. If you want to distribute the trained artifacts with the project, copy the relevant files into that local directory and update this README if the filenames differ. The screenshot of the project root confirms that the folder exists, but the files inside it should be checked separately before you claim that model artifacts are included.

Check file sizes and data-distribution terms before committing model or data files to a public repository. Large files may need Git LFS or an external download link; do not commit credentials, private paths, or restricted data.

## Run the Notebook

1. Clone or download this repository.
2. Open `sigap_gambut_bigru_peatland_hotspot_forecasting.ipynb` in [Google Colab](https://colab.research.google.com/) or Jupyter Notebook.
3. Make sure the required daily-cache file and any supporting artifacts are available at the paths listed in [Data Requirements](#data-requirements).
4. Run the notebook cells from top to bottom. The notebook includes a dependency-installation cell for additional packages.
5. Review the data-audit output before interpreting the downstream results.
6. To regenerate the final model, execute the final-training section. Training may take time, and results can vary across environments and hardware.

The notebook uses Google Colab paths under `/content/drive/MyDrive/gemastik`. Running it locally requires changing the Google Drive mounting code and relevant paths. The included Excel sample is not sufficient by itself to reproduce the full training pipeline unless it is independently verified and converted to the expected input schema.

## Limitations

- **Daily, region-level output:** the model estimates next-day hotspot occurrence for the study area. It does not forecast exact hotspot coordinates.
- **Historical maps show observed data:** the map section visualizes historical hotspot observations by grid; it is not a spatial forecast produced by the BiGRU model.
- **No automatic real-time ingestion:** the notebook does not automatically fetch live VIIRS or ERA5-Land data. Prediction for future dates requires seven days of external features prepared using the same upstream definitions.
- **Risk score is not a calibrated probability:** the sigmoid output is treated as a score and compared with a threshold derived from development data.
- **Legacy period:** 2024–2025 has appeared in the project's previous experimental history. Its optional re-evaluation is disabled by default and should not be described as a pristine, never-seen test set.
- **Reproducibility:** performance depends on the source data, upstream feature generation, package versions, and hardware.

## Data Sources and Acknowledgments

The notebook documents the upstream data pipeline as using or referencing the following sources and products:

- [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov/) for VIIRS fire/hotspot observations.
- [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/) for ERA5-Land meteorological and land-surface variables.
- [Global Forest Watch](https://www.globalforestwatch.org/) / WRI peatland mapping as a peat-area mask.
- [WorldPop](https://www.worldpop.org/) as contextual population data, not as a final model predictor.
- [geoBoundaries](https://www.geoboundaries.org/) for administrative boundary data.

The exact source files, preprocessing decisions, and derived features should be checked against the manifests and upstream data-preparation workflow used in the original project.

---

**Keywords:** peatland hotspot prediction, wildfire risk, time-series classification, BiGRU, GRU, TensorFlow, Keras, environmental data science, GEMASTIK XIX 2026, Ogan Komering Ilir, South Sumatra.
