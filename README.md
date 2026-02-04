# monsoon-benchmark

Benchmarking suite for Indian monsoon onset prediction across climatology, deterministic models, and probabilistic (ensemble) models. The repository includes evaluation scripts, reliability and skill score analysis, and example datasets on a 4° grid.

## Paper

This repository is released alongside the preprint submitted to arXiv [`2602.03767`](https://arxiv.org/abs/2602.03767).

**Decision-oriented benchmarking to transform AI weather forecast access: Application to the Indian monsoon**  
Rajat Masiwal, Colin Aitken, Adam Marchakitus, Mayank Gupta, Katherine Kowal, Hamid A. Pahlavan, Tyler Yang, Y. Qiang Sun, Michael Kremer, Amir Jina, William R. Boos, and Pedram Hassanzadeh  
arXiv: 2602.03767

## Data Availability
All data used in this repository is publicly available on [Zenodo](https://doi.org/10.5281/zenodo.18407547). Once downloaded, the files can be moved into this repository following the structure outlined below.

## Repository Layout

- `codes/` — analysis scripts (MAE/FAR/MR, reliability, skill scores, threshold generation)
- `complete_onset_benchmarking_climatology.sh` — end-to-end pipeline for climatology baseline
- `complete_onset_benchmarking_deterministic_models.sh` — end-to-end pipeline for deterministic models
- `complete_onset_benchmarking_probabilistic_models.sh` — end-to-end pipeline for probabilistic models
- `imd_rainfall_data_4p0/` — IMD rainfall observations (1901–2024, 4° grid)
- `imd_rainfall_threshold_4p0/` — 5‑day wet spell threshold file (`mwset4x4.nc4`)
- `rainfall_4p0/` — example model forecast datasets (4° grid)
- `ind_map_shpfile/` — India boundary shapefile for plotting

## Requirements

### Python

The analysis scripts rely on the following Python packages:

- `numpy`
- `xarray`
- `matplotlib`
- `geopandas`
- `scipy`
- `seaborn`
- `h5netcdf`

Install with:

```bash
pip install numpy xarray matplotlib geopandas scipy seaborn h5netcdf
```

### Optional: CDO for Regridding

Model forecasts should be on the same grid as IMD observations. If needed, you can regrid with CDO using the grid files in `codes/cdo_grid_config/`:

```bash
cdo rempacon,codes/cdo_grid_config/4p0_grid_ind.txt input.nc output.nc
```

## Data Expectations

### IMD Observations

- Location: `imd_rainfall_data_4p0/`
- Format: yearly NetCDF files named `{year}.nc`
- Variable: `RAINFALL` (daily precipitation in mm)
- Dimensions: `lat`/`lon` and `TIME`

### Thresholds

- Location: `imd_rainfall_threshold_4p0/mwset4x4.nc4`
- Variable: `MWmean`
- Generation: `codes/onset_threshold_imd/imd_onset_threshold.py`

### Model Forecasts

- Location: `rainfall_4p0/<MODEL_NAME>/{year}.nc`
- Variable: `tp` (daily precipitation in mm)
- Dimensions: `time`, `day`, `lat`, `lon`
- For ensemble models: additional `number` dimension

## Quickstart

### 1) Climatology Baseline

```bash
./complete_onset_benchmarking_climatology.sh
```

### 2) Deterministic Model (example)

```bash
BASE_MODEL_NAME=FuXi \
MODEL_FORECAST_DIR=./rainfall_4p0/FuXi \
./complete_onset_benchmarking_deterministic_models.sh
```

### 3) Probabilistic Model (example)

```bash
BASE_MODEL_NAME=NeuralGCM \
MODEL_FORECAST_DIR=./rainfall_4p0/NeuralGCM \
MEM_NUM=50 \
./complete_onset_benchmarking_probabilistic_models.sh
```

All scripts write outputs under `./output/<MODEL_NAME>` by default.

## Evaluation Windows and Periods

Each pipeline supports common evaluation periods via the `PERIOD` environment variable:

- `recent`: 2019–2024
- `extended`: 1965–1978 and 2019–2024
- `common`: 2004–2021
- `full`: 1965–2024
- `custom`: set `CUSTOM_YEARS` to a space‑separated list

Examples:

```bash
PERIOD=recent ./complete_onset_benchmarking_deterministic_models.sh

PERIOD=custom CUSTOM_YEARS="2010 2011 2015 2020 2021" \
  ./complete_onset_benchmarking_probabilistic_models.sh
```

## Outputs

### MAE / FAR / MR (Deterministic and Climatology)

- NetCDF metrics by grid cell (MAE, FAR, Miss Rate)
- Spatial plots with India boundary and CMZ overlays

### Probabilistic Reliability

- Reliability diagrams PNG
- Binned reliability CSVs

### Probabilistic Skill Scores

- Overall skill scores CSV
- Binned skill scores CSV
- Heatmap PNG

## Script Details

For deeper configuration options and method descriptions, see:

- `codes/mae_far_mr_climatology/README.md`
- `codes/mae_far_mr_deterministic_models/README.md`
- `codes/mae_far_mr_probablistic_models/README.md`
- `codes/realibility_diagram/README.md`
- `codes/probabilistic_skill_scores/README.md`
- `codes/onset_threshold_imd/README.md`

# Citation

```bibtex
@misc{masiwal2026decisionorientedbenchmarkingtransformai,
      title={Decision-oriented benchmarking to transform AI weather forecast access: Application to the Indian monsoon},
      author={Rajat Masiwal and Colin Aitken and Adam Marchakitus and Mayank Gupta and Katherine Kowal and Hamid A. Pahlavan and Tyler Yang and Y. Qiang Sun and Michael Kremer and Amir Jina and William R. Boos and Pedram Hassanzadeh},
      year={2026},
      eprint={2602.03767},
      archivePrefix={arXiv},
      primaryClass={cs.LG},
      url={https://arxiv.org/abs/2602.03767},
}
```