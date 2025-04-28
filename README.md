# README: Enquiries and Transactions Analysis

## Overview

This project contains SQL scripts for analyzing and processing data related to **enquiries** and **transactions** within a database. It uses a **ClickHouse** database with two tables: `enquiries` and `txns`. The goal is to:

1. Integrate the data from both tables, linking **enquiries** and **transactions** based on `user_id` and `date`.
2. Identify **first transactions** for each enquiry within a **30-day window**.
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

### 1. Aggregate Count of Enquiries and Transactions (`cte` query)

This query aggregates and counts the number of **enquiries** and **transactions** for each `user_id` and `date`. It uses a **Common Table Expression (CTE)** to combine both the `enquiries` and `txns` data and then groups them by `user_id` and `date`.

#### Query Steps:
- **Counts the number of enquiries** for each `user_id` and `date`.
- **Counts the number of transactions** for each `user_id` and `date`.
- **Combines** the counts using a `UNION ALL` approach.
- **Groups** the results by `user_id` and `date` and then **orders** them by `user_id` and `date`.

### 2. Identifying First Transaction for Each Enquiry (`row_number` query)

This query identifies the **first transaction** linked to each enquiry within a **30-day window**. It ensures that only the **first transaction** for each enquiry is counted, based on the transaction date and the order of transaction IDs.

#### Query Steps:
- Joins the `enquiries` and `txns` tables on the `user_id`.
- Filters for transactions that occurred within **30 days** of an enquiry.
- Uses `ROW_NUMBER()` to assign a rank to each transaction for an enquiry, ordering by the transaction date in descending order.
- Groups the results by `enquiry_id` and `e_date` and only selects those where `rn = 1` (first transaction).
- Aggregates the result into an array of transaction IDs (`txn_id_array`) associated with each enquiry.

## Results

### 1. Aggregate Enquiries and Transactions Count

The first query provides a table with the following columns:

| Column    | Description                        |
|-----------|------------------------------------|
| `user_id` | Unique identifier for the user    |
| `date`    | The date of enquiry or transaction |
| `c_eq`    | The count of enquiries for the user and date |
| `c_tx`    | The count of transactions for the user and date |

### 2. First Transaction per Enquiry

The second query provides a table with the following columns:

| Column       | Description                                            |
|--------------|--------------------------------------------------------|
| `enquiry_id` | Unique identifier for the enquiry                      |
| `e_date`     | Date of the enquiry                                    |
| `user_id`    | Unique identifier for the user                         |
| `txn_id_array` | Array of transaction IDs linked to the enquiry        |

## Setup Instructions

1. **Install ClickHouse** if you don't already have it. Follow the instructions on the [ClickHouse website](https://clickhouse.tech/docs/en/getting-started/) for installation.

2. **Create Tables**: Execute the `CREATE TABLE` statements in your ClickHouse database to set up the `enquiries` and `txns` tables.

3. **Insert Data**: Run the `INSERT INTO` queries to populate the tables with sample data.

4. **Execute Queries**: Run the provided queries to analyze the data and generate the desired results.

## Conclusion

This script provides a comprehensive solution for analyzing **enquiries** and **transactions** data. It helps identify **first transactions** linked to enquiries within a 30-day window and provides aggregate counts of both enquiries and transactions, making it useful for customer behavior analysis and revenue impact forecasting.
