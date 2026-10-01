# GoodCabs Transportation Data Engineering Project

## Project Description

This project builds a scalable transportation analytics data platform for GoodCabs using Databricks-Lakeflow Spark Declarative Pipelines. It ingests city and trip data from CSV files, processes the data through a medallion architecture, and publishes curated Gold views for reporting and city-level analysis.

The pipeline supports both historical full loads and ongoing incremental trip ingestion. Raw data is preserved in the Bronze layer, standardized and validated in the Silver layer, and combined into an analytics-ready trip model in the Gold layer.

## Architecture

```text
CSV files in cloud storage
        |
        v
Bronze: raw city and trip ingestion
        |
        v
Silver: cleansing, validation, CDC upserts, calendar dimension
        |
        v
Gold: enriched fact trips and city-specific analytical views
```

The project follows a three-layer medallion architecture:

- **Bronze**: Captures raw source data with ingestion metadata and schema-rescue handling.
- **Silver**: Applies column standardization, type conversions, data-quality expectations, and change-data processing.
- **Gold**: Joins trips with city and calendar dimensions to create a reporting-ready fact view, with additional views filtered by city.

## Data Sources

- `city.csv`: City reference data containing city identifiers and names.
- Trip CSV exports: Daily transportation trip records containing trip date, city, passenger category, distance, fares, and driver/passenger ratings.
- Full-load and incremental-load file sets are organized under the project data directory.
- The pipeline code is configured to read from Amazon S3 locations under `s3://goodcabs/data-store/`.

## Pipeline Details

### Bronze Layer

- Reads city data as a materialized view using Spark CSV ingestion.
- Ingests trips incrementally with Databricks Auto Loader through Structured Streaming.
- Enables schema inference and schema evolution/rescue behavior for incoming files.
- Renames `distance_travelled(km)` to `distance_travelled_km` for downstream compatibility.
- Adds source file path and ingestion timestamp metadata.

### Silver Layer

- Standardizes trip column names and business data types.
- Converts trip dates to `date` values and normalizes passenger categories.
- Applies data-quality expectations for business dates and passenger/driver ratings.
- Uses Delta change data feed and Auto CDC flows to upsert trip records by trip ID.
- Uses Slowly Changing Dimension Type 1 behavior for the trip target.
- Creates a cleaned city dimension.
- Generates a configurable calendar dimension with date, month, quarter, weekday, weekend, and Indian national holiday attributes.

### Gold Layer

- Creates `transportation.gold.fact_trips` by joining Silver trips with city and calendar dimensions.
- Exposes business-friendly fields such as city name, passenger category, distance, sales amount, ratings, and calendar attributes.
- Provides city-specific analytical views for Chandigarh, Coimbatore, Indore, Jaipur, Kochi, Lucknow, Mysore, Surat, Vadodara, and Visakhapatnam.

## Technologies Used

- **Databricks**: Cloud data engineering and pipeline execution platform.
- **Python**: Pipeline implementation language.
- **PySpark**: Distributed data processing, DataFrame transformations, streaming reads, and Spark SQL execution.
- **Databricks Lakeflow Declarative Pipelines**: Pipeline definitions using `pyspark.pipelines`, including tables, views, materialized views, expectations, and CDC flows.
- **Apache Spark Structured Streaming**: Incremental processing of trip files.
- **Databricks Auto Loader**: Efficient file discovery and incremental cloud-object-storage ingestion using the `cloudFiles` source.
- **Delta Lake**: Transactional storage and table management, including Change Data Feed and write/compaction optimization.
- **Delta Change Data Feed**: Change tracking for Bronze and Silver tables.
- **Auto CDC**: Key-based change application into the Silver trips table.
- **SCD Type 1**: Latest-value representation for trip updates.
- **Spark SQL / SQL**: Calendar generation and Gold analytical view creation.
- **Amazon S3**: Configured cloud object storage source for city and trip data.
- **CSV**: Input file format for city and trip exports.

## Repository Structure

```text
1. data/
  city/                         City source data
  trips/Full Load/              Historical trip files
  trips/Incremental Load/       Incremental trip files

2. transportation_pipeline/transformations/
  project_setup.ipynb           Project setup notebook
  bronze/                       Raw ingestion pipelines
  silver/                       Cleansing and dimensional pipelines
  gold/                         Analytics views and city-specific views

3. other_files/
  architecture.png              Architecture reference diagram
```

## Result

The final Gold layer provides a consistent, enriched transportation fact dataset that can be consumed by dashboards, SQL analysis, and city-level operational reporting without requiring analysts to work directly with raw ingestion files.
