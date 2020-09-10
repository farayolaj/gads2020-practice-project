
# Examining Billing data with BigQuery

## Task 1: Use BigQuery to import data
### Sign in to BigQuery and create a dataset
You can export billing data directly to BigQuery as outlined here. However, for the purposes of this lab, a sample CSV billing file has been prepared for you. It is located in a Cloud Storage bucket where it is accessible to your student account. You will import this billing information into a BigQuery table and examine it.

1. Specify the following:

Property | Value
-------- | -----
Dataset ID: |	imported_billing_data
Data location: |	US
Default table expiration > Number of days after table creation:	| In 1 day (86400 seconds)

```bash
bq mk --location=US --dataset --default_table_expiration=86400 imported_billing_data
```
2. Run the following command to confirm dataset creation

```bash
bq ls
```

### Create a table and import

Property |	Value
-------- | -------
Create table from:	| Google Cloud Storage
Select file from GCS bucket	| gs://cloud-training/archinfra/export-billing-example.csv
File format |	CSV

1. Use BigQuery to import data into table named "sampleinfotable"
```bash
 bq --location=US load --source_format=CSV --autodetect \
 imported_billing_data.sampleinfotable gs://cloud-training/archinfra/export-billing-example.csv
```
> - `--autodetect` tells the command to detect the schema automatically
  - Notice how the name of the table contains the dataset it belongs to

## Task 2: Examine the table
1. Run the following command. This displays the schema that BigQuery automatically created based on the data it found in the imported CSV file. Notice that there are strings, integers, timestamps, and floating values.
```bash
bq show imported_billing_data.sampleinfotable
```

As you can see in Total Rows, this is a relatively small table with 44 rows.

2. Locate the row that has the Description: Network Internet Ingress from EMEA to Americas.
```bash
bq query --use_legacy_sql=false \
'SELECT Measurement1_Total_Consumption, Measurement1_Units, Cost, Description FROM imported_billing_data.sampleinfotable WHERE Description="Network Internet Ingress from EMEA to Americas"'
```
What was the total consumption and units consumed?

- 9,738 bytes
- 9,738,199 bytes
- 9 bytes
- 9,738,199,000 bytes

3. Look at the Cost column.
The cost was 0.0, so with an ingress of 9.7 Mbytes, traffic from EMEA to the Americas had no charge.

Locate the row that has the Description: Network Internet Egress from Americas to China.
```bash
bq query --use_legacy_sql=false \
'SELECT Measurement1_Total_Consumption, Measurement1_Units, Description FROM imported_billing_data.sampleinfotable WHERE   Description="Network Internet Egress from Americas to China"'
```

Can you interpret the information?

- 5,542 bytes exited the Americas and was transferred to China at a charge of 1e-06.
- 5,542,000 bytes exited the Americas and was transferred to China at a charge of 1e-06.
- 5,542 bytes exited China and was transferred to the Americas at a charge of 1e-06.

## Task 3: Compose a simple query
When you reference a table in a query, both the dataset ID and table ID must be specified; the project ID is optional.

> If the project ID is not specified, BigQuery will default to the current project.

Now construct a simple query based on the Cost field.

1. Paste the following in your terminal:
```bash
bq query --use_legacy_sql=false \
'SELECT * FROM imported_billing_data.sampleinfotable WHERE Cost > 0'
```

2. How many rows had cost greater than 0?

  - 104 rows
  - 10 rows
  - 20 rows
  - 44 rows

3. How many rows involved non-zero charges?
The table shows 20 rows and they all have non-zero charges.

## Task 4: Analyze a large billing dataset with SQL
In the next activity, you use BigQuery to analyze a sample dataset with 22,537 lines of billing data.

> The cloud-training-prod-bucket.arch_infra.billing_data dataset used in this task is shared with the public.

1. Paste the following in the terminal:
```bash
bq query --use_legacy_sql=false \
'SELECT COUNT(*) as number_of_billing_data FROM `cloud-training-prod-bucket.arch_infra.billing_data`'
```
Verify that the result is 22,537 lines of billing data.

2. To find the latest 100 records where there were charges (cost > 0), for New Query, paste the following in terminal:
```bash
bq query --use_legacy_sql=false \
'SELECT product, resource_type, start_time, end_time, cost, project_id, project_name, project_labels_key, currency, currency_conversion_rate, usage_amount, usage_unit FROM `cloud-training-prod-bucket.arch_infra.billing_data` WHERE Cost > 0 ORDER BY end_time DESC LIMIT 100'
```

3. To find all charges that were more than 3 dollars, paste the following in terminal:
```bash
bq query --use_legacy_sql=false \
'SELECT product, resource_type, start_time, end_time, cost, project_id, project_name,  project_labels_key, currency, currency_conversion_rate, usage_amount, usage_unit FROM `cloud-training-prod-bucket.arch_infra.billing_data` WHERE cost > 3'
```

4. To find the product with the most records in the billing data, for New Query, paste the following in terminal:
```bash
bq query --use_legacy_sql=false \
'SELECT product, COUNT(*) AS billing_records FROM `cloud-training-prod-bucket.arch_infra.billing_data` GROUP BY product ORDER BY billing_records DESC'
```

Which product had the most billing records?

- Cloud SQL has 10,271 records
- Cloud Pub/Sub has 10,271 records
- Stackdriver has 11,271 records

5. To find the most frequently used product costing more than 1 dollar, paste the following in terminal:
```bash
bq query --use_legacy_sql=false \
'SELECT product, COUNT(*) AS billing_records FROM `cloud-training-prod-bucket.arch_infra.billing_data` WHERE cost > 1 GROUP BY product ORDER BY billing_records DESC'
```

Which product had the most billing records of over $1

- Compute Engine has 17 charges costing more than 1 dollar.
- Kubernetes Engine has 7 charges costing more than 1 dollar.
- Cloud SQL has 15 charges costing more than 1 dollar.

6. To find the most commonly charged unit of measure, for Compose New Query, paste the following in Query Editor:
```bash
bq query --use_legacy_sql=false \
'SELECT usage_unit, COUNT(*) AS billing_records FROM `cloud-training-prod-bucket.arch_infra.billing_data` WHERE cost > 0 GROUP BY usage_unit ORDER BY billing_records DESC'
```

What was the most commonly charged unit of measure?

- Requests were the most commonly charged unit of measure with 6,539 requests.
- Requests were the most commonly charged unit of measure with 504 requests.
- Byte-seconds were the most commonly charged unit of measure with 2,937 requests.

7. To find the product with the highest aggregate cost, for New Query, paste the following in Query Editor:
```bash
bq query --use_legacy_sql=false \
'SELECT product, ROUND(SUM(cost),2) AS total_cost FROM `cloud-training-prod-bucket.arch_infra.billing_data` GROUP BY product ORDER BY total_cost DESC'
```

Which product has the highest total cost?

- Compute Engine has an aggregate cost of $112.02.
- Cloud SQL has an aggregate cost of $47.37
- BigQuery has an aggregate cost of $114.02

## Task 5: Review
In this lab, you imported billing data into BigQuery that had been generated as a CSV file. You ran a simple query on the file. Then you accessed a shared dataset containing more than 22,000 records of billing information. You ran a variety of queries on that data to explore how you can use BigQuery to ask and answer questions by running queries.