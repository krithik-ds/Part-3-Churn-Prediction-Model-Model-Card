# Part 3 — Churn Prediction Model & Model Card

Capstone: D2C Customer Churn Intelligence & Retention API

---

## What's in this repo

```
├── churn_model.ipynb         # Main notebook: Preprocessing → training → threshold tuning → evaluation → export
├── model.pkl                 # Output: Saved Random Forest model pipeline
├── metrics.json              # Output: Test set performance metrics
├── error_analysis.md         # Detailed report on 10 specific False Positive / False Negative cases
├── model_card.md             # Model documentation (intended use, bias, metrics, limitations)
├── requirements.txt          # Python dependencies
├── data/                     # Folder to place raw data (see below)
└── charts/                   # Saved charts (feature importances)
```

---

## Setup

1. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

2. **Data setup:**
   Create a `data/` folder in the root or place the dataset package files in the `../dataset/` directory. By default, the notebook expects the data to be in `../dataset/rfm_modeling_snapshot.csv`.

---

## Run the Notebook

```bash
jupyter notebook churn_model.ipynb
```

Run all cells top to bottom. It will load the data, train the baseline Logistic Regression and Random Forest models, select the 0.40 probability threshold, output metrics to `metrics.json`, and save the model to `model.pkl`.

---

## Key Results (Test Set)

We selected the **Random Forest Classifier** at a probability threshold of **0.40** (optimized on validation set for business costs).

| Metric | Baseline (Logistic Regression) | Random Forest (Threshold 0.40) |
|---|---:|---:|
| **Accuracy** | 79.17% | **82.14%** |
| **Precision** | 79.88% | **79.67%** |
| **Recall (Sensitivity)** | 77.98% | **86.31%** |
| **F1-Score** | 78.92% | **82.86%** |
| **ROC-AUC** | 88.48% | **88.83%** |

### Confusion Matrix (Random Forest, Threshold 0.40):
* **True Negatives (TN):** 131
* **False Positives (FP):** 37 (targeted with coupon, minor margin loss)
* **False Negatives (FN):** 23 (missed churners, high lost LTV risk)
* **True Positives (TP):** 145 (correctly caught and targeted)

You can find the saved Confusion Matrix heatmap in the `charts/confusion_matrix.png` file.

---

## Business & Threshold Justification
* Standard threshold (0.50) is not good for churn because losing a customer costs around **₹1,400** (average spend), while sending an offer/campaign coupon costs only **₹25**.
* By reducing the threshold to **0.40**, we increased our **Recall from 72.1% to 86.3%** on the test set. We catch 23 more churners, saving potential customers from leaving. This trade-off is highly profitable for the brand.

---

## Top 5 Features Driving Churn
1. **`recency_days` (28.9% importance):** Days since the last order.
2. **`last_visit_days_ago` (22.1% importance):** Inactivity on the web/app.
3. **`monetary_180d` (8.9% importance):** Customer spend in the last 180 days.
4. **`frequency_180d` (5.9% importance):** Number of purchases in the last 180 days.
5. **`category_diversity_180d` (4.7% importance):** Number of distinct categories bought.
