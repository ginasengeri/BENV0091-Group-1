# Wind curtailment dataset (Elexon)

Builds one table: every wind farm bid-offer acceptance (June 2022 – June 2025) with its curtailed volume and the Elexon features, saved as `elexon_data.csv`.

**General idea (for every Elexon pull):**
- determine base url and endpoint for data you want
- determine required parameters and other parameters you want to filter by
- then pull the data, `response.json()` returns a list of dicts (or a dict with a `data` key)
- save as dataframe, each dict is a row, the keys are columns

**Documentation:** https://developer.data.elexon.co.uk, https://bmrs.elexon.co.uk/api-documentation

| Dataset | Endpoint | Use | Request limit found |
|---|---|---|---|
| Bid-offer acceptances (BOALF) | `/datasets/BOALF/stream` | Accepted levels, SO flag | none (one request) |
| BM units | `/reference/bmunits/all` | Wind farm IDs (`fuelType == 'WIND'`) | none |
| Physical notifications (PN) | `/datasets/PN/stream` | Planned output | 3 years returns empty, so yearly |
| Metered generation (B1610) | `/datasets/B1610/stream` | Actual output per unit | 1 year fails, so monthly |
| Wind generation forecast (WINDFOR) | `/datasets/WINDFOR/stream` | Day-ahead wind forecast (GB total) | none (one request) |
| TSDF / NDF | `/datasets/TSDF/stream`, `/datasets/NDF/stream` | Day-ahead demand forecasts | 3 years returns empty, so quarterly |
| INDO / ITSDO | `/datasets/INDO`, `/datasets/ITSDO` | Actual demand (lagged) | 28 days |
| Interconnectors | `/generation/outturn/interconnectors` | Interconnector flows (lagged) | 7 days |
