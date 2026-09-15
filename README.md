# Heat Stress Alerting Simulation Framework

A data-driven simulation framework for evaluating heat-stress severity classification across multiple environmental indices in resource-constrained South Asian settings.
The notebook uses **ERA5 hourly reanalysis data**, simulates spatial variation across **25 sensor nodes using urban heat-island (UHI) offsets**, computes multiple heat-stress indices, and compares their resulting severity classifications against a three-tier alert framework.

## Overview

This project investigates how different heat-stress indicators classify hazardous conditions under the same meteorological input.
Four index pipelines are evaluated:

* **Heat Index (HI)**
* **Environmental Stress Index (ESI)**
* **Simplified Wet-Bulb Globe Temperature (sWBGT)**
* **Air Temperature (Tₐ)**

The simulation is designed around a district-scale scenario in **Mohali, Punjab**, with **Ludhiana used as a station proxy** for the published UPSDMA air-temperature thresholds because district-specific values are not provided in the source framework.

Rather than relying on synthetic weather data, the pipeline derives its meteorological inputs from a real **1991–2025 ERA5 single-point hourly dataset**.

## Research Pipeline

The notebook follows this workflow:

```text
ERA5 hourly CSV
       │
       ▼
Data loading & UTC → IST conversion
       │
       ▼
Meteorological variables
(Tₐ, Td, SSRD)
       │
       ├───────────────┐
       ▼               ▼
Baseline 1991–2020   Event year 2022
       │               │
       ▼               ▼
Index threshold      25 simulated
derivation           sensor nodes
       │               │
       │          UHI temperature
       │          offsets applied
       │               │
       │               ▼
       │        HI / ESI / sWBGT / Tₐ
       │               │
       └───────┬───────┘
               ▼
       Severity classification
        None / Yellow / Orange / Red
               │
               ▼
       Agreement & comparison
           analyses
               │
               ▼
        Tables + Figures
```

## Data

The notebook expects a single ERA5 CSV containing hourly observations including:

* `valid_time`
* `t2m` — 2 m air temperature
* `d2m` — 2 m dew-point temperature
* `ssrd` — surface solar radiation downward
* latitude
* longitude

The supplied dataset spans **1991–2025**, allowing the notebook to extract both the climatological baseline and the 2022 event year from the same source file.

### Preprocessing

The loader performs the following transformations:

* Kelvin → °C for air and dew-point temperature
* UTC → Indian Standard Time (IST)
* Surface solar radiation conversion from hourly J/m² to W/m²
* Removal of duplicated timestamps
* Clipping of dew-point temperature to air temperature where rounding-level inconsistencies occur

Daily statistics are calculated using **IST calendar days**, rather than UTC days.

## Baseline and Event Periods

Two periods are used:

| Period     | Dates     | Purpose                                      |
| ---------- | --------- | -------------------------------------------- |
| Baseline   | 1991–2020 | Derivation of climatological thresholds      |
| Event year | 2022      | Sensor-node simulation and severity analysis |

The complete baseline is kept separate from the event year so that the 2022 analysis does not influence its own thresholds.

## Heat-Stress Indices

### Heat Index (HI)

The notebook implements the Rothfusz regression for conditions at or above 27 °C and a simpler Steadman-based formulation below that threshold.

Relative humidity is estimated from air temperature and dew-point temperature using a Magnus approximation.

### Environmental Stress Index (ESI)

The ESI formulation incorporates:

* Air temperature
* Relative humidity
* Surface solar radiation

following the formulation attributed in the notebook to Moran et al. (2001).

### Simplified WBGT (sWBGT)

A simplified wet-bulb globe temperature formulation is calculated from air temperature and humidity.

The notebook explicitly treats this as a simplified approximation rather than a replacement for full physical WBGT measurement.

### Air Temperature (Tₐ)

Raw air temperature is retained as a fourth comparison pipeline and is classified using the published UPSDMA thresholds.

## Severity Thresholds

All four pipelines use the same three severity levels:

```text
None
Yellow
Orange
Red
```

For **Tₐ**, the notebook uses the published UPSDMA thresholds:

| Tier   | Tₐ threshold |
| ------ | -----------: |
| Yellow |     37.62 °C |
| Orange |     40.39 °C |
| Red    |     43.83 °C |

These values are based on the UPSDMA framework and use **Ludhiana as a proxy for Mohali District**.

For HI, ESI and sWBGT, equivalent thresholds are derived from the real 1991–2020 ERA5 baseline using:

* 75th percentile → Yellow
* 85th percentile → Orange
* 95th percentile → Red

The threshold calculation is performed over the full baseline distribution rather than using a day-of-year moving window.

## Sensor-Node Simulation

The 2022 meteorological series is extended into a simulated district sensor network consisting of:

```text
25 sensor nodes
```

