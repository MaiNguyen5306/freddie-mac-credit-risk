# Feature Engineering and Leakage Review Workpaper

## 1. Workpaper Information

- **Project:** Freddie Mac Credit Risk Analysis
- **Testing area:** Feature engineering and model leakage controls
- **Testing type:** Simulated Enterprise Independent Testing
- **Evidence source:** `notebooks/04_feature_engineering.ipynb`
- **Primary outcome:** 24-month 90-plus-day delinquency or REO
- **Development vintages:** 2015–2016
- **Test vintage:** 2017
- **Historical comparison vintage:** 2006

> This is an independent portfolio simulation using publicly available
> Freddie Mac data. It is not an evaluation of Freddie Mac's internal
> controls.

## 2. Testing Objective

Evaluate whether the modeling population and candidate predictors are
complete, traceable, temporally appropriate, and protected from
post-origination information leakage.

## 3. Population Reconciliation

The feature-engineering process merged:

- 200,000 validated origination records; and
- 200,000 validated loan-level outcome records.

The merge produced 200,000 unique loan-level records with:

- zero duplicate loan identifiers;
- zero missing outcomes;
- zero missing population assignments; and
- 124,451 model-eligible loans across 2015–2017.

## 4. Time-Based Population Design

| Population | Vintages | Eligible loans | Defaults | Default rate |
|---|---|---:|---:|---:|
| Development | 2015–2016 | 82,387 | 583 | 0.708% |
| Later-vintage test | 2017 | 42,064 | 397 | 0.944% |
| Historical comparison | 2006 | 37,555 | 967 | 2.575% |

No loan identifiers overlapped between the three populations.

Censored loans were excluded from supervised model development and
evaluation rather than being treated as known nondefaults.

## 5. Predictor Selection

Twenty-four origination-time predictors were approved:

- 15 numeric predictors; and
- nine categorical predictors.

The selected fields represent borrower credit characteristics, original
loan structure, property characteristics, and origination channel
information available at or before origination.

## 6. Excluded Features

The following fields were excluded:

| Feature | Reason |
|---|---|
| `property_valuation_method` | 99.70% missing and one observed level |
| `special_eligibility_program` | 96.52% missing and unstable coverage |
| `amortization_type` | Constant |
| `interest_only_indicator` | Constant |
| `prepayment_penalty_indicator` | Constant |

High-cardinality identifiers and location fields such as seller name,
MSA, and postal code were also excluded from the baseline predictor
set.

`vantagescore_4` was excluded because it was not available for the
historical vintages used in this project.

## 7. Feature Transformations

The following transformations were performed:

- Original UPB was transformed using the natural logarithm of UPB plus
  one.
- LTV and CLTV were capped at 200 for their continuous model features.
- Separate indicators identify original LTV and CLTV values above 100.
- Missing-value indicators were created for credit score, DTI, LTV,
  and CLTV.
- Original source values remain preserved in the validated origination
  data for audit traceability.

No high-ratio loans were deleted.

## 8. Missingness Review

DTI contained the only material missingness among retained numeric
features:

- development population: 6.996%;
- test population: 3.763%.

Other retained numeric features had either no missingness or only
isolated missing values.

Median imputation will be fitted using only the 2015–2016 development
population during model-pipeline construction.

## 9. Categorical Stability Review

All nine retained categorical predictors were compared between the
development and 2017 test populations.

No categorical levels appeared in the 2017 test population that were
absent from the development population.

The future encoding pipeline will nevertheless ignore unknown
categories safely to support reproducible scoring.

## 10. Leakage Review

The approved predictor list was compared with prohibited
post-origination fields.

Prohibited information included:

- loan outcomes and censoring indicators;
- monthly performance balances;
- delinquency statuses;
- loan age and servicing history;
- modifications;
- zero-balance events;
- actual losses; and
- other future performance information.

No prohibited fields were included in the predictor list.

Dates, vintage, population role, and loan identifiers are retained only
for traceability and population assignment and are not model
predictors.

## 11. Output Files

The validated outputs are:

- `data/processed/modeling_dataset.parquet`
- `data/processed/feature_manifest.csv`

The modeling dataset contains:

- 200,000 rows;
- 30 columns;
- 24 approved predictors; and
- zero duplicate loan identifiers.

## 12. Preliminary Conclusion

Feature selection, time-based population separation, categorical
stability testing, and predictor-level leakage controls passed without
exception.

EIT model-methodology review remains partially complete until the
preprocessing pipeline and baseline model are confirmed to fit all
imputation, scaling, encoding, and estimation steps using only the
2015–2016 development population.
