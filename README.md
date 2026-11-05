# Fintech-Loan-Analytics
End-to-end Fintech Loan Analytics using Google BigQuery, SQL and Looker. Includes cloud data transformation, financial analysis and interactive dashboard development.
# Fintech Loan Analytics & Executive Dashboard
### Google BigQuery | SQL | Google Cloud Storage | Looker | LookML | Data Analytics

## Project Overview

Developed an end-to-end Fintech Loan Analytics solution using Google BigQuery and Looker to analyze loan portfolios, monitor outstanding balances, evaluate loan distribution, and provide actionable business insights through an interactive dashboard.

The project focuses on helping the Treasury department monitor loan health, understand financial exposure, identify geographic concentrations, and evaluate borrower profiles through SQL-based data transformation and business intelligence reporting.

**Project Type:** Independent Cloud Data Analytics Project  
**Domain:** FinTech | Banking | Financial Analytics  
**Tools:** Google BigQuery, Google Cloud Storage, SQL, Looker, LookML, Google Sheets

---

## Business Problem

The Treasury department at TheLook Fintech needed a centralized analytical solution to answer important questions about its loan portfolio.

The key business requirements were:

- What is the total outstanding amount across all loans?
- What percentage of outstanding loans falls into each loan status category?
- Which US states have the highest number of outstanding loans?
- Which customers own their homes outright and have current loans?
- How can stakeholders monitor these metrics through an interactive dashboard?

The objective was to transform raw loan and customer data into structured analytical tables and an interactive dashboard for financial monitoring.

---

## Project Objectives

1. Explore and understand loan and customer datasets using BigQuery.
2. Import external geographical mapping data from Google Cloud Storage.
3. Combine loan and regional information using SQL joins.
4. Transform data using CTAS statements and nested-field extraction.
5. Remove duplicate loan purposes and create aggregated reporting tables.
6. Analyze outstanding loan balances, loan status, and borrower distribution.
7. Build an interactive Looker dashboard to monitor loan portfolio health.

---

## Technology Stack

| Technology | Application |
|---|---|
| Google BigQuery | Cloud data exploration, querying and transformation |
| SQL | Joins, aggregations, filtering and data deduplication |
| Google Cloud Storage | Source CSV ingestion |
| Looker | Interactive dashboard development |
| LookML | Exploration of dimensions and measures |
| Google Sheets | Connected Sheets for analytical reporting |

---

## Dataset Overview

The project uses the Fintech dataset available in the Google Cloud training environment.

**Primary tables:**

| Table | Purpose |
|---|---|
| `loan` | Loan amounts, statuses, issue dates, interest rates and application details |
| `customer` | Customer information, annual income and homeownership |
| `state_region` | US state, subregion and regional classifications |
| `loan_with_region` | Combined loan and geographical information |
| `loan_purposes` | Deduplicated loan purpose categories |
| `loan_count_by_year` | Annual loan count summary |

**External data source:**

US state-to-region mapping CSV files were imported from Google Cloud Storage.

---

## Project Implementation

### 1. BigQuery Data Exploration

Explored the Fintech dataset to identify the relevant tables, columns, data types and nested records.

Key activities:
- Examined table details, schemas and sample records.
- Identified `loan_amount` as the loan amount field.
- Located `issue_date` and `issue_year` for time-based analysis.
- Identified `int_rate` for interest rate analysis.
- Explored the nested `application` record to locate loan purpose information.

### 2. Importing External Data

Imported the state-region mapping CSV file from Google Cloud Storage into BigQuery.

```sql
LOAD DATA OVERWRITE fintech.state_region
(
  state STRING,
  subregion STRING,
  region STRING
)
FROM FILES (
  format = 'CSV',
  uris = [
    'gs://sureskills-lab-dev/future-workforce/da-capstone/temp_35_us/state_region_mapping/state_region_*.csv'
  ]
);
```

**Outcome:** Created a structured geographical mapping table for regional loan analysis.

### 3. Joining Loan and Regional Data

Combined loan information with geographical classifications using an INNER JOIN on the state column.

```sql
SELECT
    lo.loan_id,
    lo.loan_amount,
    sr.region
FROM fintech.loan AS lo
INNER JOIN fintech.state_region AS sr
    ON lo.state = sr.state;
```

**Outcome:** Integrated loan and geographical information into a single analytical dataset.

### 4. Creating an Analytical Table Using CTAS

Created a reusable table to store the joined dataset.

```sql
CREATE OR REPLACE TABLE fintech.loan_with_region AS
SELECT
    lo.loan_id,
    lo.loan_amount,
    sr.region
FROM fintech.loan AS lo
INNER JOIN fintech.state_region AS sr
    ON lo.state = sr.state;
```

**Outcome:** Created `loan_with_region` for downstream analysis and Google Sheets Connected Sheets integration.

### 5. Working with Nested Data

Extracted loan purpose information from the nested `application` record using BigQuery dot notation.

```sql
SELECT
    loan_id,
    application.purpose AS purpose
FROM fintech.loan;
```

**Outcome:** Retrieved borrower loan purposes from nested application data.

### 6. Data Deduplication

Identified duplicate loan purpose categories and extracted unique values using DISTINCT.

```sql
CREATE OR REPLACE TABLE fintech.loan_purposes AS
SELECT DISTINCT
    application.purpose AS purpose
FROM fintech.loan
WHERE application.purpose IS NOT NULL;
```

**Outcome:** Created a separate table containing unique loan purpose categories.

### 7. Annual Loan Amount Analysis

Aggregated loan amounts by issue year to understand annual loan disbursement trends.