Each node receives a randomly generated UHI temperature offset sampled uniformly between:

```text
1.5 °C and 3.5 °C
```

For every simulated node, the notebook recomputes:

* Air temperature
* Relative humidity
* Heat Index
* Environmental Stress Index
* Simplified WBGT

The maximum value across all 25 nodes is then used as the district-level value for severity classification.

This allows the framework to represent localized hot spots without simply averaging away spatial extremes.

## Severity Classification

Each hourly value is classified into:

```text
None     → 0
Yellow   → 1
Orange   → 2
Red      → 3
```

Classification is performed independently for:

* HI
* ESI
* sWBGT
* Tₐ

The notebook also aggregates hourly classifications into **daily peak severity**, where the highest severity reached during a day becomes that day's classification.

## Evaluation

The framework produces several comparison analyses.

### Severity distribution

The notebook calculates the number and percentage of days and hours assigned to each severity tier for each index pipeline.

### Pairwise tier agreement

The four pipelines are compared pairwise to determine how similarly they classify heat-stress severity.

### Quadratic-weighted Cohen's kappa

Because the severity categories are ordinal, the notebook implements **quadratic-weighted Cohen's kappa** directly.

The weighting treats larger disagreements differently from adjacent-tier disagreements. For example, a disagreement between `None` and `Red` is penalized more heavily than a disagreement between `Yellow` and `Orange`.

### Threshold comparison

The notebook compares:

1. Published UPSDMA Tₐ thresholds with their ERA5-derived equivalents.
2. All six pairwise combinations of the four index pipelines.

## Outputs

The pipeline generates result tables and figures in:

```text
/content/heat_stress_results/
```

The notebook includes visualizations for:

* Daily peak severity composition by index
* Index time series with severity thresholds
* Pairwise tier agreement
* Kappa agreement matrix
* Threshold similarity/comparison

The optional final cell packages the generated results into a ZIP archive for download from Google Colab.

## Running the Notebook

### Option 1 — Google Colab

Open the notebook in Google Colab and run the cells sequentially.
When prompted, upload the ERA5 CSV.
The notebook intentionally **does not provide a synthetic-data fallback**. If the ERA5 dataset is missing, the pipeline raises an error instead of silently generating replacement data.

### Option 2 — Local Jupyter environment

The analytical portions of the notebook use standard Python scientific-computing libraries:

```text
Python
NumPy
pandas
Matplotlib
```

The notebook also contains Google Colab-specific upload/download functionality, so the file-upload cells should be adapted when running outside Colab.

## Repository Structure

A suggested repository structure is:

```text
.
├── heat_stress_colab_final.ipynb
├── README.md
├── data/
│   └── README.md
├── results/
│   └── README.md
└── LICENSE
```

The ERA5 dataset itself is not required to be committed to the repository and may be kept separately because of file size and data-distribution considerations.

## Important Assumptions and Limitations

This is a **simulation and analytical framework**, not a deployed heat-alert system.

Several limitations should therefore be considered:

* The sensor network is simulated rather than composed of physical devices.
* UHI effects are represented as temperature offsets rather than dynamically modeled urban microclimates.
* A single ERA5 grid point is used as the meteorological source.
* Ludhiana thresholds are used as a proxy for Mohali because district-specific UPSDMA values are not available in the framework used.
* HI, ESI and sWBGT thresholds are percentile-derived rather than directly published operational thresholds.
* The sWBGT calculation is a simplified approximation and does not reproduce full WBGT instrumentation.
* Taking the maximum of the 25 simulated nodes emphasizes localized extremes and should not be interpreted as a spatial average.
* The framework evaluates classification behavior; it does not by itself establish clinical or epidemiological thresholds.

## Reproducibility

The sensor simulation uses a fixed random seed:

```python
seed = 0
```

This makes the generated UHI offsets reproducible between runs, assuming the same input data and configuration.
The principal simulation parameters are defined near the beginning of the notebook, including:

```python
N_NODES = 25
UHI_MIN = 1.5
UHI_MAX = 3.5

BASELINE_START = "1991-01-01"
BASELINE_END   = "2020-12-31"

SIM_START = "2022-01-01"
SIM_END   = "2022-12-31"
```

## Project Context

This repository contains the computational component of a broader research effort on **vulnerability-differentiated heat-stress alerting for resource-constrained South Asian regions**.
The framework is intended to support research into how different heat-stress indicators and localized temperature variation can influence alert classification, particularly in settings where dense environmental sensing and conventional WBGT instrumentation may be difficult to deploy.

## Status

**Research prototype / simulation framework**

The notebook is intended for research, experimentation, comparison of heat-stress indices, and development of an eventual resource-constrained alerting architecture. It should not be interpreted as a validated operational public-health warning system.

## License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for the full license text.

---

### Citation

Not Available
