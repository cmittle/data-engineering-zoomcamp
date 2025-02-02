# Module 2 Homework for workflow orchestration

## Question 1:  Within the execution for Yellow Taxi data for the year 2020 and month 12: what is the uncompressed file size (i.e. the output file yellow_tripdata_2020-12.csv of the extract task)?
        128.3 MB
        134.5 MB
        364.7 MB
        692.6 MB

   ### Solution:  
       #  In Kestra open flow 06_gcp_taxi and in purge task change disabled to true so the file is kept. run for yellow on year 2020 and month 12. Look at output file and see that it is size 128.3 MB

## Question 2: What is the rendered value of the variable file when the inputs taxi is set to green, year is set to 2020, and month is set to 04 during execution?
        {{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv
        green_tripdata_2020-04.csv
        green_tripdata_04_2020.csv
        green_tripdata_2020.csv
   ### Solution:

## Question 3: How many rows are there for the Yellow Taxi data for all CSV files in the year 2020?
        13,537.299
        24,648,499
        18,324,219
        29,430,127
   ### Solution:

## Question 4: How many rows are there for the Green Taxi data for all CSV files in the year 2020?
        5,327,301
        936,199
        1,734,051
        1,342,034
   ### Solution:

## Question 5: How many rows are there for the Yellow Taxi data for the March 2021 CSV file?
        1,428,092
        706,911
        1,925,152
        2,561,031
   ### Solution:

## Question 6: How would you configure the timezone to New York in a Schedule trigger?
        Add a timezone property set to EST in the Schedule trigger configuration
        Add a timezone property set to America/New_York in the Schedule trigger configuration
        Add a timezone property set to UTC-5 in the Schedule trigger configuration
        Add a location property set to New_York in the Schedule trigger configuration
   ### Solution:


