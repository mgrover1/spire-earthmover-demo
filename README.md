<p align="center">
  <img src="images/spire-logo.svg" alt="Spire Global" width="320">
</p>

# Spire AI-S2S on Earthmover

A reproducible walkthrough of **Spire's AI-driven Sub-Seasonal-to-Seasonal (AI-S2S) hindcast** (`Spire/saifs-s2s-hindcast`) on the [Earthmover Data Marketplace](https://www.earthmover.io/marketplace/), opened straight into `xarray` through the Arraylake catalog.

The walkthrough notebook is written for **energy desks** evaluating extended-range (week 1–6) signal over North America: a 200-member, 46-day, daily-resolved ensemble shipped as **pre-aggregated statistics** (mean, standard deviation, percentiles, climatological probabilities, anomalies) plus weather-regime probabilities.

---

## What's in this repo

| Path | Purpose |
|-|-|
| `notebooks/Spire_AI_S2S_on_Earthmover.ipynb` | The webinar walkthrough, North America focus (Colab-ready; Project Pythia template) |
| `notebooks/Spire_AI_S2S_on_Earthmover_India.ipynb` | Asia webinar variant of the walkthrough: Indian summer monsoon focus (rainfall anomaly + 850 hPa flow, T-max, metro rainfall fans, regional area means) |
| `notebooks/Spire_AI_S2S_seam_diagnostic.ipynb` | Diagnosis of the 0°/360° longitude seam artifact, the post-processing fix, and the checks to verify a repaired store |
| `pyproject.toml` + `uv.lock` | `uv`-managed Python environment, pinned for reproducibility |
| `images/spire-logo.svg` | Spire word-mark (CC BY-SA 4.0 via Wikimedia Commons) |
| `images/ATTRIBUTION.md` | Logo provenance & license |

---

## Prerequisites

