# Chicago Food Inspections ETL Pipeline

An end-to-end Data Engineering project built with **PySpark, Databricks, and Delta Lake** using the City of Chicago Food Inspections dataset.

The project processes **316,703 inspection records** through a structured **Bronze → Silver → Gold** pipeline and includes data cleaning, validation, quality flags, analytics tables, and end-to-end reconciliation.

---

## Project Architecture

```text
Chicago Open Data
        |
        v
     Raw CSV
        |
        v
 Bronze Delta Layer
        |
        v
 PySpark Transformations
        |
        v
 Silver Delta Layer
        |
        +----------------------+
        |                      |
        v                      v
 Risk Analysis Gold      Yearly Trends Gold
        |                      |
        +----------+-----------+
                   |
                   v
         Data Reconciliation
```

---

## Technology Stack

- Python
- PySpark
- Databricks
- Apache Spark
- Delta Lake
- Spark SQL
- Git / GitHub

---

## Dataset

**Source:** City of Chicago Food Inspections Open Data

Initial dataset:

- **316,703 records**
- **17 source columns**
- CSV source format

Important fields include:

- Inspection ID
- Business Name
- Facility Type
- Risk
- Address
- City
- State
- ZIP Code
- Inspection Date
- Inspection Type
- Results
- Violations
- Latitude
- Longitude

---

## Pipeline Layers

### Bronze Layer

The Bronze layer preserves the raw source data in Delta format.

```text
Records: 316,703
Columns: 17
```

The raw values are retained before transformation so that the original dataset remains available for traceability and reprocessing.

---

### Silver Layer

The Silver layer performs cleaning, standardization, validation, and data-quality checks.

Key transformations include:

- Converted `inspection_id` from string to long
- Converted `inspection_date` into Spark date format
- Converted latitude and longitude into double precision
- Standardized city and state values
- Trimmed whitespace
- Converted location values to uppercase
- Corrected known city-name inconsistencies
- Preserved original city values for traceability

The resulting Silver dataset contains:

```text
Records: 316,703
Columns: 23
```

No records were dropped during the transformation process.

---

## Data Quality Framework

Instead of deleting suspicious records, the pipeline assigns quality flags so they can be reviewed later.

### City Validation

Detects:

- Missing city
- Invalid city values

Results:

```text
NOT_FLAGGED     316,513
MISSING_CITY        182
INVALID_CITY          8
```

---

### State Validation

Detects:

- Missing states
- Out-of-state records
- Known state/ZIP mismatches

Results:

```text
NOT_FLAGGED             316,602
MISSING_STATE                76
OUT_OF_STATE_REVIEW          24
STATE_ZIP_MISMATCH            1
```

---

### ZIP Validation

Checks whether ZIP codes follow standard US ZIP formats.

Results:

```text
NOT_FLAGGED    316,661
MISSING_ZIP         42
```

---

### Coordinate Validation

Checks for:

- Missing coordinates
- Partial coordinates
- Invalid latitude/longitude ranges

Results:

```text
NOT_FLAGGED            315,645
MISSING_COORDINATES      1,058
```

---

## Overall Data Quality Status

Individual validation checks are combined into a single overall status.

```text
NOT_FLAGGED             315,407
USABLE_WITHOUT_GEO        1,028
REVIEW_REQUIRED              268
```

`REVIEW_REQUIRED` is prioritized when a record contains issues that may affect downstream analysis.

Records containing only missing geographic coordinates remain available as `USABLE_WITHOUT_GEO`.

---

## Gold Layer

The Gold layer produces analytics-ready datasets for downstream reporting and analysis.

### Gold Table 1 — Failure Rate by Risk Category

Only completed inspection outcomes were used:

- Pass
- Fail
- Pass w/ Conditions

Results:

| Risk Category | Total Inspections | Failed Inspections | Failure Rate |
|---|---:|---:|---:|
| Risk 1 (High) | 206,710 | 45,112 | 21.82% |
| Risk 2 (Medium) | 47,449 | 10,696 | 22.54% |
| Risk 3 (Low) | 17,679 | 5,133 | 29.03% |

These results describe the observed dataset and do not imply that risk category causes a particular failure rate.

---

### Gold Table 2 — Yearly Inspection Trends

The second Gold table aggregates completed inspections by year.

Metrics include:

- Total inspections
- Failed inspections
- Failure rate percentage

The dataset contains inspection records from:

```text
2010 – 2026
```

Total completed inspections:

```text
271,880
```

Total failed inspections:

```text
60,968
```

---

## Data Reconciliation

End-to-end reconciliation checks verify that transformations and aggregations remain consistent.

Final validation:

```text
Bronze Records:             316,703
Silver Records:             316,703
Yearly Gold Rows:                17
Completed Inspections:      271,880
Failed Inspections:          60,968
Risk Gold Rows:                   3
Risk-Eligible Inspections:  271,838
```

All reconciliation checks passed.

```text
PIPELINE STATUS: SUCCESS
```

---

## Key Data Engineering Concepts Demonstrated

This project demonstrates:

- ETL pipeline development
- PySpark transformations
- Data profiling
- Schema management
- Data type conversion
- Data cleaning
- Data standardization
- Data-quality validation
- Record-level quality flags
- Delta Lake storage
- Bronze / Silver / Gold architecture
- Aggregation pipelines
- Data reconciliation
- Analytics-ready dataset creation

---

## Repository Structure

```text
chicago-food-inspections-pyspark-etl/
│
├── README.md
│
├── notebooks/
│   └── Chicago_Food_Inspections_ETL_Final.ipynb
│
└── src/
    └── Chicago_Food_Inspections_ETL_Final.py
```

---

## Key Learning Outcomes

Through this project, I gained hands-on experience with:

- Processing large datasets using PySpark
- Designing layered Data Engineering pipelines
- Implementing data-quality rules without unnecessarily dropping records
- Working with Delta Lake tables
- Building reusable transformations
- Creating analytics-ready Gold datasets
- Validating data across pipeline layers
- Designing reconciliation checks to detect data loss or aggregation errors

---

## Future Improvements

Potential extensions include:

- Automated ingestion directly from the Chicago Open Data API
- Incremental ingestion instead of full refresh
- Databricks Workflows for orchestration
- Additional geographic validation
- Delta Lake `MERGE` for incremental processing
- Data-quality monitoring and alerts
- Unit and integration testing
- Dashboard integration using Power BI or Databricks SQL

---

## Author

**Rahul Kumar Nelakurthi**
