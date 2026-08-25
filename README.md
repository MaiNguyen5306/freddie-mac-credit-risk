# Freddie Mac Credit Risk: Probability of Default Modeling and Independent Testing

**Freddie Mac**, formally known as the Federal Home Loan Mortgage Corporation, is a U.S. government-sponsored enterprise that purchases mortgages from lenders and helps support the availability of mortgage funding. Borrowers generally receive their loans from banks or other lenders rather than directly from Freddie Mac.

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

<details>
<summary><strong>Compare the 12-, 24-, and 36-month outcomes</strong></summary>

| Outcome period | Eligible loans | Censored loans | Observed defaults |
|---|---:|---:|---:|
| 12 months | 179,108 | 20,892 | 579 |
| 24 months | 162,006 | 37,994 | 1,947 |
| 36 months | 138,440 | 61,560 | 5,133 |

The 12-month period retains more eligible loans but captures fewer defaults. The 36-month period captures more defaults but excludes more loans because they lack a complete observation period. The 24-month period provides a middle ground between event capture and population retention.

These results were constructed and validated in [`03_outcome_construction.ipynb`](notebooks/03_outcome_construction.ipynb).

</details>

### Class Imbalance

In classification, a **class** is one possible outcome category. In this project, class `1` represents default and class `0` represents nondefault. The dataset is **class imbalanced** because nondefaults appear much more frequently than defaults.

| Model population           | Defaults | Nondefaults | Default rate |             Approximate ratio |
| -------------------------- | -------: | ----------: | -----------: | ----------------------------: |
| 2015–2016 development      |      583 |      81,804 |       0.708% | 1 default per 140 nondefaults |
| 2017 final test            |      397 |      41,667 |       0.944% | 1 default per 105 nondefaults |
| 2006 historical comparison |      967 |      36,588 |       2.575% |  1 default per 38 nondefaults |

<details>
<summary><strong>How were the default rates and ratios calculated?</strong></summary>

The default rate is calculated as:

`Defaults ÷ (Defaults + Nondefaults) × 100`

The approximate ratio is calculated as:

`Nondefaults ÷ Defaults`

Calculations by population:

- **2015–2016 development:** `81,804 ÷ 583 = 140.32`, or approximately 1 default per 140 nondefaults.
- **2017 final test:** `41,667 ÷ 397 = 104.95`, or approximately 1 default per 105 nondefaults.
- **2006 historical comparison:** `36,588 ÷ 967 = 37.84`, or approximately 1 default per 38 nondefaults.

The ratios are rounded to the nearest whole number for easier interpretation.

</details>

Because the outcome is imbalanced, ordinary accuracy can be misleading. A model could classify nearly every loan as a nondefault and appear highly accurate while failing to identify most actual defaults. For this reason, the project evaluates probability quality and risk ranking using ROC AUC, average precision, Brier score, lift, and default capture rather than relying only on accuracy.

<details>
<summary><strong>What do the model-evaluation measures mean?</strong></summary>

| Measure | Question answered | Interpretation |
|---|---|---|
| **ROC AUC** | Does the model rank defaulted loans above nondefaulted loans? | `0.50` represents random ranking and `1.00` represents perfect ranking. Higher is better. |
| **Average precision** | Can the model find rare defaults without flagging too many nondefaults? | Summarizes precision and recall across multiple risk cutoffs. Higher is better. |
| **Brier score** | Are the predicted probabilities close to the actual outcomes? | Measures probability error. `0` is perfect, so lower is better. |
| **Lift** | How much riskier is the selected high-risk group than the full population? | A lift above `1.0` indicates that the model concentrates more defaults than random selection. |
| **Default capture** | What share of all defaults appears in the selected high-risk group? | Higher capture means more defaults are identified within the selected group. |

No single measure provides a complete evaluation. The project uses these measures together to examine risk ranking, rare-event detection, probability quality, and concentration of defaults.

</details>

Raw source files and loan-level processed outputs are not included in the repository because of their size and source-access requirements.


## Analytical Workflow

The project follows the loan data from its original files to a tested probability-of-default model. Each stage answers a different question and produces evidence that another person can review.

### 1. Origination Data Audit

**Purpose:** Confirm that the information recorded when each mortgage began is complete, consistently structured, and suitable for later analysis.

Origination information includes characteristics such as credit score, original loan balance, interest rate, loan term, property information, and borrower information. Because these fields later become potential model inputs, errors at this stage could affect every subsequent result.

The audit checked:

- whether the expected files, columns, row counts, and field counts were present;
- whether loan identifiers were unique, complete, and free of duplicates;
- how missing values differed across vintages;
- whether numeric ranges, extreme values, and categorical codes were reasonable;
- whether the cleaned output reconciled to the original source population.

The audit confirmed 200,000 unique originated loans with no missing or duplicated loan identifiers.

Three loans had unusually high loan-to-value or combined loan-to-value ratios. These records were investigated rather than automatically deleted. The values were internally consistent with loans associated with the **Home Affordable Refinance Program (HARP)**, a program that allowed some borrowers with limited or negative home equity to refinance. The records were therefore retained and marked for review.

Supporting evidence:

* [`01_data_audit.ipynb`](notebooks/01_data_audit.ipynb) — data cleaning, profiling, and validation code;
* [`03_origination_testing_workpaper.md`](docs/eit/03_origination_testing_workpaper.md) — testing procedures, evidence, and conclusions.

