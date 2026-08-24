# Freddie Mac Credit Risk: Probability of Default Modeling and Independent Testing

This project uses Freddie Mac Single-Family Loan-Level sample data to estimate the probability that a mortgage will experience a serious payment failure within its first 24 months. For this analysis, **default** is operationally defined as a loan becoming at least three monthly payments past due or entering **real-estate-owned (REO)** status, meaning the lender or mortgage investor has taken ownership of the property after foreclosure.

The model uses only information available when the loan was originated, such as credit score, debt-to-income ratio, loan-to-value ratio, interest rate, loan amount, and property characteristics.

The project also simulates **Enterprise Independent Testing (EIT)** work by checking whether the source data, default calculations, model inputs, testing populations, and reported results are complete, accurate, and reproducible.

## What Does the Model Predict?

Each eligible loan has an observed 24-month outcome:

* `1` means the loan defaulted within 24 months.
* `0` means the loan completed the 24-month observation period without defaulting.

The logistic regression model learns patterns from these historical outcomes. Instead of returning only `0` or `1`, it produces a **Probability of Default (PD)** for each loan.

For example, a predicted PD of `0.03` means the model estimates a 3% probability that the loan will default within 24 months. A higher PD indicates greater estimated risk, but it does not guarantee that the loan will default.

## Project Objective

The project addresses two questions:

1. Can origination information identify and rank loans with elevated 24-month default risk?
2. Can an independent tester reproduce the data populations, default outcomes, model results, and reported conclusions?

For this project, a default occurs when a loan:

* reaches delinquency status `03` or greater, meaning it is at least three monthly payments past due; or
* reaches real-estate-owned status `RA`;

within the first 24 monthly reporting periods after the original first payment date.


## Data Scope

The project uses Freddie Mac Single-Family Loan-Level sample data. A **vintage** is the year in which a group of mortgage loans was originated.

Four vintages were selected for different analytical purposes:

* **2006 — historical comparison:** Used to examine how the final model behaves on loans from an older and economically different period.
* **2015 — initial model training:** Used to train the first version of the model.
* **2016 — development-stage validation:** Used to evaluate the first version on newer, unseen data while the methodology was still being developed.
* **2017 — independent later-vintage test:** Kept out of model development and used as the final test of the completed model.

All four vintages were audited for data quality before modeling. The first model was built using 2015 loans and checked using 2016 loans. We then combined 2015 and 2016 to build the final model and checked it using 2017 loans. The 2017 outcomes were used only to evaluate the completed model, not to teach the model how to make predictions.

### Source Data Structure

The source data contains two main types of records:

* **Origination records:** One record per loan containing information known when the mortgage began, such as credit score, original balance, interest rate, loan-to-value ratio, and property characteristics.
* **Monthly performance records:** Repeated records showing what happened to each loan over time, including payment status, delinquency, remaining balance, modification information, payoff, and foreclosure-related outcomes.

### Validated Populations

The validated source data contains:

- 200,000 unique originated loans;
- 12,850,534 monthly performance records;
- 33 cleaned origination fields;
- 35 monthly performance fields.

After applying the 24-month outcome requirements, 162,006 loans were eligible for model development or evaluation. The other loans were excluded because they did not default and did not have enough monthly history to confirm a complete 24-month nondefault outcome.

The population and data-quality results can be reviewed in:

- [`01_data_audit.ipynb`](notebooks/01_data_audit.ipynb) — origination cleaning and validation;
- [`02_performance_audit.ipynb`](notebooks/02_performance_audit.ipynb) — monthly performance validation;
- [`03_outcome_construction.ipynb`](notebooks/03_outcome_construction.ipynb) — default, eligibility, and censoring calculations.

### Why Use a 24-Month Outcome?

The project constructed 12-, 24-, and 36-month outcomes and selected 24 months as the primary modeling period. This is a project design choice, not a universal definition.

A 12-month period may be too short to observe enough defaults, while a 36-month period requires more performance history and leaves fewer loans with complete observation periods. The 24-month period provides a practical balance between allowing time for defaults to occur and retaining enough eligible loans for modeling and testing.

