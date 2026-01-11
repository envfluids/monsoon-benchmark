# Monsoon Onset Reliability Analysis

A Python script for evaluating the reliability of probabilistic monsoon onset forecasts compared against IMD (India Meteorological Department) observations over the Core Monsoon Zone (CMZ).

## Overview

This script performs reliability analysis by comparing ensemble forecast probabilities with observed monsoon onset events, generating reliability diagrams that show how well forecast probabilities correspond to actual occurrence frequencies.


## Required Data Files
1. **Model Data**: NetCDF files with precipitation forecasts
   - Both the forecast and ground truth (IMD data and threshold) needs to be on the same grid. This generally requires regridding the forecast output to the IMD grid. This can be done using **CDO** : `cdo rempacon,<gridfile.txt> <input_file.nc> <output_file.nc>`  
   - Format: `{year}.nc`
   - Variables: `tp` (total precipitation) daily precip in mm
   - Dimensions: `init_time/time`: initialization date in datetime64[ns], `day`/`step`: forecast step (0-35 in days) in int64, `lat`, `lon`, `number/member`: ensemble member number (1,2,3...n, where n is the total number of ensemble members) in int64 

2. **IMD Rainfall Data**: NetCDF files with observed rainfall
   - Format: `data_{year}.nc` or `{year}.nc`
   - Variables: `RAINFALL`
   - Dimensions: `lat`/`latitude`, `lon`/`longitude`, `time`/`TIME`

3. **Threshold Data**: NetCDF file 
   - Variables: `MWmean`: Monsoon onset threshold values
   - Dimensions: `lat`, `lon`
   - Resolution: Must match IMD data resolution

## Methodology

### Onset Detection Algorithm
The script uses a **5-day wet spell criterion**:
1. **First day condition**: Daily rainfall > 1 mm
2. **Cumulative condition**: 5-day rolling sum > local threshold
3. **Date filter options**:
   - **MOK enabled**: Only considers onset after June 2nd (Monsoon Onset over Kerala date)
   - **MOK disabled**: Considers onset from May 1st onwards

### Reliability Calculation
1. **Probability Bins**: 10 equal bins from 0.0 to 1.0 (0.0-0.1, 0.1-0.2, ..., 0.9-1.0)
2. **Ensemble Probability**: Fraction of members predicting onset in each day bin
3. **Conditional Frequency**: Observed occurrence rate for each probability bin


### Spatial Focus
Analysis is automatically restricted to the **Core Monsoon Zone (CMZ)** using resolution-dependent polygon coordinates:
- **1-degree resolution**: High-resolution polygon with 22 vertices
- **2-degree resolution**: Medium-resolution polygon with 17 vertices  
- **4-degree resolution**: Coarse-resolution polygon with 9 vertices

## Usage

### Command Line Interface

```bash
# Full example with MOK filter (default) - Days 1-15
python onset_reliability_diagram_cmz.py \
    --model_forecast_dir "../../model_forecast_data/ngcm51/climatology/tp_2p0" \
    --imd_folder "../../imd_rainfall_data/2p0" \
    --thres_file "../../imd_onset_threshold/mwset2x2.nc4" \
    --mem_num 51 \
    --max_forecast_day 15 \
    --save_path "./output" \
    --years 2019 2020 2021 2022 2023 2024 \
    --file_pattern "{}.nc" \
    --date_filter_year 2024 \
    --mok

# Full example with MOK filter - Days 16-30
python onset_reliability_diagram_cmz.py \
    --model_forecast_dir "../../model_forecast_data/ngcm51/climatology/tp_2p0" \
    --imd_folder "../../imd_rainfall_data/2p0" \
    --thres_file "../../imd_onset_threshold/mwset2x2.nc4" \
    --mem_num 51 \
    --max_forecast_day 30 \
    --save_path "./output" \
    --years 2019 2020 2021 2022 2023 2024 \
    --file_pattern "{}.nc" \
    --date_filter_year 2024 \
    --mok
```

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `--model_forecast_dir` | str | Yes | - | Directory containing model forecast data files  |
| `--imd_folder` | str | Yes | - | Directory containing IMD rainfall NetCDF files |
| `--thres_file` | str | Yes | - | Path to monsoon onset threshold NetCDF file |
| `--mem_num` | int | Yes | - | Number of ensemble members to be used (should be ≤ total available members) |
| `--save_path` | str | Yes | - | Output directory for results and figures |
| `--years` | list | Yes | - | Years to process (space-separated integers) |
| `--max_forecast_day` | int | No | 15 | Maximum forecast day (15 for days 1-15 analysis or 30 for days 16-30 analysis) |
| `--file_pattern` | str | No | `{}.nc` | File naming pattern for forecast files (single file per year) |
| `--date_filter_year` | int | No | 2024 | Year to use for date filtering (default: 2024; Change to 2023 for FuXi) |
| `--mok` | flag | No | True | Enable MOK filter (onset after June 2nd only) |
| `--no-mok` | flag | No | False | Disable MOK filter (allow onset from May 1st) |

### Day Bins

The script automatically sets day bins based on `max_forecast_day`:
- **15-day forecasts**: Days 1-5, 6-10, 11-15
- **30-day forecasts**: Days 16-20, 21-25, 26-30

## Output Files

### 1. Reliability Diagram (PNG)
- **Filename**: `reliability_{max_forecast_day}day.png`
- **Content**: 
  - Main plot: Reliability curve with error bars showing observed vs. forecast probabilities
  - Secondary y-axis: Logarithmic frequency histogram showing distribution of forecast probabilities
  - Perfect reliability line (1:1 diagonal) for reference
  - Grid lines and proper axis labels

### 2. Reliability Results (CSV)
- **Filename**: `reliability_results_{max_forecast_day}day_mok.csv` (with MOK) or `reliability_results_{max_forecast_day}day_no_mok.csv` (without MOK)
- **Columns**:
  - `Bin_Range`: Probability bin range (e.g., "0.0-0.1")
  - `N_Forecasts`: Number of forecasts in bin
  - `Mean_Forecast_Prob`: Average forecast probability in bin
  - `Observed_Frequency`: Actual occurrence rate (reliability)
  - `Frequency`: Relative frequency of forecasts in bin

