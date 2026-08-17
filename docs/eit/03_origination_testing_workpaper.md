# Origination-Data Testing Workpaper

## Workpaper information

- Engagement: Mortgage Data and Credit-Risk Reporting Control Testing
- Workpaper area: Origination-data ingestion, validation, and transformation
- Data vintages: 2006, 2015, 2016, and 2017
- Population: 200,000 origination records
- Primary evidence: `notebooks/01_data_audit.ipynb`
- Processed output: `data/processed/originations_clean.parquet`
- Testing status: Completed, with two procedures requiring future performance-data work

## Purpose

This workpaper documents testing performed over the simulated mortgage
origination-data process. The procedures evaluate whether the data population is
complete, identifiable, valid, consistently transformed, and supported by documented
exception review.

This educational case study uses public Freddie Mac data. It does not evaluate
Freddie Mac's internal controls or provide an audit or assurance conclusion.

## Population and scope

Testing covered four sample origination files:

- 2006 vintage
- 2015 vintage
- 2016 vintage
- 2017 vintage

The combined population contained 200,000 loans. Automated procedures generally
covered the complete origination population.

A preliminary referential-integrity procedure tested 100,000 monthly performance
records. Full-population performance testing remains part of the next project phase.

## Testing summary

| Test ID | Testing objective | Procedure performed | Result | Conclusion |
|---|---|---|---|---|
| EIT-01 | Confirm that files follow the approved schema. | Inspected file structure and applied the documented 31-field origination schema to all four vintages. | All files loaded successfully with 31 source fields. The combined table contained 32 columns after adding vintage. | Partially complete — origination schema passed; performance schema remains untested. |
| EIT-02 | Confirm population completeness through cleaning and export. | Reconciled raw, cleaned, exported, and reloaded record counts. | All stages contained 200,000 records. | Pass |
| EIT-03 | Confirm that loan identifiers are valid and unique. | Tested identifiers for missing and duplicate values. | 200,000 unique identifiers; zero missing and zero duplicates. | Pass |
| EIT-04 | Confirm that performance records link to valid originations. | Compared 100,000 sampled performance records with the origination identifier population. | Zero sampled performance records had unmatched identifiers. | Partially complete — the sample passed; full-population testing remains required. |
| EIT-05 | Confirm consistent treatment of blank and sentinel values. | Documented field-level missing codes, converted them to nulls in a separate cleaned table, and reconciled missingness by vintage. | Post-conversion missingness agreed with the original audit. Raw data remained unchanged. | Pass |
| EIT-06 | Confirm valid numeric and date transformations. | Converted governed numeric fields and YYYYMM date fields using documented rules. | Conversions completed without changing the record population or loan identifiers. | Pass |
| EIT-07 | Identify invalid ranges and category codes. | Profiled numeric minima and maxima and inspected categorical value sets. | Ordinary fields passed. Three loans exceeded the conservative ratio-review threshold. | Pass with observation |
| EIT-08 | Confirm that exceptions are investigated and documented. | Inspected the high-ratio loans using HARP indicators, pre-HARP identifiers, loan purpose, LTV, and CLTV. | All three loans were valid HARP refinance observations. Each had a pre-HARP identifier and CLTV greater than or equal to LTV. | Pass |

## Detailed results

### Population completeness

The following populations reconciled:

- Raw records: 200,000
- Cleaned records: 200,000
- Exported records: 200,000
- Reloaded Parquet records: 200,000
- Unique loan identifiers: 200,000
- Missing loan identifiers: 0
- Duplicate loan identifiers: 0

No records were lost or duplicated during the documented cleaning and export process.

### Missing-value treatment

The audit distinguished ordinary missing data from structurally unavailable or
not-applicable fields.

Key observations included:

- `vantagescore_4` was unavailable for 100% of the tested population.
- `property_valuation_method` was approximately 99.78% unavailable.
- Blank `pre_harp_loan_identifier` values were generally structurally not applicable.
- Blank `special_eligibility_program` values represented unavailable or
  not-applicable information.
- Core fields such as credit score, MSA, and DTI retained sufficient information for
  later analysis.

`vantagescore_4` and `property_valuation_method` were excluded from the primary
modeling plan because they contained almost no usable information.

### Exception review

Three loans exceeded the project's conservative LTV or CLTV review threshold.

The review determined that:

- All three had `harp_indicator = Y`.
- All three contained pre-HARP loan identifiers.
- All three were no-cash-out refinance loans.
- All three had CLTV greater than or equal to LTV.
- The records were internally consistent with high-ratio HARP refinancing.

The values were retained without capping or replacement. An
`extreme_ratio_review_flag` was added so these observations can be evaluated in later
sensitivity analysis.

## Exceptions and observations

### Observation O-01 — Sparse fields

VantageScore 4 and property valuation method contained insufficient usable information
for the primary model.

Disposition: Exclude these fields from the primary modeling feature set and document
the limitation.

### Observation O-02 — High mortgage ratios

Three legitimate HARP loans exceeded the project's general ratio-review threshold.

Disposition: Retain the disclosed values, maintain the review flag, and assess their
effect during sensitivity testing.

### Scope limitation L-01 — Performance referential integrity

The initial identifier procedure covered 100,000 performance rows rather than the
complete performance population.

Disposition: Perform a chunked full-population identifier test during the monthly
performance audit.

## Overall conclusion

The completed procedures support a preliminary conclusion that the origination-data
pipeline preserved population completeness, identifier integrity, documented
missing-value treatment, and reproducible exception review.

No material data-integrity exceptions were identified within the completed
origination-data scope.

Final conclusions over the broader mortgage reporting process cannot be reached until
monthly performance files, delinquency outcomes, and reporting reconciliations have
been tested.

## Supporting evidence

- `notebooks/01_data_audit.ipynb`
- `data/processed/originations_clean.parquet`
- `docs/eit/01_process_narrative.md`
- `docs/eit/02_eit_test_plan.md`
- `docs/eit/risk_control_matrix.xlsx`