### Class Imbalance

In classification, a **class** is one possible outcome category. In this project, class `1` represents default and class `0` represents nondefault. The dataset is **class imbalanced** because nondefaults appear much more frequently than defaults.

| Model population           | Defaults | Nondefaults | Default rate |             Approximate ratio |
| -------------------------- | -------: | ----------: | -----------: | ----------------------------: |
| 2015–2016 development      |      583 |      81,804 |       0.708% | 1 default per 140 nondefaults |
| 2017 final test            |      397 |      41,667 |       0.944% | 1 default per 105 nondefaults |
| 2006 historical comparison |      967 |      36,588 |       2.575% |  1 default per 38 nondefaults |

Because the outcome is imbalanced, ordinary accuracy can be misleading. A model could classify nearly every loan as a nondefault and appear highly accurate while failing to identify most actual defaults. For this reason, the project evaluates probability quality and risk ranking using ROC AUC, average precision, Brier score, lift, and default capture rather than relying only on accuracy.

Raw source files and loan-level processed outputs are not included in the repository because of their size and source-access requirements.


## Analytical Workflow

### 1. Origination Data Audit

The origination audit validates:

* expected files and schemas;
* row and column counts;
* unique loan identifiers;
* duplicate and missing identifiers;
* missingness by vintage;
* numeric ranges and extreme ratios;
* disclosed categorical values;
* output-file reconciliation.

The audit produced 200,000 unique loan records with no duplicate or missing loan identifiers.

Three unusually high LTV or CLTV records were reviewed and determined to be internally consistent HARP-related loans. They were retained and flagged rather than automatically removed.

### 2. Monthly Performance Audit

The performance audit processes the large raw files in memory-conscious chunks and validates:

* referential integrity against the origination population;
* unique loan-month combinations;
* reporting-period formats and sequence;
* delinquency, modification, and zero-balance codes;
* balance and maturity fields;
* population totals by vintage.

All 12,850,534 performance records reconciled to the origination population, with zero unmatched loan identifiers, duplicate loan-months, or invalid reporting-period formats.

Testing identified one isolated remaining-maturity discrepancy affecting 37 records and one modified record that could not be independently recalculated because the modified maturity date was unavailable. Both items were quantified and documented in the exception log.

### 3. Outcome Construction and Testing

The project constructs 12-, 24-, and 36-month outcomes while distinguishing between:

* observed defaults;
* eligible nondefaults with complete observation windows;
* censored loans without complete observation windows.

The 24-month outcome was independently reperformed for a stratified sample of 120 loans covering 6,791 raw performance records.

All 120 independent checks agreed with the constructed outcomes, eligibility indicators, observed-month calculations, and censoring classifications.

### 4. Feature Engineering and Leakage Controls

Only information available at origination is eligible for model use.

Controls include:

* time-based development and test separation;
* exclusion of post-origination performance fields;
* prohibited-predictor testing;
* feature-availability review;
* categorical-level stability testing;
* development-fitted imputation and encoding.

Five unsuitable source fields were excluded because of extreme missingness or constant values.

The final model uses:

* 24 predictors
* 15 numeric predictors
* 9 categorical predictors
* 84 transformed features

Key transformations include logged original unpaid principal balance, capped LTV and CLTV measures, high-ratio indicators, and source-missingness indicators.

### 5. Baseline PD Modeling

The baseline model is an unweighted, L2-regularized logistic regression selected for transparency and interpretability.

Population design:

| Population             |   Vintage | Eligible loans | Defaults | Default rate |
| ---------------------- | --------: | -------------: | -------: | -----------: |
| Model development      | 2015–2016 |         82,387 |      583 |       0.708% |
| Independent final test |      2017 |         42,064 |      397 |       0.944% |
| Historical comparison  |      2006 |         37,555 |      967 |       2.575% |

The 2017 and 2006 populations were not used to fit the final model, and identifier testing confirmed no population overlap.

## Model Results

