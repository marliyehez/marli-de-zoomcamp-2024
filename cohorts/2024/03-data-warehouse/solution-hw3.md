# Homework Week 3

### SETUP:

```sql
-- Create external table referring to GCS path
CREATE OR REPLACE EXTERNAL TABLE nytaxi.external_green_tripdata
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://nytaxi-bob/green-taxi-2022/green_tripdata_2022-*.parquet']
);

-- Create a non-partitioned table from external table
CREATE OR REPLACE TABLE nytaxi.green_tripdata_non_partitioned AS
SELECT * FROM nytaxi.external_green_tripdata;
```

### Question 1:

What is count of records for the 2022 Green Taxi Data??

> 840,402

```sql
SELECT COUNT(1)
FROM `golden-union-392713.nytaxi.green_tripdata_non_partitioned`;
```

### Question 2:

Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?

> 0 MB for the External Table and 6.41MB for the Materialized Table

```sql
-- External table
SELECT COUNT(DISTINCT PULocationID)
FROM `golden-union-392713.nytaxi.external_green_tripdata`;

-- Materialized table
SELECT COUNT(DISTINCT PULocationID)
FROM `golden-union-392713.nytaxi.green_tripdata_non_partitioned`;
```

### Question 3:

What is count How many records have a fare_amount of 0?

> 1,622

```sql
SELECT COUNT(1)
FROM `golden-union-392713.nytaxi.green_tripdata_non_partitioned`
WHERE fare_amount = 0;
```

### Question 4:

What is the best strategy to make an optimized table in Big Query if your query will always order the results by PUlocationID and filter based on lpep_pickup_datetime? (Create a new table with this strategy)

> Partition by lpep_pickup_datetime Cluster on PUlocationID

```sql
CREATE OR REPLACE TABLE nytaxi.green_tripdata_non_partitioned
PARTITION BY DATE(lpep_pickup_datetime)
CLUSTER BY PULocationID
AS
SELECT *
FROM `golden-union-392713.nytaxi.external_green_tripdata`;
```

### Question 5:

Write a query to retrieve the distinct PULocationID between lpep_pickup_datetime 06/01/2022 and 06/30/2022 (inclusive)

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 4 and note the estimated bytes processed. What are these values?

> 12.82 MB for non-partitioned table and 1.12 MB for the partitioned table

```sql
-- Query for the non-partitioned table
SELECT DISTINCT PULocationID
FROM `golden-union-392713.nytaxi.green_tripdata_non_partitioned`
WHERE DATE(lpep_pickup_datetime) BETWEEN '2022-06-01' AND'2022-06-30';

-- Query for the partitioned table
SELECT DISTINCT PULocationID
FROM `golden-union-392713.nytaxi.green_tripdata_partitioned`
WHERE DATE(lpep_pickup_datetime) BETWEEN '2022-06-01' AND'2022-06-30';
```

### Question 6:

Where is the data stored in the External Table you created?

> GCP Bucket

### Question 7:

It is best practice in Big Query to always cluster your data:

> False

### Question 8 (Bonus: Not worth points):

Write a `SELECT count(\*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

> 0 Bytes processed. i think it because BigQuery may still be able to estimate the number of rows by reading **metadata** such as table statistics and schema information, without actually reading the entire table. Or maybe BigQuery might **cache metadata** or other query planning information, resulting a 0 Bytes processed metric even if caching is disabled for the query results.
