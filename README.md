# Freddie Mac Credit Risk: PD Modeling and Independent Testing

An end-to-end credit-risk analytics project using Freddie Mac Single-Family Loan-Level sample data to construct loan outcomes, develop an explainable probability-of-default model, and demonstrate Enterprise Independent Testing practices.

The project combines data validation, process and control analysis, exception management, reproducible testing, feature-leakage controls, model evaluation, and documented workpapers.

## Project Objective

The project addresses two related questions:

1. Can origination information identify loans with elevated 24-month default risk?
2. Can the underlying data, outcomes, controls, and reported model results be independently tested and reconciled?

A default is defined as a loan reaching delinquency status `03` or greater, or real-estate-owned status `RA`, within the first 24 monthly reporting periods following the original first payment date.

## Data Scope

The analysis uses Freddie Mac Single-Family Loan-Level sample files for four vintages:

* 2006: historical comparison
* 2015: initial model training
* 2016: internal validation
* 2017: independent later-vintage testing

The validated population contains:

* 200,000 unique originated loans
* 12,850,534 monthly performance records
* 162,006 loans eligible for 24-month model evaluation
* 33 cleaned origination fields
* 35 monthly performance fields

Raw source files are not included in the repository because of their size and source-access requirements.

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
