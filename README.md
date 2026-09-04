# Freddie Mac Credit Risk: PD Modeling & Independent Testing

## Overview

This project develops a **probability-of-default (PD) model** predicting 24-month mortgage defaults using Freddie Mac Single-Family Loan-Level data. Beyond standard predictive modeling, the project simulates an **Enterprise Independent Testing (EIT)** workflow through structured testing documentation, a risk-control matrix, exception tracking, and independent reperformance checks.

**Tech Stack & Skills:** Python (pandas, NumPy, scikit-learn), Logistic Regression, EIT and Model Risk Management, Feature Engineering, Imbalanced Classification, Parquet and Chunked Data Processing, Git.

For the complete methodology, testing evidence, terminology, and limitations, see the [Detailed Project Documentation](docs/DETAILED_README.md).

## 📊 Key Outcomes & Model Performance

- **Scale:** Audited and processed **12.8M+ monthly performance records** covering 200,000 unique loans from the 2006, 2015, 2016, and 2017 vintages. Chunked processing prevented memory overflow.
- **Predictive Performance:** The model achieved an ROC AUC of **0.840**, demonstrating strong ability to rank defaulted loans above nondefaulted loans in the unseen 2017 test population.
- **Risk Concentration:** The highest-risk 10% of loans contained **50.4% of all observed defaults** and defaulted at approximately **five times the overall rate**, despite defaults representing less than 1% of the population.
- **Testing Documentation:** Created EIT-style test plans, workpapers, an exception log, and a risk-control matrix aligned with structured bank testing and model-governance practices.

## 🛠 Analytical & EIT Workflow

1. **Data Audit and Reconciliation:** Verified schemas, unique identifiers, categorical values, numeric ranges, and population totals across origination and monthly performance records. Investigated and documented exceptions, including HARP-related LTV anomalies.

2. **Outcome Construction:** Created 12-, 24-, and 36-month outcomes while distinguishing confirmed defaults, eligible nondefaults, and censored loans. The primary 24-month target identifies loans reaching delinquency status `03` or greater or entering REO status.

3.  **Feature Engineering and Leakage Controls:** Transformed 24 original loan characteristics into 84 numeric model inputs through missing-value handling, categorical encoding, and risk-related transformations. All preprocessing rules were learned exclusively from the 2015–2016 development population to prevent information from the test data influencing the model.

4. **Baseline Modeling:** Trained an unweighted, L2-regularized logistic regression model. Performance was evaluated using ROC AUC, average precision, Brier score, lift, and default capture.

5. **Historical Sensitivity Testing:** Evaluated the completed model on the 2006 vintage, an older population with a substantially higher observed default rate of **2.57%**.

## Model Results

| Population | Purpose | Default rate | ROC AUC | Highest-risk 10% captured |
|---|---|---:|---:|---:|
| 2015–2016 | Model development | 0.71% | 0.857 | 56.4% of defaults |
| 2017 | Unseen final test | 0.94% | 0.840 | 50.4% of defaults |
| 2006 | Historical comparison | 2.57% | 0.802 | 38.2% of defaults |

### What the Results Mean

- **Strong performance on unseen data:** On the 2017 loans, the model had approximately an **84% chance of ranking a defaulted loan as riskier than a nondefaulted loan**.

- **Useful risk concentration:** The model’s highest-risk 10% of 2017 loans contained **50.4% of all observed defaults**. In other words, reviewing only one-tenth of the loans would identify approximately half of the defaults.

- **Limited decline outside development:** ROC AUC decreased only from `0.857` during development to `0.840` on the unseen 2017 population, indicating that the model’s risk-ranking ability remained reasonably consistent.

- **More difficult historical population:** The model’s ROC AUC decreased to `0.802` on the higher-default 2006 population, but it continued to provide useful risk ranking under substantially different conditions.

<details>
<summary><strong>What do these measures mean?</strong></summary>

- **Default rate:** The percentage of eligible loans that actually defaulted within 24 months.
- **ROC AUC:** Measures how well the model ranks defaulted loans above nondefaulted loans. `0.50` represents random ranking and `1.00` represents perfect ranking.
- **Highest-risk 10% captured:** The percentage of all defaults found among the 10% of loans assigned the highest predicted risk.

</details>

## 📂 Repository Structure

The repository separates analytical code, testing documentation, and shareable results.

```text
freddie-mac-credit-risk/
├── data/
│   └── processed/          # Feature manifests, coefficients, and performance summaries
├── docs/
│   ├── DETAILED_README.md  # Complete methodology, results, and limitations
│   └── eit/                # Test plans, workpapers, risk-control matrix, and exception log
├── notebooks/
│   ├── 01_data_audit.ipynb
│   ├── 02_performance_audit.ipynb
│   ├── 03_outcome_construction.ipynb
│   ├── 04_feature_engineering.ipynb
│   └── 05_pd_modeling.ipynb
├── requirements.txt
└── README.md
```

## Disclaimer

This is an educational portfolio project using Freddie Mac Single-Family Loan-Level sample data. The model is not a production underwriting or credit-decision system, and the testing work does not represent formal independent validation performed by Freddie Mac or a financial institution. Production use would require additional governance, calibration, economic-cycle testing, fairness analysis, stability monitoring, and formal model-risk validation.
