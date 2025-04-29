# README: Enquiries and Transactions Analysis

## Overview

This project contains SQL scripts for analyzing and processing data related to **enquiries** and **transactions** within a database. It uses a **ClickHouse** database with two tables: `enquiries` and `txns`. The goal is to:

1. Integrate the data from both tables, linking **enquiries** and **transactions** based on `user_id` and `date`.
2. Identify **transactions** for each enquiry within a **30-day window**.
3. Provide aggregate results such as the **count of enquiries** and **transactions** by `user_id` and `date`.

## Tables

### 1. `enquiries` Table

This table contains information about user **enquiries**:

| Column Name  | Data Type | Description                               |
|--------------|-----------|-------------------------------------------|
| `enquiry_id` | String    | Unique identifier for the enquiry         |
| `user_id`    | String    | Unique identifier for the user            |
| `date`       | Date      | Date when the enquiry was made            |

### 2. `txns` Table

This table contains information about **transactions**:

| Column Name  | Data Type | Description                               |
|--------------|-----------|-------------------------------------------|
| `txn_id`     | String    | Unique identifier for the transaction     |
| `user_id`    | String    | Unique identifier for the user            |
| `date`       | Date      | Date when the transaction occurred        |
| `amount`     | UInt16    | The amount involved in the transaction    |

## Data Insertions

### Enquiries Data

The script includes **INSERT statements** for adding entries into the `enquiries` table, representing user enquiry records.

### Transactions Data

The script includes **INSERT statements** for adding entries into the `txns` table, representing user transaction records.

## Queries

### 1. Enquiry and Transaction Counts

This query aggregates and counts the number of **enquiries** and **transactions** for each `user_id` and `date`. It uses a **Common Table Expression (CTE)** to combine both the `enquiries` and `txns` data and then groups them by `user_id` and `date`.

#### Query Steps:
- **Counts the number of enquiries** for each `user_id` and `date`.
- **Counts the number of transactions** for each `user_id` and `date`.
- **Combines** the counts using a `UNION ALL` approach.
- **Groups** the results by `user_id` and `date` and then **orders** them by `user_id` and `date`.

### 2. Enquiry Conversion Mapping 

This task tracks the conversion of user enquiries into transactions. A user expresses interest by submitting an enquiry form, and an enquiry is considered "converted" if the user completes a transaction within 30 days of submitting the form. Each transaction is linked to at most one enquiry — specifically, the first eligible enquiry made within the preceding 30 days. However, a single enquiry can be associated with multiple transactions. The goal is to accurately measure the conversion performance of enquiries over time.

#### Query Steps:
- Joins the `enquiries` and `txns` tables on the `user_id`.
- Filters for transactions that occurred within **30 days** of an enquiry.
- Uses `ROW_NUMBER()` to assign a rank to each `enquiry_id` based on the enquiry date, in cases where multiple enquiries are linked to a single transaction within 
  the last 30 days. Only the most recent (latest) enquiry within that 30-day window is considered.
- Aggregates the result into an array of transaction IDs (`txn_id_array`) associated with each enquiry.

## Results

### 1. Enquiry and Transaction Counts

The first query provides a table with the following columns:

| Column    | Description                        |
|-----------|------------------------------------|
| `user_id` | Unique identifier for the user    |
| `date`    | The date of enquiry or transaction |
| `c_eq`    | The count of enquiries for the user and date |
| `c_tx`    | The count of transactions for the user and date |

### 2. Enquiry Conversion Mapping

The second query provides a table with the following columns:

| Column       | Description                                            |
|--------------|--------------------------------------------------------|
| `enquiry_id` | Unique identifier for the enquiry                      |
| `e_date`     | Date of the enquiry                                    |
| `user_id`    | Unique identifier for the user                         |
| `txn_id_array` | Array of transaction IDs linked to the enquiry        |

## Setup Instructions

1. **Use [ClickHouse](https://fiddle.clickhouse.com/)** website to run the code.

2. **Create Tables**: Execute the `CREATE TABLE` statements in your ClickHouse database to set up the `enquiries` and `txns` tables.

3. **Insert Data**: Run the `INSERT INTO` queries to populate the tables with sample data.

4. **Execute Queries**: Run the provided queries to analyse the data and generate the desired results.

## Conclusion

This script provides a comprehensive solution for analysing **enquiries** and **transactions** data. It helps identify **transactions** linked to enquiries within a 30-day window and provides aggregate counts of both enquiries and transactions, making it useful for customer behaviour analysis and revenue impact forecasting.
