# NYC Yellow Taxi Trip Records 2024 - BigQuery Homework Queries

This README contains queries and answers to key questions analyzed using Google BigQuery on NYC Yellow Taxi trip records (January–June 2024).

## Question 1: Count of Records for 2024 Yellow Taxi Data
**Query:**
```sql
SELECT COUNT(*) AS total_trips
FROM `nyc-taxi-project-450808.nyc_taxi.yellow_taxi_external`;
```
---

## Question 2: Distinct Number of PULocationIDs for External and Materialized Tables
**Query:**
```sql
SELECT COUNT(DISTINCT PULocationID) AS distinct_pu_locations
FROM `nyc-taxi-project-450808.nyc_taxi.yellow_taxi_external`;
```
---

## Question 3: Count of Records with Fare Amount Equal to 0
**Query:**
```sql
SELECT COUNT(*) AS zero_fare_trips
FROM `nyc-taxi-project-450808.nyc_taxi.yellow_taxi`
WHERE fare_amount = 0;
```
---

## Question 4: Optimized Table Strategy in BigQuery
**Scenario:** Optimize table for queries filtering by `tpep_dropoff_datetime` and ordering by `VendorID`.

**Best Practice:** Partition by `tpep_dropoff_datetime` and cluster by `VendorID`.

**Query to Create Optimized Table:**
```sql
CREATE TABLE `nyc-taxi-project-450808.nyc_taxi.optimized_yellow_taxi`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT *
FROM `nyc-taxi-project-450808.nyc_taxi.yellow_taxi_external`;
```
---


