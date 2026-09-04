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

This stage prepared the cleaned origination data for modeling. A **feature**, also called a **predictor**, is a piece of information used by the model to estimate default risk, such as credit score, debt-to-income ratio, or interest rate.

The most important control was preventing **data leakage**. Leakage occurs when a model is given information that would not have been available at the time the prediction was supposed to be made. For example, using a borrower’s later delinquency status to predict default at origination would make the model appear unrealistically accurate.

The project applied the following controls:

- limited predictors to information available when the mortgage was originated;
- excluded monthly performance and other post-origination information;
- separated development and testing populations by vintage;
- learned missing-value replacements and category encodings from the development population only;
- tested for prohibited predictors and compared missing values and category levels across populations.

Five source fields were excluded:

| Excluded field | Reason |
| --- | --- |
| `property_valuation_method` | 99.70% missing and only one observed value |
| `special_eligibility_program` | 96.52% missing with unstable coverage |
| `amortization_type` | Constant value |
| `interest_only_indicator` | Constant value |
| `prepayment_penalty_indicator` | Constant value |

Fields with almost no usable information or no variation cannot meaningfully help the model distinguish higher-risk loans from lower-risk loans.

The final model uses:

- 24 source predictors;
- 15 numeric predictors;
- 9 categorical predictors;
- 84 transformed features after preprocessing.

The preprocessing steps include:

- replacing missing numeric values with the development-population median;
- replacing missing categorical values with the most frequent development-population value;
- converting categorical values into model-readable indicator columns;
- applying a logarithmic transformation to original loan balance to reduce the influence of very large values;
- capping LTV and CLTV at 200 while separately flagging values above 100;
- creating indicators that identify whether selected source values were originally missing.

These transformations preserve the original business meaning of the data while making the inputs more suitable for logistic regression.

**Supporting evidence:**

- [`04_feature_engineering.ipynb`](notebooks/04_feature_engineering.ipynb)
- [`06_feature_leakage_workpaper.md`](docs/eit/06_feature_leakage_workpaper.md)
- [`feature_manifest.csv`](data/processed/feature_manifest.csv)

<details>
<summary><strong>Feature-engineering terminology</strong></summary>

- **Feature or predictor:** Information supplied to the model to help estimate default risk.
- **Feature engineering:** Preparing or transforming source information into model-ready inputs.
- **Data leakage:** Allowing information unavailable at prediction time to influence model development.
- **Imputation:** Replacing a missing value using a defined rule, such as the median.
- **One-hot encoding:** Converting categories into separate indicator columns that a model can process.
- **Log transformation:** Compressing the scale of a numeric field so extremely large values have less influence.
- **Capping:** Limiting a value at a selected threshold while retaining a separate indicator that the threshold was exceeded.

</details>

### 5. Baseline Probability-of-Default Modeling

A **probability-of-default (PD) model** estimates how likely each loan is to meet the project’s 24-month default definition. The output is a probability between 0 and 1. For example, a predicted PD of `0.02` means an estimated 2% probability of default—not that the loan will definitely default.

The project uses **logistic regression**, a classification method commonly used for outcomes with two possible classes. Here:

- `1` means the loan defaulted within the 24-month period;
- `0` means the eligible loan completed the period without defaulting.

Logistic regression was selected as the baseline model because its predictions and predictor relationships are easier to explain and independently review than those of many more complex machine-learning models. The model uses L2 regularization to reduce excessive reliance on individual predictors.

The modeling populations were separated by loan vintage:

| Population | Vintage | Eligible loans | Defaults | Observed default rate | Purpose |
| --- | ---: | ---: | ---: | ---: | --- |
| Model development | 2015–2016 | 82,387 | 583 | 0.708% | Fit the final model |
| Final test | 2017 | 42,064 | 397 | 0.944% | Evaluate performance on a later period |
| Historical comparison | 2006 | 37,555 | 967 | 2.575% | Examine performance in an older, higher-default period |

The 2017 and 2006 populations were not used to fit or adjust the final model. Loan-identifier testing also confirmed that the three populations did not overlap.

Because defaults were uncommon, the model was fitted without automatically forcing the two outcome classes to have equal weight. Its performance was evaluated using measures designed to assess probability quality and risk ranking on class-imbalanced data.

**Supporting evidence:**

- [`05_pd_modeling.ipynb`](notebooks/05_pd_modeling.ipynb)
- [`07_pd_model_testing_workpaper.md`](docs/eit/07_pd_model_testing_workpaper.md)
- [`model_performance_summary.csv`](data/processed/model_performance_summary.csv)
- [`model_coefficients.csv`](data/processed/model_coefficients.csv)

<details>
<summary><strong>Modeling terminology</strong></summary>

