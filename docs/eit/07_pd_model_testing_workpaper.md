# Baseline PD Model Testing Workpaper

## 1. Workpaper Purpose

This workpaper documents the development and independent evaluation of a baseline probability-of-default model using Freddie Mac Single-Family Loan-Level sample data.

The work supports Enterprise Independent Testing concepts by documenting the modeling population, methodology, test populations, performance results, control checks, limitations, and supporting evidence.

## 2. Model Objective

The model estimates the probability that an eligible mortgage loan experiences a 24-month default event.

A default is defined as:

- delinquency status `03` or greater; or
- real-estate-owned status `RA`;

within the first 24 monthly reporting periods measured from the original first payment date.

Loans without a complete 24-month observation window and without an observed default are treated as censored and excluded from model fitting and performance evaluation.

## 3. Model Methodology

The baseline model is an unweighted logistic regression.

Model configuration:

- L2 regularization;
- regularization parameter `C = 1`;
- `liblinear` solver;
- maximum 1,000 iterations;
- random seed 53;
- no class weighting;
- no performance-period variables used as predictors.

Preprocessing includes:

- development-population median imputation for numeric fields;
- numeric standardization;
- most-frequent imputation for categorical fields;
- one-hot encoding;
- first categorical level dropped as the reference;
- unknown categorical levels ignored during scoring.

The final model contains:

- 24 source predictors;
- 15 numeric predictors;
- 9 categorical predictors;
- 84 transformed model features.

The model converged successfully after 10 iterations.

## 4. Population Design

| Population | Vintage | Eligible loans | Defaults | Observed default rate | Intended use |
|---|---:|---:|---:|---:|---|
| Model development | 2015–2016 | 82,387 | 583 | 0.708% | Final model fitting |
| Later-vintage test | 2017 | 42,064 | 397 | 0.944% | Independent final testing |
| Historical comparison | 2006 | 37,555 | 967 | 2.575% | Historical sensitivity analysis |
| Total | 2006, 2015–2017 | 162,006 | 1,947 | — | Reconciled scoring population |

The 2017 and 2006 populations were not used to fit the final model.

Loan-identifier testing confirmed no overlap between the development, final-test, and historical-comparison populations.

## 5. Testing Procedures

The following procedures were completed:

1. Reconciled model populations to the validated 24-month outcome file.
2. Confirmed unique loan identifiers in each population.
3. Confirmed zero missing development and test targets.
4. Confirmed zero overlap between population partitions.
5. Confirmed the model converged successfully.
6. Saved and independently reloaded the fitted model artifact.
7. Generated probabilities using the reloaded model.
8. Reconciled 162,006 prediction rows to 162,006 unique eligible loans.
9. Confirmed zero missing predicted probabilities.
10. Compared observed and predicted default rates.
11. Calculated ROC AUC, average precision, and Brier score.
12. Evaluated top-decile lift and default capture.
13. Reviewed risk ordering across prediction deciles.
14. Reviewed coefficient direction and magnitude.
15. Tested scoring populations for categorical levels not observed during development.

## 6. Performance Results

| Population | Observed rate | Mean predicted probability | Prediction/observed ratio | ROC AUC | Average precision | Brier score | Top-decile lift | Top-decile capture |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Model development | 0.708% | 0.714% | 1.009 | 0.857 | 0.048 | 0.0069 | 5.64 | 56.4% |
| 2017 final test | 0.944% | 0.879% | 0.931 | 0.840 | 0.059 | 0.0091 | 5.04 | 50.4% |
| 2006 historical comparison | 2.575% | 3.255% | 1.264 | 0.802 | 0.092 | 0.0253 | 3.82 | 38.2% |

## 7. Independent Test Assessment

The 2017 final-test ROC AUC of 0.840 demonstrates strong discrimination and represents only a modest decrease from the development ROC AUC of 0.857.

The model slightly underpredicts the aggregate 2017 default rate:

- observed default rate: 0.944%;
- mean predicted probability: 0.879%;
- prediction-to-observed ratio: 0.931.

The highest-risk 10% of 2017 loans contains approximately 50.4% of all observed defaults and has a lift of approximately 5.04.

Observed default rates generally decrease from the highest-risk decile to the lowest-risk decile. Minor variation among the lowest-risk deciles is not considered significant because those deciles contain very few defaults.

## 8. Coefficient Review

The numeric coefficient review produced generally reasonable directional relationships:

- higher credit score is associated with lower default risk;
- more borrowers are associated with lower default risk;
- higher combined loan-to-value ratio is associated with higher default risk;
- higher debt-to-income ratio is associated with higher default risk;
- higher interest rate is associated with higher default risk;
- mortgage-insurance percentage is positively associated with default risk.

Categorical coefficients represent comparisons with an omitted reference level. They should not be interpreted as standalone or causal effects.

Correlated LTV and CLTV variables and their related indicators should also not be interpreted independently.

The credit-score missingness indicator has a fitted coefficient of zero because the development population contained no missing credit-score observations.

## 9. Historical Comparison Limitation

The 2006 historical population contains the channel value `T`, which was not present in the 2015–2016 development population.

Testing identified:

- 22,031 affected 2006 loans;
- 37,555 total eligible 2006 loans;
- 58.66% of the historical population affected;
- zero unknown categorical levels in the 2017 final-test population.

The encoder safely processes unknown levels using its configured `handle_unknown="ignore"` treatment. However, the model cannot separately estimate the effect of channel `T`.

Accordingly, the 2006 results are restricted to historical sensitivity analysis and should not be interpreted as equivalent to the primary 2017 independent test.

This item is documented as scope limitation `L-03`.

## 10. Testing Conclusion

The baseline model passed the primary development and independent-testing procedures.

Supporting conclusions include:

- model convergence confirmed;
- model artifact independently reloaded;
- scored population fully reconciled;
- zero missing probabilities;
- strong development and 2017 discrimination;
- effective final-test risk ordering;
- strong top-decile lift and default capture;
- mild aggregate underprediction in 2017;
- no unknown categories affecting the primary 2017 test.

The model is appropriate as a transparent portfolio baseline and analytical demonstration.

It is not represented as a production underwriting model. Production use would require additional governance, calibration, stability monitoring, fairness review, economic-cycle testing, challenger analysis, and formal model-risk validation.

## 11. Supporting Evidence

- `notebooks/05_pd_modeling.ipynb`
- `models/baseline_pd_logistic.joblib`
- `data/processed/pd_predictions.parquet`
- `data/processed/model_performance_summary.csv`
- `data/processed/model_decile_analysis.csv`
- `data/processed/model_coefficients.csv`
- `data/processed/unknown_category_audit.csv`
- `data/processed/modeling_dataset.parquet`
- `data/processed/feature_manifest.csv`
- `docs/eit/05_outcome_testing_workpaper.md`
- `docs/eit/06_feature_leakage_workpaper.md`
- `docs/eit/risk_control_matrix.xlsx`
- `docs/eit/exception_log.xlsx`