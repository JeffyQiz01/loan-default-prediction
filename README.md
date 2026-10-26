# Loan Default Prediction System

A Python machine-learning project that predicts the `Default` target in
`LC_loans_granting_model_dataset.csv`. The training notebook compares **Logistic
Regression, Decision Tree, and Random Forest**. This project does **not** include
Explainable AI (XAI).

## Project objective

Build and evaluate a classification model that estimates whether a loan record
is likely to default (`Default = 1`) or not default (`Default = 0`) using
historical loan data.

## Project structure

loan_default_prediction_project/
├── train.ipynb                 # EDA, preprocessing, model training/evaluation
├── predict.ipynb  # Predict on new applicant CSVs
├── requirements.txt
├── README.md
├── Dataset/
│   └── LC_loans_granting_model_dataset.csv
└── artifacts/                          # Created by the training notebook
    ├── loan_default_model.joblib
    ├── validation_model_comparison.csv
    ├── test_metrics.csv
    ├── test_predictions.csv
    └── run_metadata.json


The exact contents of `artifacts/` are generated when the notebook's save-artifacts
cell runs. The training notebook displays EDA charts inline; **it does not currently
save those charts as PNG files or write EDA summary CSV files**.

## 1. Setup

Use Python 3.10 or newer.

Create and activate a virtual environment in Windows PowerShell:

```powershell
python -m venv .loan
.\.loan\Scripts\Activate.ps1
```

If PowerShell blocks activation, run this for the current terminal session:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
```

Install the dependencies:

```powershell
python -m pip install -r requirements.txt
```

The notebook needs Jupyter/IPython in addition to the model-training dependencies.
If they are not listed in your requirements file, install them with:

```powershell
python -m pip install jupyter ipython
```

## 2. Dataset

Place the dataset at:

```text
Dataset/LC_loans_granting_model_dataset.csv
```

The training notebook has a dataset-path setting near the top. Check that it points
to the actual CSV location before running the notebook.

The target column must be named `Default` and contain values that can be interpreted
as 0 (non-default) and 1 (default), including supported text values such as
`Yes`/`No` or `Default`/`Non-default`.

## 3. Run the training notebook

From the project folder, launch Jupyter:

```powershell
python -m jupyter notebook
```

Open `train.ipynb` (or the final filename you have given your training
notebook), then run **all cells from top to bottom**.

The notebook workflow:

1. Loads and validates the dataset and target column.
2. Performs exploratory data analysis (EDA): target distribution, missing values,
   numeric feature distributions, default rates by selected categories, numeric
   correlation, and default rate over time when a recognized date column exists.
3. Excludes the target, common identifiers, and selected columns that may be known
   only after a loan outcome.
4. Splits data into training, validation, and test sets. It uses chronological
   splitting when a recognized date column is sufficiently complete and all three
   splits contain both target classes; otherwise it uses stratified random splitting.
5. Imputes missing numeric values with the median, standardizes numeric features,
   imputes categorical values with the most frequent category, and one-hot encodes
   categorical features.
6. Trains Logistic Regression, Decision Tree, and Random Forest.
7. Compares models using validation metrics and selects the model with the highest
   validation PR-AUC.
8. Evaluates the selected model on the held-out test set, showing a classification
   report and confusion matrix.
9. Saves the selected model and evaluation reports under `artifacts/`.

## 4. Files saved by the training notebook

When the final save cell runs successfully, it creates:

- `artifacts/loan_default_model.joblib` — selected trained model and metadata needed
  by the prediction workflow.
- `artifacts/validation_model_comparison.csv` — validation metrics for all three models.
- `artifacts/test_metrics.csv` — held-out test metrics for the selected model.
- `artifacts/test_predictions.csv` — actual labels, predicted labels, and predicted
  default probabilities for the test set.
- `artifacts/run_metadata.json` — selected model, feature list, split method, and
  test metrics.

**EDA charts are displayed inside the notebook, not saved to `artifacts/eda/` by the
current notebook code.** If PNG files or EDA CSV summaries are required for submission,
the notebook needs an additional save-to-file step.

## 5. Predict for new applicants

First, run the training notebook successfully so that
`artifacts/loan_default_model.joblib` exists.

Open `predict.ipynb` and run its cells from top to bottom.
The notebook asks you to select an applicant CSV file and displays:

- Default probability (%)
- Predicted default (`Yes`/`No`)
- Risk band (`Low`, `Medium`, or `High`)

The applicant CSV must contain all predictor columns used during training, with the
same feature names and compatible data formats. Do not use the `Default` target as
a substitute for a missing predictor. Prediction results are displayed in the
notebook and are not saved to a new CSV.

If you use the standalone script instead, run:

```powershell
python predict.ipynb 
```

## 6. Models and evaluation

The training notebook compares:

- **Logistic Regression** — baseline linear classifier
- **Decision Tree** — single tree-based classifier
- **Random Forest** — ensemble of decision trees

Model selection is based on validation PR-AUC. The held-out test evaluation reports
accuracy, precision, recall, F1-score, ROC-AUC, PR-AUC, a classification report, and
a confusion matrix. Use the actual metrics printed by your current notebook run;
results can change when the data, features, split, dependencies, or code change.

## 7. EDA topics

The notebook displays visualizations and summaries for:

- Target class distribution
- Missing-value percentages
- Numeric feature distributions (using a sample for plotting)
- Default rate by selected categorical features, such as loan purpose,
  home ownership, employment length, and state
- Numeric-feature correlation matrix
- Default rate over time when a recognized issue/origination date is available

These plots describe associations in the dataset; they do not prove that a feature
causes loan default.

## Important data-quality and research checks

1. Confirm the definition and encoding of `Default` from the dataset documentation.
2. Review feature definitions and ensure each retained predictor is available at the
   time a loan application would be assessed. Name-based exclusions do not guarantee
   that all target leakage has been removed.
3. The target is imbalanced in the previously observed dataset (approximately 80%
   non-default and 20% default), so accuracy alone is not sufficient.
4. A 0.50 probability threshold is a starting point. Threshold selection should use
   validation data and consider the costs of false positives and false negatives.
5. Investigate unusually low default rates in the latest periods; newer loans may
   not have had enough time for outcomes to be observed.
6. Check probability calibration, fairness, legal requirements, security, and
   monitoring before considering any real-world lending use.

## Project limitations

This is an educational/research project, not a production lending decision system.
Model metrics are estimates on a held-out test set and should not be used as the sole
basis for real lending decisions.