- **Probability of default (PD):** The estimated chance that a loan will meet the defined default condition during a specified period.
- **Logistic regression:** A statistical classification method that estimates the probability of a two-class outcome.
- **Classification:** Assigning or estimating an outcome with defined classes, such as default and nondefault.
- **Baseline model:** A relatively simple and explainable model used as the project’s starting benchmark.
- **L2 regularization:** A method that discourages extremely large model coefficients and can reduce overfitting.
- **Model fitting:** Learning relationships between predictors and outcomes from the development data.
- **Overfitting:** Learning the development data too specifically, resulting in weaker performance on other data.
- **Unweighted model:** A model fitted without deliberately giving defaulted loans greater importance than nondefaulted loans.

</details>

## Model Results

The model was evaluated from several perspectives because no single metric fully describes credit-risk performance:

- **risk ranking:** whether loans with higher predicted PDs defaulted more frequently;
- **probability accuracy:** whether predicted probabilities were close to observed outcomes;
- **default concentration:** whether the highest-risk group contained a meaningful share of actual defaults;
- **stability:** whether performance remained reasonably consistent outside the development population.

| Population | Observed default rate | Mean predicted PD | ROC AUC | Average precision | Brier score | Top-decile lift | Top-decile default capture |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 2015–2016 development | 0.708% | 0.714% | 0.857 | 0.048 | 0.0069 | 5.64 | 56.4% |
| 2017 final test | 0.944% | 0.879% | 0.840 | 0.059 | 0.0091 | 5.04 | 50.4% |
| 2006 historical comparison | 2.575% | 3.255% | 0.802 | 0.092 | 0.0253 | 3.82 | 38.2% |

### Interpretation of the 2017 Final Test

The 2017 population provides the main test of how the completed model performs on later loans that were not used for model fitting.

- **Risk ranking remained strong:** The ROC AUC was `0.840`, compared with `0.857` in development. This means the model generally assigned higher PDs to loans that defaulted than to loans that did not.
- **The change from development was limited:** The ROC AUC decreased by `0.017`, suggesting that ranking performance remained reasonably stable on the later population.
- **Predicted and observed portfolio risk were close:** The mean predicted PD was `0.879%`, while the observed default rate was `0.944%`. The model therefore slightly underestimated total default risk by `0.065` percentage points.
- **High-risk loans were concentrated near the top:** The highest-risk 10% of loans contained approximately `50.4%` of all 2017 defaults.
- **Top-decile lift was `5.04`:** A loan in the highest-risk 10% was approximately five times as likely to default as a loan selected from the overall 2017 population.
- **Risk ordering was generally consistent:** Observed default rates increased as the predicted-risk groups increased.

Overall, the model provides useful risk ranking on the 2017 population. However, its probabilities should not be treated as production-ready estimates without additional calibration, monitoring, and model-risk validation.

**Supporting evidence:**

- [`05_pd_modeling.ipynb`](notebooks/05_pd_modeling.ipynb)
- [`07_pd_model_testing_workpaper.md`](docs/eit/07_pd_model_testing_workpaper.md)
- [`model_performance_summary.csv`](data/processed/model_performance_summary.csv)
- [`model_decile_analysis.csv`](data/processed/model_decile_analysis.csv)

<details>
<summary><strong>How to read the model metrics</strong></summary>

- **Observed default rate:** The percentage of eligible loans that actually defaulted.
- **Mean predicted PD:** The average default probability assigned by the model.
- **ROC AUC:** Measures risk-ranking ability. A value of `0.50` represents random ranking, while `1.00` represents perfect ranking.
- **Average precision:** Summarizes how effectively the model identifies defaults across risk thresholds. It is especially useful when defaults are uncommon; higher is better.
- **Brier score:** The average squared difference between predicted probabilities and actual `0` or `1` outcomes. Lower is better.
- **Top-decile lift:** Compares the default rate in the highest-risk 10% with the default rate of the entire population.
- **Top-decile default capture:** The percentage of all defaults found within the highest-risk 10% of loans.
- **Calibration:** The degree to which predicted probabilities agree with observed default rates.

</details>

## Historical Comparison and Limitation

The 2006 loans were used to explore how the final model behaves on an older, higher-default population. They were not used to fit or adjust the model.

The model achieved a ROC AUC of `0.802` on this population, indicating that it still provided useful risk ranking. However, the comparison has an important data limitation.

The 2006 population contains a value of `T` in the `channel` field. This value did not appear in the 2015–2016 development population used to fit the model.

The issue affects:

- 22,031 of the 37,555 eligible 2006 loans;
- approximately 58.66% of the historical comparison population.

The model’s categorical encoder can safely process an unfamiliar category without producing an error. However, because `T` was absent during model development, the model could not learn a separate relationship between this channel value and default risk.

