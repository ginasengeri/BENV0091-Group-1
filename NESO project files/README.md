# How much of NESO's day-ahead demand forecast error comes from its embedded solar forecast error?

Group project idea (UCL MSc Energy Systems and Data Analytics, BENV0091). We measure how much of the error in NESO's day-ahead national demand forecast is explained by the error in its day-ahead embedded solar forecast (April 2021 to October 2026). We would then look at when the effect is largest, whether over- and under-forecasting differ, and how it has changed over time. 

Later steps add machine learning.

## Folder contents

| File | What it does |
|---|---|
| `fetch_neso_pvlive.py` | Downloads NESO historic demand (2021-2026), the day-ahead demand forecast performance file and PV_Live national solar into `raw_data/` |
| `fetch_neso_forecasts.py` | Downloads NESO embedded solar and wind forecast archives into `raw_forecasts/` and checks that every forecast issue is kept. Use `--years 2026 --skip-demand` to fetch only 2026 |
| `extract_day_ahead.py` | Keeps the latest forecast issued before a D-1 cut-off (08:00, 09:00, 10:00 UTC) for each half-hour. Writes `day_ahead_embedded_forecasts.csv` |
| Notebook | Regression, plots and project proposal |

## Run order

```
pip install requests pandas pytz pvlive-api statsmodels
python fetch_neso_pvlive.py
python fetch_neso_forecasts.py
python extract_day_ahead.py
```

## Key definitions

- **`end_utc`** is the join key. It is the UTC time at the end of each half-hour, built from the UK settlement date and period. It is unique even on 46- and 50-period clock-change days.
- **Errors are forecast minus outturn**, so positive means over-forecast.
  - `demand_err = demand_fc - ND`
  - `solar_err_HH = solar_fc_HH - EMBEDDED_SOLAR_GENERATION`
  - `wind_err_HH = wind_fc_HH - EMBEDDED_WIND_GENERATION`
- **`HH` (08, 09, 10)** is the D-1 forecast cut-off hour in UTC. NESO publishes its day-ahead demand forecast at 09:00-10:00 UK time, so we test all three.
- **`solar_err_pvlive_09`** compares the forecast with PV_Live outturn. It is a robustness check only, because NESO's solar estimate is PV_Live rescaled to NESO's capacity, so the two are not independent.

## Data sources (NESO Data Portal, https://www.neso.energy/data-portal)

- Historic demand data (`historic-demand-data`)
- Day-ahead half-hourly demand forecast performance (`day-ahead-half-hourly-demand-forecast-performance`)
- Embedded wind and solar forecasts archive (`embedded-wind-and-solar-forecasts`)
- 1-day-ahead demand forecast (`1-day-ahead-demand-forecast`)
- PV_Live (Sheffield Solar), national half-hourly solar outturn

## Notes

- Scripts and raw data files are too large and are not committed
