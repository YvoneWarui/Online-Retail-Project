# Online Retail Data Cleaning & Validation in SQL

## Overview
This repository contains an end-to-end data cleaning, standardization, and validation pipeline executed on the **Online Retail** dataset using **SQL** (MySQL via `ipython-sql` / `PyMySQL` in Jupyter Notebooks).

The goal of this project is to transform raw, unformatted transaction data (~540k+ records) into a cleansed, validated, and analytics-ready dataset while preserving financial integrity for downstream metrics.

---

## Technical Stack
* **Language:** SQL (MySQL Dialect)
* **Interface:** Jupyter Notebook / IPython
* **Extensions:** `ipython-sql`, `pymysql`
* **Database Target:** `online_retail`

---

## Pipeline & Process Breakdown

### 1. Dataset Exploration (Task 1)
* Inspected schema types, initial row limits, and overall structural counts.
* Evaluated initial statistics: **540,455 rows** across **8 columns**.
* Cataloged numerical features (`Quantity`, `UnitPrice`) versus categorical string features (`InvoiceNo`, `StockCode`, `Description`, `InvoiceDate`, `CustomerID`, `Country`).

---

### 2. Data Cleaning (Task 2)

#### Missing Value Strategy (Single-Pass Tally Counter)
* Implemented conditional aggregations (`SUM(CASE WHEN ... THEN 1 ELSE 0 END)`) across all columns to evaluate null/blank occurrences in a single table scan.
* **`CustomerID` Handling:** Imputed **135,080** missing entries as `'Guest'` instead of deleting them. This preserves core financial revenue figures while isolating anonymous guest behavior from loyal customer cohorts.
* **`Description` Handling:** Deleted **1,454** records where `Description` was missing **AND** `UnitPrice` was `$0.00`. These records represented internal system glitches, broken logs, or failed data packets rather than legitimate consumer behavior.

#### Duplicate Removal (Table Swapping Method)
* Identified **5,268** exact duplicate rows using multi-column `GROUP BY` and `HAVING COUNT(*) > 1` queries.
* Executed the **Table Swapping Method**:
  1. Created `onlineretail_clean` holding only `DISTINCT *` records.
  2. Dropped the original uncleaned table.
  3. Renamed `onlineretail_clean` back to `onlineretail`.
* Verified exact row count alignment post-deduplication (**535,187 active rows**).

---

### 3. Standardization & Formatting

#### Date Conversions
* Converted raw string representations in `InvoiceDate` (`MM/DD/YYYY HH:MM`) into proper MySQL `DATETIME` formats using `STR_TO_DATE()`.
* Formally modified column metadata to native `DATETIME` data types.

#### String Case Uniformity
* Validated case consistency across `Country`, `Description`, and `StockCode` fields using `LOWER()` grouping.
* Standardized `StockCode` string variations to `UPPERCASE` across all rows.

#### Schema & Column Integrity
* Checked column metadata inside `information_schema.columns` to ensure proper precision, length, and lack of accidental trailing whitespace.
* Verified non-numeric `CustomerID` records mapped strictly to defined rules (such as assigned `'Guest'` identities).

---

### 4. Data Validation & Anomaly Detection

#### Outliers & Non-Positive Metrics
* Flagged negative/zero values across `Quantity` (**9,725** records) and `UnitPrice` (**1,062** records).
* Built a dedicated `validated_retail` analytical staging table:
  * Filtered out zero/invalid prices (`UnitPrice > 0`).
  * Engineered a `TransactionType` feature to cleanly separate **Cancellations/Returns** (`InvoiceNo LIKE 'C%'` or `Quantity < 0`) from **Standard Sales**.

---

## How to Run

### Prerequisites
* **MySQL Server** running locally on port `3306`.
* **Python environment** with `jupyter`, `ipython-sql`, and `pymysql` installed:
  ```bash
  pip install jupyter ipython-sql pymysql