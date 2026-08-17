# Mortgage Data and Credit-Risk Reporting Process Narrative

## Case-study purpose

This case study uses the public Freddie Mac Single-Family Loan-Level Dataset to
simulate how an Enterprise Independent Testing team could evaluate data controls
supporting mortgage credit-risk analysis and reporting.

The case study does not assess Freddie Mac's internal controls and does not claim
access to Freddie Mac's internal systems, procedures, employees, or control evidence.
The business process, control activities, and organizational roles described below
are hypothetical and were created for portfolio and educational purposes.

## Business objective

The simulated business process is intended to produce complete, accurate, valid,
and traceable mortgage data for portfolio monitoring, delinquency analysis, and
credit-risk reporting.

Management and risk analysts should be able to rely on the resulting data when
evaluating portfolio composition, borrower risk, delinquency performance, vintage
trends, and probability of default.

## Hypothetical stakeholders

- Data Operations receives and loads mortgage origination and performance files.
- Credit Risk Analytics transforms the data and calculates portfolio risk measures.
- Risk Management reviews trends, exceptions, and risk indicators.
- Enterprise Independent Testing evaluates whether key controls are designed and
  executed consistently.
- Management uses validated reports and analytical outputs to support decisions.

## End-to-end process

### 1. Data acquisition

Origination and monthly performance files are received from an approved source.
The expected files, vintages, layouts, and reporting periods are documented.

### 2. Data ingestion

Raw pipe-delimited files are loaded without changing the source data. File existence,
row counts, column counts, and field order are checked before transformation.

### 3. Data validation and cleaning

Loan identifiers, missing-value codes, numeric ranges, categorical values, and date
formats are validated. Raw missing-value codes are converted to proper null values
in a separate cleaned dataset while the raw data remains unchanged.

### 4. Exception identification and review

Records outside established review thresholds are flagged. Exceptions are investigated
using related loan attributes and supporting documentation before values are corrected,
excluded, or retained.

### 5. Origination and performance integration

Monthly performance records are matched to valid origination records using loan
identifiers. Unmatched records, duplicate loan-month observations, invalid reporting
periods, and incomplete performance windows are identified and documented.

### 6. Risk-measure calculation

Validated monthly records are used to calculate delinquency outcomes, vintage
performance, risk segments, and an explainable probability-of-default model.
Observation and outcome windows are separated to prevent future-information leakage.

### 7. Reporting and monitoring

Summary tables and dashboards communicate portfolio trends, risk indicators,
testing exceptions, assumptions, and limitations. Analytical outputs are reconciled
to validated source populations before reporting.

## Current completed evidence

The origination-data audit has:

- Validated 200,000 origination records across the 2006, 2015, 2016, and 2017 vintages.
- Confirmed that all 200,000 loan identifiers are unique and nonmissing.
- Matched all 100,000 sampled performance records to origination identifiers.
- Applied and validated the official 31-field origination schema.
- Audited blank values and Freddie Mac missing-value codes.
- Validated numeric ranges, categorical values, and date conversions.
- Investigated three unusually high mortgage-ratio records.
- Determined that all three reviewed records were legitimate HARP observations.
- Preserved the reviewed values and created an exception-review flag.
- Exported and reloaded a validated 200,000-row, 33-column Parquet dataset.

## Next testing phase

The next phase will evaluate the monthly performance-data process. Testing will cover
file completeness, schema validity, loan-identifier matching, duplicate loan-month
records, reporting-period validity, delinquency-status values, performance-window
coverage, and repeatable calculation of loan-level delinquency outcomes.