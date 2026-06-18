# Model Card — D2C Churn Prediction Model

This model card provides information about our churn prediction model, which predicts if a D2C customer will churn (no purchases) in the next 60 days.

---

## 1. Intended Use
* **Primary Users:** Marketing, customer relationship management (CRM), and customer support teams.
* **Intended Use Case:** To identify customers at risk of churn so that we can send targeted retention campaigns (like offers, loyalty perks, or customer service follow-ups) instead of giving discounts blindly.
* **Out-of-Scope Uses:** 
  - Predicting the exact day a customer will churn.
  - Using the model for pricing decisions or blocking customers.
  - Applying the model to B2B customers (designed only for D2C retail).

---

## 2. Data Used
* **Data Sources:** Combined features from customer profile (`customers.csv`), order history (`orders.csv`), support tickets (`support_tickets.csv`), and web/app events (`web_events_snapshot.csv`).
* **Snapshot Date:** `2025-09-30`. All features are engineered from data available on or before this date to prevent future data leakage.
* **Target Definition:** `churn_next_60d` — binary label where `1` means the customer did not make any purchase in the 60 days following the snapshot date (`2025-10-01` to `2025-11-29`), and `0` means they made at least one purchase.
* **Data Splits:**
  - **Train Set:** 1,728 customers (53% non-churn, 47% churn)
  - **Validation Set:** 336 customers (56% non-churn, 44% churn)
  - **Test Set:** 336 customers (50% non-churn, 50% churn)

---

## 3. Model Approach
* **Baseline Model:** Logistic Regression classifier (scikit-learn).
* **Final Model:** Random Forest Classifier (scikit-learn) with hyperparameters set to:
  - `n_estimators = 150`
  - `max_depth = 8`
  - `min_samples_leaf = 8`
  - `random_state = 42`
* **Preprocessing:**
  - Categorical fields (like `city_tier`, `age_group`, `acquisition_channel`, `loyalty_tier`, `preferred_category`, `marketing_consent`) are One-Hot Encoded. Missing `loyalty_tier` values are imputed with `'None'`.
  - Numerical fields are scaled using `StandardScaler`.
* **Decision Threshold:** We selected a decision threshold of **0.40** (instead of standard 0.50) to prioritize Recall (capturing more churners) because the cost of losing a customer (₹1,400) is much higher than the cost of a retention campaign (₹25).

---

## 4. Performance Metrics (Test Set)
Evaluated on the test split (336 customers) at a threshold of **0.40**:

* **Accuracy:** 82.14%
* **Precision:** 79.67% (when we predict a customer will churn, they actually churn 79.7% of the time)
* **Recall (Sensitivity):** 86.31% (we correctly catch 86.3% of all actual churners)
* **F1-Score:** 82.86%
* **ROC-AUC:** 0.8883

### Confusion Matrix on Test Set:
* **True Negatives (TN):** 131 (Predicted active, stayed active)
* **False Positives (FP):** 37 (Predicted churn, stayed active)
* **False Negatives (FN):** 23 (Predicted active, actually churned)
* **True Positives (TP):** 145 (Predicted churn, actually churned)

---

## 5. Limitations
* **Cold Start Problem:** The model requires order and engagement history. It may not perform well for brand new signups who haven't placed their first order yet (handled by the onboarding sequence instead).
* **Short-Term Context:** The model uses web events and support tickets from a 30-day and 90-day window. If customer behavior changes over a longer cycle, the model will not capture it.
* **Product Quality Issues:** The model predicts churn based on behavior, but it cannot foresee sudden external events (e.g., competitor launching a huge sale, or supply chain delays).

---

## 6. Ethical Risks & Bias
* **Age Group Bias:** The model uses `age_group` as a feature. Older age groups (like `45+`) might have different buying frequencies, which could cause the model to flag them more often and give them more discounts.
* **City Tier Discrimination:** Customers in `Tier 3` cities might have longer delivery times, which correlates with churn. The model might systematically predict higher churn risk for Tier 3 customers. We must ensure we do not reduce service quality or discriminate against Tier 3 customers based on this.
* **Discount Addiction:** Repeatedly targeting high-churn-risk customers with discounts might train them to never buy at full price, hurting brand value.

---

## 7. Monitoring & Retraining Plan
* **Drift Monitoring:** We must monitor if the distribution of key features (like `recency_days` or `sessions_30d`) shifts in production compared to our training dataset.
* **Prediction Distribution:** Monitor the percentage of customers flagged as "high risk" week-over-week. A sudden jump suggests data issues or code changes.
* **Retraining Trigger:** The model should be retrained **every quarter** (90 days) using the latest customer snapshots to capture seasonal behavior changes (like Diwali or holiday sales).
* **When the model should NOT be used:** Do not use the model during major website crashes or shipping disruptions, as the baseline customer behavior will be temporarily altered, making the predictions unreliable.