1. **Python 3.12** (the `arraylake` client requires `>=3.12,<3.15`).
2. **[`uv`](https://docs.astral.sh/uv/)** for environment + dependency management.
   Install on macOS / Linux:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```
3. **An Earthmover account** with access to the `Spire/saifs-s2s-hindcast` repo. Sign up or browse the listing at <https://app.earthmover.io>.

> Map plots use `cartopy`. On Apple Silicon `uv` installs the prebuilt wheel directly — no manual `geos`/`proj` setup needed.

---

## Quickstart

From a fresh clone:

```bash
# 1. Resolve and install the locked environment
uv sync

# 2. Log in to Earthmover (browser flow; one-time per machine)
uv run arraylake auth login

# 3. Launch JupyterLab and open the notebook
uv run jupyter lab notebooks/Spire_AI_S2S_on_Earthmover.ipynb
```

The India notebook runs the same way: `uv run jupyter lab notebooks/Spire_AI_S2S_on_Earthmover_India.ipynb`.

The diagnostic notebook runs the same way: `uv run jupyter lab notebooks/Spire_AI_S2S_seam_diagnostic.ipynb`.

Or open the notebook in [Google Colab](https://colab.research.google.com/) and run the first setup cell, which installs the dependencies and authenticates to Arraylake.

---

## What the notebooks show

### `Spire_AI_S2S_on_Earthmover.ipynb` — webinar walkthrough

| Section | What it does |
|-|-|
| **Open the dataset** | One `xr.open_zarr` call per zarr group (`mean_stddev`, `percentiles`, `probabilities`, `anomalies`, `regimes`); picks the latest valid issuance |
| **500 hPa pattern, wks 1–6** | Anomaly fill + absolute height contours over North America |
| **T-max anomaly, wks 1–6** | Weekly-mean 2-m T-max anomaly (°F vs ERA5 1991–2020) over North America |
| **Weather regimes** | Daily CONUS regime probability bars |
| **Load-center fans** | 200-member percentile fans at Houston, Chicago, New York and Washington |
| **ISO area means** | Cosine-weighted PJM / MISO / ERCOT / NYISO T-max anomaly curves over the full horizon |

### `Spire_AI_S2S_on_Earthmover_India.ipynb` — Asia webinar walkthrough (Indian monsoon)

Same open recipe and helpers as the North America notebook, with the regional sections re-pointed at the Indian summer monsoon.

| Section | What it does |
|-|-|
| **Rainfall + 850 hPa flow, wks 1–6** | Weekly-mean precipitation anomaly (mm/day vs ERA5 1991–2020) with 850 hPa ensemble-mean wind vectors over India |
| **T-max anomaly, wks 1–6** | Weekly-mean 2-m T-max anomaly (°C) over India |
| **Metro rainfall fans** | 200-member percentile fans of daily rainfall at Delhi, Mumbai, Kolkata and Chennai |
| **Regional area means** | Cosine-weighted rainfall anomaly curves for Northwest, Central, South Peninsula and Northeast India boxes |

### `Spire_AI_S2S_seam_diagnostic.ipynb` — seam artifact: diagnose and verify the fix

| Section | What it does |
|-|-|
| **Reproduce** | Global map and Western Europe zoom of the seam at 0°, column profiles for four fields, seam ratio vs lead |
| **Fix** | 3-point zonal filter on the 12 columns around 0°; before/after seam ratio and collateral on mean, stddev, percentiles and anomalies |
| **Climatology check** | The implied climatology's one-column step at the wrap |
| **Verification recipe** | Pass/fail seam-ratio report across every variable in the store |
| **Per-member results** | Summary of the raw-GRIB analysis run on OSC Cardinal (all 200 members affected from ~day 15) |

---

## The dataset at a glance

| Property | Value |
|-|-|
| **Provider** | Spire Global |
| **Access path** | `Spire/saifs-s2s-hindcast` on Earthmover / `arraylake` (real-time sibling: `Spire/saifs-s2s-daily`, 15-issuance rolling window) |
| **Record** | 1096 preallocated `reference_time` slots; 221 valid issuances to date (2026-01-01 → 2026-08-09), NaT-filled where empty and not chronologically ordered on disk |
| **Forecast horizon** | 46 days, daily resolved |
| **Ensemble size** | 200 probabilistic members |
| **Grid** | 0.5° global, 361 × 720, longitudes `[0, 359.5]` |
| **Format** | Zarr v3 (Icechunk-backed), one global map per chunk |
| **Headline variables** | 2-m T-max / T-min, 10-m and 100-m wind, MSLP, precipitation, downward shortwave, geopotential height (500 / 850 hPa), thickness |
| **Underlying observations** | Spire's commercial smallsat constellation: GNSS radio occultation, GNSS-R soil moisture, ocean surface winds, SST |

See Spire's [AI-S2S launch announcement](https://spire.com/blog/weather-climate/spire-global-unveils-its-ai-s2s-model-with-groundbreaking-long-range-weather-forecasting/) for the science background.

---

## Troubleshooting

| Symptom | Fix |
|-|-|
| `arraylake auth login` opens but never returns | Complete the browser flow on the same machine; on remote/SSH boxes use a personal access token via `ARRAYLAKE_TOKEN` instead. |
| `KeyError: 'Spire/saifs-s2s-hindcast'` | Your account isn't entitled to this repo yet — request access on the Earthmover marketplace listing. |
| `ValueError: unable to decode ... zarr_format=3` | You're on `zarr<3`. `uv sync` pins `zarr>=3`; if you're outside `uv`, upgrade explicitly. |
| Cartopy fails to render coastlines on first run | Cartopy downloads the Natural Earth shapefiles on first call (~10 MB). Re-run the cell. |
| `.sel(reference_time=...)` raises on NaT | The store has NaT-filled slots. Select the exact `reference_time` first, then use `method='nearest'` on lat/lon, as the notebook does. |

---

## Links

- [Spire AI-S2S model overview](https://spire.com/blog/weather-climate/spire-global-unveils-its-ai-s2s-model-with-groundbreaking-long-range-weather-forecasting/)
- [Earthmover platform documentation](https://docs.earthmover.io/)
- [`arraylake` on PyPI](https://pypi.org/project/arraylake/)
- [Project Pythia Foundations](https://foundations.projectpythia.org/) — the notebook template used here
- [NOAA CPC Week 3-4 outlook](https://www.cpc.ncep.noaa.gov/products/predictions/WK34/) — public S2S benchmark

---

## Logo attribution

The Spire word-mark is included from [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Spire_Logo.svg) under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Full attribution and trademark notice in [`images/ATTRIBUTION.md`](images/ATTRIBUTION.md). This repository is not an official Spire product.
