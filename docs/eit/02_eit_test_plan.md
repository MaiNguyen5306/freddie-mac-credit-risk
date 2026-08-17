# Enterprise Independent Testing Plan

## 1. Engagement title

Mortgage Data and Credit-Risk Reporting Control Testing

## 2. Purpose

This case study simulates how an Enterprise Independent Testing team could evaluate
data controls supporting mortgage portfolio monitoring and credit-risk reporting.

The testing is designed to determine whether the hypothetical process produces data
that is complete, accurate, valid, traceable, and suitable for calculating loan-level
delinquency outcomes and portfolio-risk measures.

This is an educational portfolio project using public Freddie Mac data. It does not
evaluate Freddie Mac's internal controls or represent work performed for a financial
institution.

## 3. Testing objectives

The testing objectives are to determine whether:

1. Expected mortgage data files are received and processed using the correct schemas.
2. Record populations remain complete through ingestion, cleaning, and export.
3. Loan identifiers are valid, unique, and traceable across data sources.
4. Missing-value and data-type transformations follow documented rules.
5. Invalid numeric, categorical, date, and delinquency values are identified.
6. Testing exceptions are investigated and supported by documented evidence.
7. Monthly performance records are complete and chronologically valid.
8. Derived delinquency outcomes agree with the underlying monthly records.
9. Model predictors are separated from future performance information.
10. Reported totals and risk indicators reconcile to validated analytical populations.

## 4. Scope

### In scope

- Origination records from the 2006, 2015, 2016, and 2017 sample vintages
- Monthly performance records for the same vintages
- File and schema validation
- Population completeness and reconciliation
- Identifier uniqueness and referential integrity
- Missing-value transformations
- Numeric, categorical, and date validation
- Exception identification and disposition
- Duplicate loan-month testing
- Reporting-period and loan-age validation
- Delinquency-status validation
- Performance-window eligibility
- Loan-level delinquency outcome calculation
- Future-information leakage review
- Reporting and dashboard reconciliation

### Out of scope

- Freddie Mac's actual internal controls or systems
- Employee access, cybersecurity, or system-permission controls
- Vendor-management controls
- Production change-management procedures
- Legal or regulatory compliance conclusions
- Personally identifiable borrower information
- Independent validation of proprietary Freddie Mac models
- An audit opinion or assurance conclusion

## 5. Testing populations

The origination population contains 200,000 records across four vintages:

- 2006
- 2015
- 2016
- 2017

The monthly performance population consists of four pipe-delimited files totaling
approximately 1.31 GB.

Automated tests will generally cover the full available population. Manual testing
will focus on identified exceptions or targeted records requiring judgment.

The earlier 100,000-row performance identifier test is preliminary evidence only.
Full-population referential-integrity testing will be completed during the performance
audit.

## 6. Testing methodology

Testing procedures may include:

- Inspection of file layouts and documented definitions
- Population-level automated testing
- Reconciliation between processing stages
- Reperformance of calculations
- Comparison with approved value sets and review thresholds
- Targeted inspection of exceptions
- Independent recalculation of selected loan outcomes
- Review of predictor timing and model-development logic
- Comparison of dashboard results with controlled summary tables

Each test will document:

- Test identifier
- Risk and control objective
- Population and period
- Testing procedure
- Expected result
- Actual result
- Exceptions identified
- Supporting evidence
- Conclusion
- Follow-up action, when required

## 7. Conclusion categories

### Pass

The test met its defined criteria, and no exceptions affecting the control objective
were identified.

### Pass with observation

The test met its primary objective, but an opportunity for clarification,
standardization, monitoring, or process improvement was identified.

### Fail

One or more exceptions indicate that the tested control objective was not achieved.
The finding requires documented impact assessment and recommended corrective action.

### Not tested

The required data, procedure, or project phase is not yet available. No control
conclusion can be reached.

## 8. Exception evaluation

Exceptions will not automatically be treated as control failures. Each exception will
be evaluated based on:

- Nature of the exception
- Number and percentage of affected records
- Potential effect on risk measures or reporting
- Whether the exception is isolated or systematic
- Whether supporting evidence explains the result
- Whether the issue can be reproduced
- Whether corrective or preventive action is needed

Valid unusual records may be retained when supporting attributes and documentation
show that the values are legitimate.

## 9. Evidence standards

Supporting evidence should be clear, reproducible, and traceable. Evidence may include:

- Jupyter notebooks containing testing code and results
- Population and reconciliation summaries
- File metadata and schema results
- Exception-level review tables
- Cleaned Parquet outputs
- Risk-and-control matrix
- Exception log
- Git commits showing documentation and code history
- Final testing-results summary

Large raw and processed loan files will remain excluded from Git. The repository will
contain the code and documentation needed to reproduce the testing.

## 10. Technology and resource constraints

The project is executed on a computer with 8 GB of RAM. Monthly performance testing
will therefore:

- Process one vintage at a time
- Read large files in chunks
- Retain only required columns
- Aggregate records as early as practical
- Save compact intermediate Parquet outputs
- Avoid loading all monthly files simultaneously
- Use single-process modeling where applicable
- Avoid large parallel model searches

These safeguards are part of the repeatable testing methodology rather than a change
to the testing objectives.

## 11. Testing phases

### Phase 1 — Origination-data testing

Covers EIT-01 through EIT-08. The primary origination audit has been completed,
subject to final workpaper preparation.

### Phase 2 — Monthly performance-data testing

Covers performance schema, full-population identifier matching, duplicate loan-month
records, reporting periods, loan ages, delinquency statuses, and observation-window
coverage.

### Phase 3 — Outcome and model-control testing

Covers independent recalculation of delinquency outcomes, eligibility rules, and
future-information leakage prevention.

### Phase 4 — Reporting and monitoring testing

Covers reconciliation of portfolio summaries, dashboards, trends, and risk indicators
to validated source populations.

## 12. Planned deliverables

- Business-process narrative
- Risk-and-control matrix
- EIT testing plan
- Origination testing workpaper
- Monthly performance testing workpaper
- Exception log
- Testing-results summary
- Portfolio-risk analytical outputs and monitoring dashboard