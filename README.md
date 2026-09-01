# Fabric Weather Dashboard

A weather trends analytics project built on Microsoft Fabric, using a scheduled Data Pipeline to ingest historical weather data from the Open-Meteo API, transform it into a star schema via a Spark notebook, and surface it through a Power BI report with time-series and anomaly detection measures.

## Architecture

(diagram + explanation coming soon)

## Tech stack
- Microsoft Fabric (Data Pipeline, Lakehouse, Notebook, Power BI semantic model)
- Open-Meteo API (no auth required)
- PySpark
- DAX

## Status
🚧 In progress

## Contents
- `notebooks/` — Spark notebook exports for data transformation
- `dax/` — documented DAX measures
- `screenshots/` — report screenshots
- `docs/` — architecture notes and design decisions
