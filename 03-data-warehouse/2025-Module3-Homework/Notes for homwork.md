# Module 3 Homework for data warehouse
## Important Note:
For this homework we will be using the Yellow Taxi Trip Records for January 2024 - June 2024 NOT the entire year of data Parquet Files from the New York City Taxi Data found here:
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
If you are using orchestration such as Kestra, Mage, Airflow or Prefect etc. do not load the data into Big Query using the orchestrator.
Stop with loading the files into a bucket.


## Load Script: 
You can manually download the parquet files and upload them to your GCS Bucket or you can use the linked script here:
You will simply need to generate a Service Account with GCS Admin Priveleges or be authenticated with the Google SDK and update the bucket name in the script to the name of your bucket
Nothing is fool proof so make sure that all 6 files show in your GCS Bucket before begining.


## NOTE: You will need to use the PARQUET option files when creating an External Table

## BIG QUERY SETUP:
Create an external table using the Yellow Taxi Trip Records.
Create a (regular/materialized) table in BQ using the Yellow Taxi Trip Records (do not partition or cluster this table).

## solution planning
1. use Kestra to orchestrate uploading to bucket as we alrady did this and it should just require some tweaking of bucket names, and file location

## Solution Steps:
1. from terminal cd to 02-workflow-orchestration folder
2. docker compose up to launch kestra orechestrator
3. create new flow 10_week3_homework and start copying in sections from 06_gcp_taxi
4. overhaul these sections
  a. bucket name, file name, file extension, etc... all need to change
  b. I keep getting errors trying to use kestra for this. Primarily that wget failes with exit code 4. Even after a 2 minut download / run time
5. Abort and manually download parquet files and upload to GCP bucket :(
        Right after manually uploading I found a thread on slack that helped me solve this. I now have
        Kestra doing all 6 uploads. I had to add a 'create bucket' task in the kestra flow as well
        I kept running in to region conflicts on BigQuery so i revised this kestra flow to create the bucket in region us-central1
        I revised the key values in the namespace for the location to be us-central1, re-run and tested, and now new table creation works and doesn't thow the location error
6. I rerun all of the uploads month 1-6 for yellow_taxidata_2024 in kestra and restarted bigquery steps
7. Create 'external table' with all 6 months of data from bucket.
8. create 'internal table' from this external table
        The internal table allows for inspection (row count, preview, etc...)



## Question 1: Question 1: What is count of records for the 2024 Yellow Taxi Data?
        65,623
        840,402
        * 20,332,093
        85,431,289
        
   ### Solution:
  After uploading files via kestra to bucket, and using BQ to compile in to external table, then convert to internal table I was finally able to run:
        SELECT COUNT(*) FROM `kestra-sandbox-449103.week3_homework.yellow_2024_main_table`
  and get the answer of 20332093

## Question 2: Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?
        18.82 MB for the External Table and 47.60 MB for the Materialized Table
        * 0 MB for the External Table and 155.12 MB for the Materialized Table
        2.14 GB for the External Table and 0MB for the Materialized Table
        0 MB for the External Table and 0MB for the Materialized Table
   ### Solution:
        Query 1: SELECT DISTINCT PULocationID FROM `kestra-sandbox-449103.week3_homework.external_yellow-trip-2024-h1v2` estimated 0B
        Query 2: SELECT DISTINCT PULocationID FROM `kestra-sandbox-449103.week3_homework.yellow_2024_main_table`  estimated 155.12MB
        


## Question 3: Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?
        BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.
        BigQuery duplicates data across multiple storage partitions, so selecting two columns instead of one requires scanning the table twice, doubling the estimated bytes processed.
        BigQuery automatically caches the first queried column, so adding a second column increases processing time but does not affect the estimated bytes scanned.
        * When selecting multiple columns, BigQuery performs an implicit join operation between them, increasing the estimated bytes processed

   ### Solution:
        Query 1: SELECT PULocationID FROM `kestra-sandbox-449103.week3_homework.yellow_2024_main_table` estimated: 155.12MB processed: 155.12 billed: 156
        Query 2: SELECT PULocationID, DOLocationID FROM `kestra-sandbox-449103.week3_homework.yellow_2024_main_table` estimated 310.24MB processed: 310.24MB billed: 311
        THe answer is double.  I'm not sure i actually know why but 3 4th answer seems correct, so let's go with that :)

## Question 4: How many records have a fare_amount of 0?
        128,210
        546,578
        20,188,016
        * 8,333
   ### Solution:
        Query: SELECT COUNT(*) 
                        FROM `kestra-sandbox-449103.week3_homework.yellow_2024_main_table`
                WHERE fare_amount = 0;
        Returned 8333

## Question 5: What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)
        Partition by tpep_dropoff_datetime and Cluster on VendorID
        Cluster on by tpep_dropoff_datetime and Cluster on VendorID
        * Cluster on tpep_dropoff_datetime Partition by VendorID
        Partition by tpep_dropoff_datetime and Partition by VendorID 
   ### Solution:
        The datetime values are down to 1 second intervals for which there are 9948395 unique values (SELECT COUNT(DISTINCT tpep_dropoff_datetime) FROM `kestra-sandbox-449103.week3_homework.yellow_2024_main_table`;) meaning any partitioning done on these would create nearly 10 million partitions from 20 million records, not a huge gain, so partionining would make more sense on vendor ID's for which there are only 3 (SELECT COUNT(DISTINCT VendorID) FROM `kestra-sandbox-449103.week3_homework.yellow_2024_main_table`;)
        Partioning on vendor id and clustering (sorting) on datetimes makes the most sense of the options listed.



## Question 6: Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive)
Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?
Choose the answer which most closely matches.
        12.47 MB for non-partitioned table and 326.42 MB for the partitioned table
        * 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table
        5.87 MB for non-partitioned table and 0 MB for the partitioned table
        310.31 MB for non-partitioned table and 285.64 MB for the partitioned table 
   ### Solution:
        Query 1: SELECT COUNT(DISTINCT VendorID) FROM `kestra-sandbox-449103.week3_homework.yellow_2024_main_table`
                        WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' and '2024-03-15';  estimated 310.24MB
        Query 2: Currently I'm struggling with creatinga  new table in BQ that is partitioned and clustered. For now I'm going to pick B b/c that's the only one with the correct first answer :)




## Question 7: Where is the data stored in the External Table you created?
        Big Query
        Container Registry
        GCP Bucket
        * Big Table

   ### Solution:
        For today's submission I will select D in case I don't complete this before closing.

## Question 8: It is best practice in Big Query to always cluster your data:
        True
        * False

   ### Solution:
        For todays submission I will select False as it seems intuitively like the right answer and I want to hav esomethign submitted incase i don't get back to thsi tomorrow night.


## BONUS Question 9 (No points): 
No Points: Write a SELECT count(*) query FROM the materialized table you created. How many bytes does it estimate will be read? Why?
   ### Solution:
