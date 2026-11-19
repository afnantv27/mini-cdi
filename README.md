# Mini-CDI: Palakkad paddy rainfall risk

A small version of a forecast → impact → payout pipeline: IMD rainfall observations, weather forecasts, and a parametric insurance trigger for paddy in Palakkad district, Kerala.

## Progress
- [x] Step 0 — Setup (conda env `minicdi`, Git + GitHub)
- [x] Step 1 — Understand the IMD rainfall data
- [ ] Step 2 — Download 1991–2025 and clip to Palakkad
- [ ] Step 3 — Explore rainfall (climatology, wet/dry years, 2018 floods)
- [ ] Step 4 — Paddy yield data

## Project structure
```
data/raw/        downloaded files, never edited (not in Git)
data/processed/  cleaned outputs
notebooks/       exploration and plots
src/             reusable code
```

## Setup
```bash
conda create -n minicdi -c conda-forge python=3.12 xarray netcdf4 pandas geopandas matplotlib ipykernel
conda activate minicdi
python -m pip install "imdlib>=0.4"
```

## Data notes: IMD gridded rainfall
- **Source:** IMD Pune, 0.25° daily gridded rainfall. Paper: Pai et al. (2014), *Mausam* 65(1), doi:10.54302/mausam.v65i1.851.
- **Grid:** 129 lat × 135 lon cells (6.5–38.5°N, 66.5–100°E) = 17,415 cells; 4,964 are Indian land.
- **Cell size at Palakkad (~10.8°N):** ≈ 27.8 km N–S × 27.3 km E–W (0.25° × 111 km, × cos 10.8° for E–W).
- **How values are made:** rain-gauge readings (6,955 stations) interpolated with Shepard inverse-distance weighting, using 1–4 nearest gauges within 1.5° (~165 km).
- **File format (.grd):** raw little-endian float32, ordered (day, lat, lon), no header. Size = days × 17,415 × 4 bytes (2018: 25,425,900). Row 0 is the southern edge (6.5°N).
- **Units / missing:** mm/day; −999 = outside Indian land (ocean, neighbouring countries) — 71.5% of values. imdlib converts −999 to NaN.
- **Day definition:** ⚠️ To verify — not stated in Pai et al. (2014). Believed to be the 24 h ending 08:30 IST (03:00 UTC).

### Limitations that matter for insurance
- A cell value is a smoothed ~28 km area value, not rain at a farm → source of basis risk.
- A cell with no gauge inside borrows values from gauges up to ~165 km away.
- Gauge numbers change over time (~1,450 in 1901 to ~3,950 in 1991–94, fewer in the paper's last years), so trends can be partly artefacts.
