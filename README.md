# Credit Risk Analysis & Default Prediction

**Author:** Guilherme Almeida

## Business problem
A lender loses money in two ways: approving a client who later defaults, and rejecting a good client who would have paid. This project estimates each applicant's **probability of default** so the credit team can approve, review or reject loans based on risk, and measures what that is worth. About 22% of the loans in the dataset defaulted.

Dataset: [Credit Risk Dataset](https://www.kaggle.com/datasets/laotse/credit-risk-dataset) (Kaggle), 32,581 loan records, 32,409 after cleaning.

## Two versions of the project

| | Original (repository root) | v2 (`v2/`) |
|---|---|---|
| Notebook | `Credit_risk_analysis.ipynb` | `v2/Credit_risk_analysis_v2.ipynb` |
| Models | k-NN and SVM | k-NN, Logistic Regression, Random Forest, XGBoost |
| Preprocessing | Median imputation and one-hot encoding on the full dataset before the split; scaler fitted on the training set | Leakage-free pipelines: imputation, scaling and encoding fitted on training data only |
| Evaluation | One train/test split | Stratified 5-fold cross-validation, tuned decision threshold, bootstrap confidence intervals |
| Extras | Power BI dashboard | Hyperparameter tuning, probability calibration, cost analysis, SHAP, robustness tests |

The original version is kept unchanged. Its best model was **k-NN (k=5)**, which caught 61% of defaulters at an overall accuracy of 89%.

## Results of v2
All numbers below come from the saved outputs of `v2/Credit_risk_analysis_v2.ipynb`, on the 30% held-out test set unless noted.

| Model | ROC-AUC | PR-AUC |
|---|---|---|
| k-NN | 0.865 | 0.735 |
| Logistic Regression | 0.870 | 0.708 |
| Random Forest | 0.933 | 0.886 |
| Random Forest (tuned) | 0.929 | 0.878 |
| **XGBoost (tuned)** | **0.950** | **0.907** |

- **The tuned XGBoost catches 86.1% of defaulters** (95% bootstrap CI 84.7% to 87.6%) with 66.8% precision, at a decision threshold chosen on out-of-fold training predictions. At the default 0.5 cutoff, k-NN catches 61.1% of defaulters.
- **Interactions drive the gain.** Tree models beat the linear and distance-based ones because default risk depends on combinations of variables (for example loan grade together with loan-to-income).
- **Probabilities are calibrated.** The raw model overstated risk (mean predicted default probability 29.2% against 21.9% observed). Isotonic calibration reduced the Brier score from 0.064 to 0.052 without hurting ranking (ROC-AUC 0.949).
- **Estimated value.** Under illustrative cost assumptions (60% loss given default, one year of forgone interest), using the model as a credit filter cuts the simulated loss from USD 13.97 million to USD 2.67 million, an **80.9% saving** versus approving every loan (95% CI 79.1% to 82.6%). A naive rule, "reject loan grades D to G", saves 37.3%.
- **The model is not just the lender's own score.** Without `loan_grade` and `loan_int_rate` the ROC-AUC falls from 0.950 to 0.904, so part of the signal is the lender's earlier pricing, but a substantial independent signal remains. Removing `person_age` costs almost nothing (ROC-AUC 0.948).
- **Risk bands separate clients well.** Observed default rate of **2.1%** (Low, 17,364 clients), **11.4%** (Medium, 7,302) and **76.1%** (High, 7,743), with the High band holding about 82% of the expected loss (expected loss = PD x LGD x EAD).
- **Where the risk is.** Loans above 30% of income default 70.4% of the time, against 11.6% below 10%; a prior default on file raises the rate from 18.4% to 37.9%.

![Risk drivers](v2/credit_risk_features.png)
![SHAP beeswarm](v2/credit_risk_shap_beeswarm.png)

### Limitations
- The cost figures are illustrative and in US dollars; they are not in the data. The notebook includes a sensitivity analysis over them.
- The data is a snapshot (no time dimension) and only approved loans have outcomes.
- Variables such as age may be restricted in real credit decisions (for example under GDPR Article 22 and anti-discrimination rules), so a compliance review would be needed before any real use.

## Interactive dashboard (original version)

**[Click here to interact with the Power BI Dashboard](https://app.powerbi.com/view?r=eyJrIjoiNzMyY2ZmYWItYWVjMy00ZDdmLWJiMjQtOTg4YjMxMDBiMjkwIiwidCI6ImRlNTM3NmEzLTdhOTEtNGM1NS1hOGQ5LTI0YjhkMTVlNWViMSJ9)**

*(If you are viewing this on mobile or without Power BI access, check the static preview below.)*

![Credit Risk Dashboard](CreditRisk_analysis.jpg)

## How to run v2
1. Install the dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. **Dataset.** The notebook downloads it from Kaggle on the first run. Copy `v2/Kaggle_token.env.example` to `v2/Kaggle_token.env` and put your own token in it (never commit this file). Alternatively, download the dataset manually and place `credit_risk_dataset.csv` in `v2/`; the download is then skipped.
3. Open `v2/Credit_risk_analysis_v2.ipynb`, with `v2/` as the working directory, and run all cells. The first run takes a few minutes. The slow steps are cached in `v2/cache/`, so later runs are much faster; delete that folder to force a full re-run.
