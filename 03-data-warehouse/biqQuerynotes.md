Notes from Big Query
# Use query function in BQ to import data from parquet files in buckets:
    CREATE OR REPLACE EXTERNAL TABLE `kestra-sandbox-449103.week3_homework.external_yellow-trip-2024-h1v2`
        OPTIONS (
        format = 'Parquet',
        uris = ['gs://kestra-sandbox-449103-bucket-week3-homeworkv2/yellow_tripdata_2024-*.parquet']
        );
    This merges all 6 of the parquet files in to 1 table in BQ called 'external'.
    THis external table cannot be direclty inspected so next query creates an internal table

# create internal table that can be inspected
    CREATE OR REPLACE TABLE `kestra-sandbox-449103.week3_homework.yellow_2024_main_table`
        AS SELECT * FROM `week3_homework.external_yellow-trip-2024-h1v2`;

 