For this reason, the 2006 results should be interpreted as a **historical sensitivity analysis** rather than as an equivalent final test. The 2017 population remains the project’s primary test because it did not contain unknown category values.

This limitation is recorded as `L-03` in the exception log.

**Supporting evidence:**

- [`05_pd_modeling.ipynb`](notebooks/05_pd_modeling.ipynb)
- [`07_pd_model_testing_workpaper.md`](docs/eit/07_pd_model_testing_workpaper.md)
- [`unknown_category_audit.csv`](data/processed/unknown_category_audit.csv)
- [`exception_log.xlsx`](docs/eit/exception_log.xlsx)

<details>
<summary><strong>What is a historical sensitivity analysis?</strong></summary>

A historical sensitivity analysis examines how a model behaves on data from a substantially different period. It can reveal changes in borrower characteristics, category values, default rates, or economic conditions.

It is informative, but it is not necessarily a fair direct comparison with the development or final-test populations. In this project, the unknown `channel` value means the 2006 results should be interpreted with additional caution.

</details>

## Enterprise Independent Testing Deliverables

In addition to developing the model, the project documents how the data, calculations, controls, and reported results were tested. This mirrors the structured evidence and workpaper practices used in Enterprise Independent Testing (EIT).

The deliverables include:

- an end-to-end process narrative explaining how data moves from raw files to reported results;
- an EIT test plan defining the testing objectives, scope, procedures, and expected evidence;
- a risk-control matrix connecting process risks with controls and testing procedures;
- an exception log recording identified issues, their impact, recommendations, and closure decisions;
- separate workpapers documenting origination, performance, outcome, feature-leakage, and model testing;
- reproducible notebooks and summary files supporting the reported conclusions.

Together, these deliverables demonstrate:

- **population reconciliation:** confirming that records remain complete as data moves through the workflow;
- **control testing:** checking whether procedures designed to prevent errors operated as expected;
- **independent reperformance:** repeating selected calculations separately and comparing the results;
- **exception management:** identifying, quantifying, documenting, and resolving testing issues;
- **evidence retention:** preserving the files and results supporting each conclusion;
- **results communication:** clearly explaining findings, limitations, and recommended actions.

All logged exceptions were reviewed and closed with documented explanations. A closed status means that the item was investigated and its treatment was documented; it does not necessarily mean that the underlying source-data condition was corrected.

This is an educational simulation of EIT work. It should not be interpreted as formal independent validation performed by a bank’s internal testing or model-risk department.

**Testing documents:**

- [`01_process_narrative.md`](docs/eit/01_process_narrative.md)
- [`02_eit_test_plan.md`](docs/eit/02_eit_test_plan.md)
- [`risk_control_matrix.xlsx`](docs/eit/risk_control_matrix.xlsx)
- [`exception_log.xlsx`](docs/eit/exception_log.xlsx)
- [`03_origination_testing_workpaper.md`](docs/eit/03_origination_testing_workpaper.md)
- [`04_performance_testing_workpaper.md`](docs/eit/04_performance_testing_workpaper.md)
- [`05_outcome_testing_workpaper.md`](docs/eit/05_outcome_testing_workpaper.md)
- [`06_feature_leakage_workpaper.md`](docs/eit/06_feature_leakage_workpaper.md)
- [`07_pd_model_testing_workpaper.md`](docs/eit/07_pd_model_testing_workpaper.md)

## Repository Structure

The repository is organized so that the analytical code, testing documentation, and shareable results can be reviewed separately.

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

The notebooks are designed to run sequentially because each stage uses files created by the previous stage.

### 1. Create the Python environment

From the repository root, create and activate a virtual environment and install the required packages:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## Tools and Skills Demonstrated

This project combines data analytics, statistical modeling, and business-control testing.

### Technical tools

- Python
- pandas and NumPy
- scikit-learn
- Jupyter Notebook
- Parquet and chunked data processing
- Excel
- Git and GitHub

### Data and analytical skills

- exploratory data analysis
- data cleaning and validation
- missing-value and outlier analysis
- feature engineering
- binary-outcome construction
- class-imbalance analysis
- logistic regression
- probability-of-default modeling
- model performance and stability analysis
- interpretation of model coefficients and risk deciles

### Business and testing skills

- business-process analysis
- risk and control identification
- population reconciliation
- testing-procedure design and execution
- independent reperformance
- exception identification and management
- evidence documentation
- workpaper preparation
- communication of results and limitations

## Limitations

This is an educational portfolio project based on Freddie Mac sample data.

The model is not presented as a production underwriting or credit-decision system. Production use would require additional governance, calibration, economic-cycle testing, fairness analysis, stability monitoring, challenger modeling, and formal model-risk validation.
