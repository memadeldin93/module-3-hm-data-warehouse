
# DE Zoomcamp 2026 — Module 3 (Data Warehouse) Homework (BigQuery)

This repo contains my answers for the **Data Engineering Zoomcamp 2026 – Module 3: Data Warehouse** homework. ([courses.datatalks.club](https://courses.datatalks.club/de-zoomcamp-2026/homework/hw3?utm_source=chatgpt.com))

---

## Environment

- **Project**: `norse-breaker-452400-q4`
- **Dataset**: `zoomcamp_m3`
- **Source files (GCS / Parquet)**: `gs://mahmoud_dezoomcamp_hw3_2026/*.parquet`

BigQuery reference docs used:
- External tables over Cloud Storage ([docs.cloud.google.com](https://docs.cloud.google.com/bigquery/docs/external-data-cloud-storage?utm_source=chatgpt.com))
- Partitioned tables ([docs.cloud.google.com](https://docs.cloud.google.com/bigquery/docs/partitioned-tables?utm_source=chatgpt.com))
- Clustered tables ([docs.cloud.google.com](https://docs.cloud.google.com/bigquery/docs/clustered-tables?utm_source=chatgpt.com))
- Cached query results (explains “0 B processed”) ([docs.cloud.google.com](https://docs.cloud.google.com/bigquery/docs/cached-results?utm_source=chatgpt.com))

---

## Question 1 — Create an external table over Parquet files in GCS

```sql
CREATE OR REPLACE EXTERNAL TABLE `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024_ext`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://mahmoud_dezoomcamp_hw3_2026/*.parquet']
);
```
---

## Question 2 — Materialize the external table + bytes read comparison

### 2.1 Create a native BigQuery table from the external table

```sql
CREATE OR REPLACE TABLE `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024` AS
SELECT * FROM `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024_ext`;
```

### 2.2 Query + estimated bytes processed

```sql
SELECT COUNT(DISTINCT PULocationID)
FROM `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024`;   -- 155.12 MB

SELECT COUNT(DISTINCT PULocationID)
FROM `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024_ext`; -- 0 B
```

---

## Question 4 — Count trips where `fare_amount = 0`

```sql
SELECT COUNT(1)
FROM `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024`
WHERE fare_amount = 0;
```

**Result:** `8,333`

---

## Question 6 — Create a partitioned + clustered table and compare bytes read

### 6.1 Create partitioned & clustered table

```sql
CREATE OR REPLACE TABLE `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024_p_c`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID
AS
SELECT * FROM `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024_ext`;
```

### 6.2 Compare bytes processed for the same filter window

```sql
SELECT DISTINCT VendorID
FROM `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15'; -- 310.24 MB

SELECT DISTINCT VendorID
FROM `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024_p_c`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15'; -- 26.84 MB
```

---

## Question 9 — Bytes processed for a full table count

```sql
SELECT COUNT(*)
FROM `norse-breaker-452400-q4.zoomcamp_m3.yellow_tripdata_2024`;   -- 0 B
```
