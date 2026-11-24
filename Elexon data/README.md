# Elexon dataset (curtailment, generation, national demand, demand forecasts, interconnectors)

Builds one table: every wind farm bid-offer acceptance (June 2022 – June 2025) with its curtailed volume and the Elexon features, saved as `elexon_data.csv`.

## Data sources

**General idea (for every Elexon pull):**
- determine base url and endpoint for data you want
- determine required parameters and other parameters you want to filter by
- then pull the data, `response.json()` returns a list of dicts (or a dict with a `data` key)
- save as dataframe, each dict is a row, the keys are columns

**Documentation:** https://developer.data.elexon.co.uk, https://bmrs.elexon.co.uk/api-documentation

| Dataset | Endpoint | Use | Request limit found |
|---|---|---|---|
| BM units | `/reference/bmunits/all` | Wind farm IDs (`fuelType == 'WIND'`) | none |
| Bid-offer acceptances (BOALF) | `/datasets/BOALF/stream` | Accepted levels, SO flag | 3 years runs out of memory, so monthly (filtered to wind each month) |
| Physical notifications (PN) | `/datasets/PN/stream` | Planned output | 3 years returns empty, so yearly |
| Metered generation (B1610) | `/datasets/B1610/stream` | Actual output per unit | 1 year fails, so monthly |
| Wind generation forecast (WINDFOR) | `/datasets/WINDFOR/stream` | Day-ahead wind forecast (GB total) | none (one request) |
| TSDF / NDF | `/datasets/TSDF/stream`, `/datasets/NDF/stream` | Day-ahead demand forecasts | 3 years returns empty, so quarterly |
| INDO / ITSDO | `/datasets/INDO`, `/datasets/ITSDO` | Actual demand (lagged) | 28 days |
| Interconnectors | `/generation/outturn/interconnectors` | Interconnector flows (lagged) | 7 days |

## Method

### Step 1: Pull the data

| Data | Pulled by | Time columns it has |
|---|---|---|
| BOA (NESO's instruction) | wind ID (`bmUnit`) | `settlementDate`; `settlementPeriodFrom` / `settlementPeriodTo` (half-hour the instruction starts / ends, 1–48); `timeFrom` / `timeTo` (exact start / end) |
| PN (farm's plan), metered generation | wind ID (`bmUnit`) | `settlementDate`; `settlementPeriod` (half-hour, 1–48) |
| TSDF, NDF, INDO, ITSDO, interconnectors | national | `settlementDate`; `settlementPeriod` (half-hour, 1–48) |
| Wind forecast | national | `startTime` (hour) |

### Step 2: Calculate curtailment

| Action | Match columns |
|---|---|
| Match `pnLevel` to each BOA: the PN of the same wind farm, same day, and the half-hour the BOA starts in | BOA `bmUnit`, `settlementDate`, `settlementPeriodFrom` = PN `bmUnit`, `settlementDate`, `settlementPeriod` |
| Calculate `curtailed_MWh` = (pnLevel − boaLevel) × hours, where PN is higher. hours = `timeTo` − `timeFrom` | each BOA row |
| Add `hour`: `timeFrom` rounded down to the hour, because the wind forecast is hourly | each BOA row |

### Step 3: Add features to each BOA

| Feature | Match columns |
|---|---|
| Metered generation (lagged: its `settlementDate` + 1 day) | `bmUnit`, `settlementDate`, `settlementPeriod` |
| Wind forecast (`startTime` renamed to `hour`) | `hour` |
| TSDF, NDF | `settlementDate`, `settlementPeriod` |
| INDO, ITSDO, interconnectors (lagged: their `settlementDate` + 1 day) | `settlementDate`, `settlementPeriod` |
