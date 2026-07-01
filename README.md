# AtmoSync — Micro-Climate Arbitrage Analytics

AtmoSync is a real-time micro-climate analytics pipeline that monitors IoT
sensors inside shipping containers, models commodity spoilage as it
happens, and surfaces "reroute to a better market" arbitrage alerts before
goods degrade below quality thresholds.

## Pipeline

```
simulator/  -->  Kafka  -->  streaming/consumer.py  -->  SQLite (atmosync.db)
(mock IoT)        topic        (validate + persist)        |
                                                             v
                                              dbt_atmosync/ (transform)
                                              - spoilage_degradation
                                              - spoilage_arbitrage
                                                             |
                                                             v
                                                  Superset dashboard
                                              (reroute alerts, live feed)
```

Currently runs end-to-end on local SQLite + Kafka via Docker. Snowflake is
the intended production warehouse — see `snowflake/MIGRATION.md` for the
swap-in path once you have credentials.

## Technologies
- Python, Apache Kafka (streaming ingestion)
- SQLite today / Snowflake-ready (warehouse)
- dbt (spoilage degradation curve + arbitrage margin modeling, in SQL)
- Apache Superset (BI dashboard)

## Folder Structure
- `simulator/` — Kafka producer generating mock sensor data
- `streaming/` — Kafka consumer, validates and writes to the database
- `database/` — schema creation/reset scripts, market & pricing seed data
- `scripts/` — sensor data validation logic
- `config/` — Kafka/DB settings, logger
- `dbt_atmosync/` — dbt project: staging + marts (spoilage & arbitrage models)
- `snowflake/` — DDL + migration guide for moving off SQLite
- `superset/` — dashboard setup guide + ready-to-paste SQL Lab queries
- `data/`, `docs/`, `images/` — supporting assets

## Running it locally

1. **Start infra**
   ```bash
   docker compose up -d zookeeper kafka superset
   ```

2. **Install Python deps**
   ```bash
   pip install -r requirements.txt
   ```

3. **Create the database + seed reference data**
   ```bash
   python -m database.create_database
   python -m database.create_market_data
   ```

4. **Start the consumer** (in its own terminal, keeps running)
   ```bash
   python -m streaming.consumer
   ```

5. **Start the simulator** (in another terminal, generates mock sensor events)
   ```bash
   python -m simulator.sensor_simulator
   ```

6. **Run dbt transforms** (after some data has streamed in)
   ```bash
   cd dbt_atmosync
   dbt run --profiles-dir .
   ```

7. **Build the dashboard** — see `superset/README.md` for connecting
   Superset to `atmosync.db` and the exact charts to build.

## Status
- ✅ Streaming ingestion (Kafka producer + consumer) working
- ✅ Local warehouse (SQLite) with sensor, market, and pricing data
- ✅ dbt models: spoilage degradation curve + arbitrage margin calculation
- ✅ Superset setup guide + ready-to-use queries
- ⏳ Snowflake migration (guide ready in `snowflake/MIGRATION.md`, pending account access)
- ⏳ Live Superset dashboard build-out (manual step, guide in `superset/README.md`)