```sql
SELECT
    issue_year,
    SUM(loan_amount) AS total_amount
FROM fintech.loan
GROUP BY issue_year
ORDER BY issue_year;
```

**Outcome:** Calculated annual loan disbursement totals for financial reporting.

### 8. Annual Loan Count Analysis

Created an aggregated table to track the number of loans issued each year.

```sql
CREATE OR REPLACE TABLE fintech.loan_count_by_year AS
SELECT
    issue_year,
    COUNT(loan_id) AS total_loans
FROM fintech.loan
GROUP BY issue_year
ORDER BY issue_year;
```

**Outcome:** Developed a year-wise loan count summary for trend analysis.

---

## Looker Dashboard – Loan Insights

Developed an interactive Looker dashboard named **Loan Insights** to provide the Treasury department with a consolidated view of loan portfolio health.

The dashboard contains four key visualizations.

### Dashboard Components

**1. Total Amount of Outstanding Loans**

- KPI: Outstanding loan balance.
- Measure: Outstanding Loans Amount.
- Visualization: Single Value.
- Conditional formatting threshold: $3 billion.
- Purpose: Monitor total financial exposure against the defined lending threshold.

**2. Percentage of Outstanding Loans by Status**

- Dimension: Loan Status.
- Measure: Count.
- Visualization: Pie Chart.
- Purpose: Understand the composition of the loan portfolio across different repayment status categories.

**3. Total Count of Outstanding Loans by State**

- Dimension: State.
- Measure: Outstanding Loans Count.
- Visualization: Bar Chart.
- Filter: Top 10 states by outstanding loan count.
- Purpose: Identify geographical concentrations of outstanding loans.

**4. Top 10 Customers by Highest Income**

- Dimensions:
  - Customer ID
  - Annual Income
  - State
  - Interest Rate
- Filters:
  - Homeownership: OWN
  - Loan Status: Current
- Sorting: Annual Income, descending.
- Row limit: 10.
- Visualization: Table.
- Purpose: Identify high-income customers who own their homes outright and have current loans.

---

## Dashboard Functionality

Implemented additional dashboard functionality to improve usability and analytical exploration.

- **Cross-filtering:** Enabled interactive filtering across dashboard visualizations.
- **Automatic refresh:** Configured refresh intervals for individual visualizations.
- **Dashboard refresh:** Set the dashboard refresh interval to hourly.
- **Conditional formatting:** Applied a red indicator when outstanding loan balances exceed $3 billion.
- **Data exploration:** Used LookML dimensions and measures to build business-focused visualizations.
- **Dashboard organization:** Arranged and labeled visualizations for easier interpretation.

---

## Key Business Insights

The solution enables stakeholders to:

1. Monitor total outstanding loan exposure against a predefined $3 billion threshold.
2. Evaluate loan portfolio composition across repayment statuses.
3. Identify states with concentrated outstanding loan volumes.
4. Examine the income profiles of customers with outright homeownership and current loans.
5. Track annual loan disbursement amounts and loan volumes.
6. Explore borrower loan purposes to understand borrowing patterns.

**Business Value:** Provides a centralized reporting solution that supports loan portfolio monitoring, financial exposure analysis, geographical risk assessment and customer segmentation.

---

## Skills Demonstrated

**SQL & Data Analytics**
- Data exploration and profiling
- INNER JOIN
- CTAS (CREATE TABLE AS SELECT)
- GROUP BY and aggregate functions
- DISTINCT and deduplication
- Nested and STRUCT field extraction
- Filtering and sorting
- Financial and time-based analysis

**Cloud Data & Business Intelligence**
- Google BigQuery
- Google Cloud Storage
- Looker and LookML
- Dimensions and measures
- Dashboard development
- KPI monitoring
- Conditional formatting
- Cross-filtering and scheduled refreshes
- Google Sheets Connected Sheets

**Business & Analytical Skills**
- Financial exposure monitoring
- Loan portfolio analysis
- Customer segmentation
- Geographic distribution analysis
- Business requirement translation
- Data-driven reporting

---

## Project Deliverables

| Deliverable | Description |
|---|---|
| `state_region` | Imported geographical mapping table |
| `loan_with_region` | Joined loan and regional dataset |
| `loan_purposes` | Distinct borrower loan purposes |
| `loan_count_by_year` | Annual loan count summary |
| `Loan Insights` | Interactive Looker dashboard |
| Connected Sheets | Spreadsheet-connected analytical data |

---

## Repository Structure

```text
Fintech-Loan-Analytics/
│
├── README.md
│
├── SQL/
│   ├── 01_data_exploration.sql
│   ├── 02_import_state_region.sql
│   ├── 03_join_loan_region.sql
│   ├── 04_create_loan_with_region.sql
│   ├── 05_nested_data_analysis.sql
│   ├── 06_deduplicate_loan_purposes.sql
│   ├── 07_annual_loan_amount.sql
│   └── 08_annual_loan_count.sql
│
├── Dashboard/
│   └── loan_insights_dashboard.png
│
└── Documentation/
    └── project_workflow.md
```

## Conclusion

This project demonstrates an end-to-end cloud data analytics workflow, from exploring and transforming raw financial data in BigQuery to developing an interactive Looker dashboard.

By combining SQL-based data preparation, cloud data integration, financial analysis and business intelligence reporting, the solution provides stakeholders with a structured way to monitor outstanding loans, assess portfolio distribution and explore customer-level information.

**Core Focus:** Transforming raw financial data into meaningful business insights using Google Cloud, SQL and Looker.
