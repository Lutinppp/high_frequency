# high_frequency

The aim of this project is to extract, transform, and aggregate high-frequency data that provides information on the current state of the energy and economic systems in Europe.

## Data Pipeline

### ENTSOG Weekly Country Panel

**Source:** ENTSOG Transparency Platform (European Network of Transmission System Operators for Gas)

**Location:** `methodo/01_entsog_weekly_panel.ipynb`

Extracts daily gas flow data from the ENTSOG AggregatedData endpoint and aggregates into weekly country-level panels for macro analysis of European gas consumption and supply dynamics.

#### Coverage

- **Countries:** DE (Germany), FR (France), IT (Italy), NL (Netherlands), ES (Spain), UK (United Kingdom), EU (aggregate)
- **Indicators:** Physical Flow, Allocation
- **Period:** 2019-11-01 to present (weekly starting on Monday)
- **Units:** TWh/week (terawatt-hours)
- **Frequency:** Daily extraction from API, weekly aggregation

#### Outputs

**1. Daily Raw Data** (`data/processed/entsog_daily_selected.csv`)
  - Filtered ENTSOG records at daily granularity (kWh/d)
  - Columns: `id`, `countryKey`, `countryLabel`, `indicator`, `periodType`, `periodFrom`, `periodTo`, `unit`, `value`, `flowStatus`, `lastUpdateDateTime`, `directionKey`, `adjacentSystemsLabel`, `operatorLabel`, `bzShort`, `date`
  - ~47,700 rows (104,281 raw API rows after deduplication by latest revision)

**2. Weekly Panel** (`data/processed/entsog_weekly_panel.csv`)
  - Aggregated weekly country-indicator panels with derived metrics
  - Columns:
    - `countryKey` – Country code (DE/FR/IT/NL/ES/UK/EU)
    - `indicator` – Physical Flow or Allocation
    - `week_start` – Monday of the reporting week (YYYY-MM-DD)
    - `twh_week` – Weekly sum in TWh
    - `wow_pct` – Week-over-week % change
    - `yoy_pct` – Year-over-year % change (52-week lag)
  - ~4,700 rows covering all 7 geographies and 2 indicators
  - NaN values appear only where no prior week exists (for WoW) or data is truly missing; contiguous reindexing ensures proper temporal comparisons

#### Pipeline Behavior

- **Initial run:** Full historical backfill from 2019-11-01 onwards (controlled by `FORCE_MAX_HISTORY_PULL = True`)
- **Subsequent runs:** Incremental append with 7-day overlap buffer to capture late revisions
- **Data quality:** Latest revision per ENTSOG record ID is retained; provisional data flagged via `flowStatus`

#### Usage

Open `methodo/01_entsog_weekly_panel.ipynb` and run all cells. Set `FORCE_MAX_HISTORY_PULL = False` in Cell 3 for incremental-only routine updates. 