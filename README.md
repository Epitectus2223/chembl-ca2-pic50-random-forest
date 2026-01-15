
# Non-linear baseline for CAII pIC50 using Random Forest (ChEMBL205)

This repository continues my CAII (CHEMBL205) pIC50 modeling series:

- **Project 1 repo:** https://github.com/Epitectus2223/chembl-ca2-pic50-alogp
- **Project 2 repo:** https://github.com/Epitectus2223/chembl-ca2-pic50-multivariate-extended

- **Project 1:** AlogP-only OLS baseline (no useful linear signal)
- **Project 2 / 2.1:** Multivariate + extended linear models (modest gains; assumption issues)
- **Project 3 (this repo):** Random Forest regression to capture non-linear relationships and interactions

**Key takeaway:** A Random Forest baseline improves predictive performance over the best linear model by capturing non-linear structure without relying on strict parametric assumptions.

---

## Summary

### Train/Test split
- **80/20 split** with fixed random seed (`random_state=42`)
- Train: **434** observations | Test: **109** observations

---

## Model: Random Forest Regressor (baseline)

### Baseline configuration
A first, reasonable baseline model was trained with:
- `n_estimators = 300`
- `max_depth = None`
- `min_samples_leaf = 5`
- `random_state = 42`
- `n_jobs = -1`

### Features
Two setups are evaluated:

1) **Baseline RF:** `AlogP`, `MW`  
2) **Extended RF:** `AlogP`, `MW`, `log(MW)`, `AlogP²`

---

## Results (Predictive performance)

### RF baseline (AlogP + MW)
- **Train:** R² = **0.659**, RMSE = **0.740**, MAE = **0.566**
- **Test:**  R² = **0.414**, RMSE = **1.031**, MAE = **0.759**

The train–test gap suggests some overfitting, but the model still generalizes with meaningful predictive signal for a first non-linear baseline.

### RF with extended features (AlogP + MW + log(MW) + AlogP²)
- **Train:** R² = **0.668**, RMSE = **0.730**, MAE = **0.557**
- **Test:**  R² = **0.409**, RMSE = **1.036**, MAE = **0.761**

Performance is nearly identical to the baseline model (slightly worse on test), suggesting that the Random Forest already captures most of the available structure with the core descriptors.

---

## Interpretability: Feature importance (MDI, extended RF)

Random Forest feature importance is a relative measure of predictive contribution (not causal and without sign).

Observed importances:
- **log(MW): 0.3317**
- **MW: 0.3170**
- **AlogP: 0.2046**
- **AlogP²: 0.1467**

Interpretation:
- Size-related descriptors (**MW and log(MW)**) dominate the predictive signal.
- Lipophilicity (**AlogP**) contributes meaningfully but is secondary.
- The presence of **AlogP²** importance supports non-linear effects of lipophilicity.

---

## Comparison vs best linear model

From prior work, the best linear model achieved approximately **R² ≈ 0.27** (with heteroscedasticity / collinearity concerns).

In contrast, the Random Forest baseline achieves:
- **Test R² = 0.414**
- **Test RMSE = 1.031**
- **Test MAE  = 0.759**

**Conclusion:** Random Forest provides a substantial improvement in predictive performance for CAII pIC50, supporting the use of ML methods when linear assumptions are too restrictive for real QSAR settings.

---

## Scripts

These scripts assume a local file named `CAII_dataset_curado.csv` is available (not included in this repo).

- `01_prepare_train_test_split.py`
  - Loads the dataset, selects features, drops missing values
  - Performs an 80/20 train/test split and prints summary statistics
  - Saves split indices to `results/split_indices.json` for consistent reuse

- `02_train_rf_baseline.py`
  - Trains a Random Forest baseline using `[AlogP, MW]`
  - Reports RMSE, MAE, and R² on train and test
  - Saves metrics to `results/p3_rf_baseline_metrics.json`

- `03_train_rf_extended_features.py`
  - Creates engineered features `log(MW)` and `AlogP²`
  - Trains a Random Forest using `[AlogP, MW, log(MW), AlogP²]`
  - Reports RMSE, MAE, and R² on train and test
  - Prints feature importances and saves outputs to `results/p3_rf_extended_metrics.json`

---

## Data note

1. Export CAII bioactivity from **ChEMBL** (CHEMBL205).
2. Filter to **IC50 (nM)** with relation "=" and organism *Homo sapiens*.
3. Deduplicate and compute/select **AlogP** and **MW**.

Expected columns:
- `pIC50`, `AlogP`, `MW`, `log(MW)`, `AlogP²`

---

## Tools used
- Python: `pandas`, `numpy`, `scikit-learn`

---

## Author
M. Osvaldo Hernández Montoya
