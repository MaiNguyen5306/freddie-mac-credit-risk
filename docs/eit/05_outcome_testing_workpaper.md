# Loan Outcome Construction and Testing Workpaper

## 1. Workpaper Information

- **Project:** Freddie Mac Credit Risk Analysis
- **Testing area:** Loan-level credit-outcome construction
- **Testing type:** Simulated Enterprise Independent Testing
- **Population:** Freddie Mac sample origination and performance data
- **Vintages:** 2006, 2015, 2016, and 2017
- **Primary horizon:** 24 months
- **Additional horizons:** 12 and 36 months
- **Evidence source:** `notebooks/03_outcome_construction.ipynb`

> This is an independent portfolio simulation using publicly available
> Freddie Mac data. It is not an evaluation of Freddie Mac's internal
> controls.

## 2. Testing Objective

Determine whether monthly performance records can be transformed into
complete, accurate, and reproducible loan-level credit outcomes suitable
for vintage analysis and probability-of-default modeling.

## 3. Outcome Definition

A default event is defined as:

- delinquency status `03` or higher; or
- REO acquisition status `RA`.

The primary outcome is `default_24m`, representing whether a loan
experienced a default event during month indexes 0 through 23, measured
from the original first-payment date.

Additional 12-month and 36-month outcomes were constructed for
sensitivity and vintage analysis.

A loan is classified as:

- **Default:** a qualifying event occurred during the applicable window;
- **Nondefault:** no qualifying event occurred and the complete window
  was observed; or
- **Censored:** no qualifying event was observed, but the complete
  outcome window was unavailable.

## 4. Population and Processing Method

The outcome-construction procedure processed:

- 200,000 origination loans;
- 12,850,534 monthly performance records;
- four vintage-level performance files; and
- 100,000 records per processing chunk.

Only selected fields were loaded from each performance file to control
memory usage.

One final outcome record was created for every origination loan.

## 5. Outcome Results

| Vintage | 12-month rate | 24-month rate | 36-month rate |
|---|---:|---:|---:|
| 2006 | 0.58% | 2.57% | 8.12% |
| 2015 | 0.15% | 0.63% | 1.26% |
| 2016 | 0.16% | 0.78% | 1.27% |
| 2017 | 0.42% | 0.94% | 5.08% |

The 2006 vintage exhibited materially higher early credit risk than the
2015–2017 vintages.

The 2017 vintage showed a notable increase between the 24-month and
36-month horizons. Calendar-period analysis is required before
attributing this increase to a particular economic event.

## 6. Full-Population Validation

The following controls were performed on the constructed outcome table:

1. Reconciled outcome rows to the origination population.
2. Reconciled supporting performance-record counts.
3. Tested for duplicate loan-level outcome records.
4. Confirmed that every loan had performance data.
5. Validated all outcome values.
6. Reconciled eligibility and censoring classifications.
7. Confirmed that nondefault outcomes had complete observation windows.
8. Confirmed that default events were preserved across longer horizons.
9. Confirmed that observed-month counts did not exceed their horizons.
10. Reloaded the exported Parquet file and reconciled all default counts.

No exceptions were identified.

## 7. Independent Outcome Recalculation

A stratified control sample of 120 loans was selected:

- 10 default loans per vintage;
- 10 nondefault loans per vintage; and
- 10 censored loans per vintage.

The sample was designed for control validation and was not used to
estimate population default rates.

A total of 6,791 monthly records were retrieved directly from the raw
performance files. Observation months and 24-month outcomes were
recalculated separately from the exported outcome table.

| Validation measure | Result |
|---|---:|
| Loans tested | 120 |
| Observation-window matches | 120 |
| Outcome matches | 120 |
| Eligibility matches | 120 |
| Censoring matches | 120 |
| Total mismatches | 0 |

## 8. Output Files

The validated output files are:

- `data/processed/loan_outcomes.parquet`
- `data/processed/outcome_summary.csv`

The Parquet outcome file contains 200,000 rows and 18 columns.

## 9. Methodological Considerations

Early exits without an observed default are classified as censored
rather than automatically classified as nondefault.

The derived outcome is an analytical portfolio definition based on
90-plus-day delinquency or REO status. It should not be presented as a
regulatory default definition.

The original first-payment date is used to establish the outcome window
because the disclosed loan-age field may reset following a loan
modification.

## 10. Conclusion

Outcome-window eligibility testing passed without exception.

Independent outcome recalculation also passed without exception. All
120 sampled outcomes and supporting classifications agreed with the
constructed outcome table.

The 24-month outcome is approved for subsequent feature engineering,
vintage analysis, and probability-of-default modeling.