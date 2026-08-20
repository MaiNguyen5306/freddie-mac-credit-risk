# Performance Data Testing Workpaper

## 1. Workpaper Information

- **Project:** Freddie Mac Credit Risk Analysis
- **Testing area:** Monthly loan-performance data
- **Testing type:** Simulated Enterprise Independent Testing
- **Population:** Freddie Mac sample performance files
- **Vintages tested:** 2006, 2015, 2016, and 2017
- **Evidence source:** `notebooks/02_performance_audit.ipynb`
- **Execution method:** Python and pandas
- **Chunk size:** 100,000 records

> This is an independent portfolio simulation using publicly available
> Freddie Mac data. It is not an evaluation of Freddie Mac's internal
> controls.

## 2. Testing Objective

Evaluate whether monthly performance data is complete, structurally
consistent, appropriately linked to the origination population, and
suitable for subsequent credit-risk analysis.

## 3. Population Tested

| Vintage | Performance records | Unique loan IDs |
|---|---:|---:|
| 2006 | 3,203,499 | 50,000 |
| 2015 | 3,467,166 | 50,000 |
| 2016 | 3,379,650 | 50,000 |
| 2017 | 2,800,219 | 50,000 |
| **Total** | **12,850,534** | **200,000** |

The complete available sample population was tested. No statistical
sampling was used for the key integrity and value-validation controls.

## 4. Procedures Performed

The following procedures were executed:

1. Confirmed that all four expected performance files existed.
2. Verified that every file contained the expected 35 fields.
3. Validated loan identifiers and monthly reporting-period formats.
4. Reconciled performance loan identifiers to the cleaned origination
   population.
5. Tested for duplicate loan-and-month combinations.
6. Tested the chronological ordering of records within each loan.
7. Validated delinquency statuses against permitted values.
8. Validated modification flags and zero-balance codes.
9. Tested relevant date fields for the expected six-digit format.
10. Tested Current Actual UPB for blank, nonnumeric, and negative values.
11. Reviewed loan-age and remaining-maturity ranges.
12. Independently recalculated remaining maturity for identified extreme
    observations.

## 5. Key Testing Results

No exceptions were identified for:

- blank loan identifiers;
- invalid monthly reporting-period formats;
- unmatched performance loan identifiers;
- duplicate loan-month records;
- chronological sorting violations;
- invalid delinquency statuses;
- invalid modification flags;
- invalid zero-balance codes;
- invalid tested date formats;
- blank or nonnumeric Current Actual UPB; or
- negative Current Actual UPB.

All 200,000 unique performance loan identifiers matched the expected
origination population.

## 6. Credit-Performance Observations

The 2006 vintage contained substantially more distressed performance
records than the post-crisis vintages.

| Vintage | 90+ delinquent rows | REO rows |
|---|---:|---:|
| 2006 | 125,626 | 19,437 |
| 2015 | 20,958 | 567 |
| 2016 | 22,262 | 216 |
| 2017 | 30,072 | 352 |

These counts represent monthly performance records rather than unique
loans. They are credit-risk outcomes and are not data-quality
exceptions.

## 7. Remaining-Maturity Review

Sixty-four records were identified with remaining maturity below zero
or above 480 months.

Twenty-six negative-maturity records reconciled exactly to the
difference between the original maturity date and monthly reporting
period. These observations were not classified as exceptions.

One modified termination record could not be independently reconciled
because the modified maturity date was not separately available in the
sample data. This was treated as a testing limitation.

Loan `F06Q30294070` contained 37 records with reported remaining
maturity between 481 and 517 months. Independent calculation using the
original maturity date produced values between 324 and 360 months.

The reported values exceeded the independently calculated values by
157 months in every affected period. No applicable modification flag or
zero-balance code explained the difference.

## 8. Exception Assessment

- **Exception:** Remaining-maturity discrepancy
- **Affected population:** One loan and 37 monthly records
- **Loan population rate:** 0.0005%
- **Record population rate:** 0.00029%
- **Severity:** Low
- **Pattern:** Isolated and consistently offset
- **Impact:** Immaterial to the overall population, but relevant for
  transparent reporting and audit traceability

## 9. Testing Limitation

The sample data does not separately disclose the modified maturity date
used to calculate remaining maturity after a loan modification.
Therefore, one modified termination record could not be independently
recalculated using the available fields.

This limitation does not affect the conclusion for the broader tested
population.

## 10. Conclusion

The tested performance-data controls are assessed as **effective with
an isolated low-severity exception**.

The performance population is sufficiently complete, accurate, and
internally consistent for the planned credit-risk analysis, provided
that the documented remaining-maturity exception and modified-maturity
limitation are retained in the project documentation.