| Population                 | Mean predicted PD | ROC AUC | Average precision | Brier score | Top-decile lift | Top-decile default capture |
| -------------------------- | ----------------: | ------: | ----------------: | ----------: | --------------: | -------------------------: |
| 2015–2016 development      |            0.714% |   0.857 |             0.048 |      0.0069 |            5.64 |                      56.4% |
| 2017 final test            |            0.879% |   0.840 |             0.059 |      0.0091 |            5.04 |                      50.4% |
| 2006 historical comparison |            3.255% |   0.802 |             0.092 |      0.0253 |            3.82 |                      38.2% |

The final 2017 test demonstrates:

* strong discrimination with ROC AUC of 0.840;
* limited deterioration from the development ROC AUC of 0.857;
* mild aggregate underprediction relative to the 0.944% observed default rate;
* concentration of approximately 50.4% of defaults in the highest-risk 10% of loans;
* generally consistent risk ordering across prediction deciles.

## Historical Comparison Limitation

The 2006 historical population contains channel value `T`, which was absent from the 2015–2016 development population.

This affects 22,031 of 37,555 eligible 2006 loans, or 58.66% of the historical comparison population.

The configured encoder safely processes unknown categories, but the model cannot estimate a separate effect for this channel level. Therefore, the 2006 results are presented only as historical sensitivity analysis and not as an equivalent substitute for the primary 2017 independent test.

The limitation is documented as `L-03` in the exception log.

## Enterprise Independent Testing Deliverables

The project includes a documented testing framework with:

* end-to-end process narrative;
* EIT test plan;
* risk-control matrix;
* exception log;
* origination testing workpaper;
* performance testing workpaper;
* outcome testing workpaper;
* feature-leakage workpaper;
* PD model testing workpaper;
* independently reproducible summary outputs.

The testing work demonstrates population reconciliation, test execution, evidence retention, exception identification, result communication, and documented closure decisions.

## Repository Structure

```text
freddie-mac-credit-risk/
├── data/
│   └── processed/
│       ├── feature_manifest.csv
│       ├── model_performance_summary.csv
│       ├── model_decile_analysis.csv
│       ├── model_coefficients.csv
│       └── unknown_category_audit.csv
├── docs/
│   └── eit/
│       ├── 01_process_narrative.md
│       ├── 02_eit_test_plan.md
│       ├── 03_origination_testing_workpaper.md
│       ├── 04_performance_testing_workpaper.md
│       ├── 05_outcome_testing_workpaper.md
│       ├── 06_feature_leakage_workpaper.md
│       ├── 07_pd_model_testing_workpaper.md
│       ├── risk_control_matrix.xlsx
│       └── exception_log.xlsx
├── notebooks/
│   ├── 01_data_audit.ipynb
│   ├── 02_performance_audit.ipynb
│   ├── 03_outcome_construction.ipynb
│   ├── 04_feature_engineering.ipynb
│   └── 05_pd_modeling.ipynb
├── requirements.txt
└── README.md
```

## Running the Project

Create and activate a virtual environment, then install the required packages:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

After obtaining the required Freddie Mac sample files, place them in the appropriate `data/raw` location and run the notebooks sequentially:

1. `01_data_audit.ipynb`
2. `02_performance_audit.ipynb`
3. `03_outcome_construction.ipynb`
4. `04_feature_engineering.ipynb`
5. `05_pd_modeling.ipynb`

The raw performance audits use chunked processing to reduce memory requirements.

## Tools and Skills Demonstrated

* Python
* pandas and NumPy
* scikit-learn
* Parquet data processing
* Jupyter Notebook
* Excel
* Git and GitHub
* data-quality testing
* business-process and control analysis
* population reconciliation
* exception management
* feature engineering
* leakage prevention
* probability-of-default modeling
* model performance and stability analysis
* technical documentation and workpaper preparation

## Limitations

This is an educational portfolio project based on Freddie Mac sample data.

The model is not presented as a production underwriting or credit-decision system. Production use would require additional governance, calibration, economic-cycle testing, fairness analysis, stability monitoring, challenger modeling, and formal model-risk validation.
