# Architecture

## Pipeline
1. **Data Pipeline** (`ingest_weather_moscow`) calls the Open-Meteo Archive API
   (no auth required) for Moscow, Russia — daily max/min temperature,
   precipitation, and max wind speed, 2021-01-01 to 2023-12-31.
2. Initial attempt: land data directly into a Lakehouse **Table** via
   Copy Data's schema mapping. This failed — the API returns nested JSON
   with parallel arrays (`daily.time`, `daily.temperature_2m_max`, etc.),
   which Fabric's schema-inference repeatedly timed out trying to flatten
   automatically.
3. **Revised approach**: land the raw JSON response as-is into Lakehouse
   **Files**, then parse and flatten it explicitly in a Spark notebook
   using `arrays_zip` + `explode` to convert the parallel arrays into
   proper rows.
4. Notebook casts `Date` from string to a real `date` type, then writes
   `fact_weather` (1,095 rows, one per day) as a Delta table.
5. `dim_date` is generated separately (not derived from `fact_weather`)
   to guarantee a continuous, gap-free calendar for time intelligence —
   includes a custom `Season` column (meteorological seasons, not
   calendar quarters) since Winter spans Dec-Feb across year boundaries.

## Model
- Star schema: `fact_weather` (many) → `dim_date` (one), single
  cross-filter direction, `dim_date` marked as the official date table.
- Direct Lake semantic model — no import/refresh schedule, queries the
  Delta tables live.

## Key DAX measures
- `Temp Rolling 7D Avg` / `Temp Anomaly (Rolling)` — short-term volatility,
  using `DATESINPERIOD`.
- `Temp Historical Avg (Month)` / `Temp Anomaly (Seasonal)` — deviation
  from each month's 2021-2023 baseline, using `CALCULATE` + `ALL(dim_date[Year])`.

## Findings
- Short-term temperature anomalies cluster heavily in winter months,
  consistent with winter having larger day-to-day volatility than summer.
- February 2021 shows the largest negative seasonal anomaly — consistent
  with Moscow's documented severe cold snap that month.
- September 2023 shows the largest positive seasonal anomaly.
