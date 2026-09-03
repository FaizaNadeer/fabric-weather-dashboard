# DAX Measures

## Core aggregations

## Rolling / short-term volatility

```dax
Temp Rolling 7D Avg =
CALCULATE(
    AVERAGE(fact_weather[TempMax]),
    DATESINPERIOD(dim_date[FullDate], MAX(dim_date[FullDate]), -7, DAY)
)
```
Trailing 7-day average max temperature, anchored to whatever date is in the current filter context.

```dax
Temp Anomaly (Rolling) =
AVERAGE(fact_weather[TempMax]) - [Temp Rolling 7D Avg]
```
How far a given day's temp deviates from its own trailing week — positive means hotter than the recent trend, negative means colder. Finding: these anomalies cluster heavily in winter months.

## Seasonal / historical deviation

```dax
Temp Historical Avg (Month) =
CALCULATE(
    AVERAGE(fact_weather[TempMax]),
    ALL(dim_date[Year])
)
```
Average max temp for a given month, blended across all years (2021-2023), ignoring which specific year is in context — the "typical" baseline for that month.

```dax
Temp Anomaly (Seasonal) =
AVERAGE(fact_weather[TempMax]) - [Temp Historical Avg (Month)]
```
How far a given month/year deviates from that month's typical baseline.

Finding: September 2023 shows the largest positive anomaly; February 2021 shows the largest negative anomaly, consistent with Moscow's documented cold snap that month.