<details>
<summary><strong>Key terms used in this audit</strong></summary>

* **Schema:** The expected structure of a dataset, including its column names, order, and data types.
* **Loan identifier:** A value used to distinguish one loan from every other loan.
* **Categorical field:** A field containing named groups or codes, such as property type, occupancy status, or loan purpose.
* **Loan-to-value (LTV):** Original loan balance divided by the property value. A higher ratio means the loan amount is large relative to the property value.
* **Combined loan-to-value (CLTV):** Total mortgage debt secured by the property divided by the property value.
* **Reconciliation:** Confirming that records and totals remain consistent between the source data and the cleaned output.

</details>


### 2. Monthly Performance Audit

**Purpose:** Confirm that the monthly records accurately track what happened to each loan after origination and connect back to the correct origination record.

Unlike the origination data, which contains one row per loan, the performance data contains one row for each month a loan was observed. A single loan can therefore have many monthly records.

Because the performance files contain more than 12.8 million records, they were processed in chunks of 100,000 rows. This allowed the full population to be tested without loading every record into the laptop’s memory at once.

The audit checked:

- whether every monthly record matched a loan in the origination population;
- whether each loan-month combination was unique and ordered correctly;
- whether reporting periods, delinquency statuses, modification flags, and loan-ending codes were valid;
- whether balance, loan-age, and remaining-maturity values were reasonable;
- whether record counts and loan counts reconciled across vintages.

All 12,850,534 monthly records matched the origination population. Testing found no missing or unmatched loan identifiers, duplicate loan-month records, invalid reporting-period formats, or ordering violations.

Testing identified two limited issues:

1. One loan contained 37 records where the reported remaining maturity differed from the independently calculated value.
2. One modified loan record could not be independently recalculated because the data did not provide the revised maturity date.

The affected records were quantified and documented in the exception log. They did not change the overall population conclusions or the primary default-outcome results.

Supporting evidence:

- [`02_performance_audit.ipynb`](notebooks/02_performance_audit.ipynb) — full-population monthly performance testing;
- [`04_performance_testing_workpaper.md`](docs/eit/04_performance_testing_workpaper.md) — testing procedures, exceptions, and conclusions;
- [`exception_log.xlsx`](docs/eit/exception_log.xlsx) — documented issues and closure evidence.

<details>
<summary><strong>Key terms used in this audit</strong></summary>

- **Loan-month:** One monthly observation for one loan.
- **Reporting period:** The year and month represented by a monthly performance record.
- **Referential integrity:** Confirmation that every performance loan identifier exists in the origination population.
- **Modification flag:** An indicator showing whether the original loan terms were changed after origination.
- **Loan-ending code:** A code showing why a loan stopped appearing in the monthly data, such as payoff, property sale, or foreclosure-related resolution.
- **Remaining maturity:** The estimated number of months left before the mortgage reaches its scheduled maturity date.

</details>

### 3. Outcome Construction and Testing

**Purpose:** Convert the monthly performance history into a clear outcome showing whether each loan defaulted within 12, 24, or 36 months.

The source data does not provide one ready-to-use field that directly answers whether a loan defaulted within a selected period. The outcome therefore had to be calculated by reviewing each loan’s monthly delinquency and REO history.

For each outcome period, loans were classified as:

- **Default:** The loan reached delinquency status `03` or greater, or entered REO status, during the selected period.
- **Eligible nondefault:** The loan completed the full observation period without an identified default.
- **Censored:** The loan did not have an observed default but also did not have enough monthly history to confirm a complete nondefault period.

Censored loans were excluded from the corresponding model population because their final outcome could not be confirmed. For example, a loan observed for only 15 months cannot be confidently classified as a 24-month nondefault.

The project created 12-, 24-, and 36-month outcomes for all 200,000 loans. The 24-month outcome was selected as the primary model target, resulting in 162,006 eligible loans.

### Independent Outcome Check

A stratified sample of 120 loans was selected across the four vintages. Their outcomes were independently recalculated from 6,791 raw monthly performance records and compared with the constructed outcome file.

All 120 sampled loans matched for:

- number of observed months;
- 24-month default or nondefault outcome;
- whether the loan had enough information to be included in the 24-month model population;
- whether an incomplete loan was correctly classified as censored.

This provided evidence that the outcome-construction rules were applied consistently.

Supporting evidence:

- [`03_outcome_construction.ipynb`](notebooks/03_outcome_construction.ipynb) — outcome construction, reconciliation, and independent sample testing;
- [`05_outcome_testing_workpaper.md`](docs/eit/05_outcome_testing_workpaper.md) — testing procedures, results, and conclusions.

<details>
<summary><strong>Key terms used in outcome construction</strong></summary>

- **Outcome:** The result the model is intended to predict.
- **Observation period:** The amount of time during which a loan’s performance is evaluated.
- **Eligible loan:** A loan with enough information to assign a confirmed outcome.
- **Censored loan:** A loan whose complete nondefault outcome cannot be confirmed because its observation history ends too early.
- **Stratified sample:** A sample selected from multiple defined groups—in this case, the different loan vintages.
- **Independent recalculation:** Repeating a calculation separately from the original output and comparing the two results.

</details>

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
