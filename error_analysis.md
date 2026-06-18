# Error Analysis Report — Churn Prediction Model

This report does a detailed analysis of where our Random Forest churn model makes mistakes. We look at two types of errors on our test set:
1. **False Positives (FP):** Customers model says will churn, but they did not churn.
2. **False Negatives (FN):** Customers model says will not churn, but they actually churned.

We have used a probability threshold of **0.40** for predicting churn. Let us look at 10 specific customer examples to understand why the model got them wrong and what the business risk is.

---

## 1. False Positive (FP) Examples
These are customers predicted as churners (probability $\ge 0.40$), but they placed at least one order in the 60 days after the snapshot date.

### Detailed Table of 5 False Positives:
| Customer ID | Recency (Days) | Last Visit (Days Ago) | Monetary (180d) | Frequency (180d) | Sessions (30d) | Avg Rating | Churn Prob | Actual Churn |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| **CUST00044** | 72 | 10 | ₹899.51 | 1 | 6 | 4.0 | 0.475 | 0 |
| **CUST00109** | 92 | 16 | ₹1622.28 | 2 | 1 | 4.5 | 0.504 | 0 |
| **CUST00335** | 148 | 22 | ₹1328.14 | 2 | 7 | 3.5 | 0.681 | 0 |
| **CUST00437** | 151 | 33 | ₹729.22 | 1 | 0 | 4.0 | 0.870 | 0 |
| **CUST00491** | 97 | 20 | ₹540.89 | 1 | 10 | 4.0 | 0.577 | 0 |

### Analysis of individual cases:

1. **CUST00044 (Prob: 47.5%)**
   - **Why model failed:** The customer did not buy anything for 72 days, which is above the average. So the model flagged them. But they were still visiting the app (`sessions_30d` = 6, `last_visit_days_ago` = 10), which showed they were still interested. They eventually bought again.
   - **Business Risk:** We might send them a discount offer (like 15% off) when they would have bought anyway. This causes a minor margin loss, but it keeps them happy.

2. **CUST00109 (Prob: 50.4%)**
   - **Why model failed:** High recency of 92 days and almost zero recent activity (`sessions_30d` = 1) made the model think they are gone. But they had a high rating (4.5) and had spent ₹1,622 before. They were just on a long buying cycle and came back.
   - **Business Risk:** Wasting campaign budget on a high-satisfaction customer who is just slow to repeat purchase.

3. **CUST00335 (Prob: 68.1%)**
   - **Why model failed:** Customer had not purchased in 148 days. The model was 68% sure they would churn. But they had 7 sessions in the last 30 days, which indicates they came back to browse and ended up ordering.
   - **Business Risk:** Giving a win-back discount to a customer who had already decided to return on their own.

4. **CUST00437 (Prob: 87.0%)**
   - **Why model failed:** Customer was totally inactive — 151 days since last order, 33 days since last visit, 0 sessions. The model correctly identified them as a massive churn risk (87%). Surprisingly, they made a purchase later. This is an outlier case.
   - **Business Risk:** We definitely would have sent a heavy discount to this customer. But since their churn risk was so high, sending a discount was the right business move anyway, even if they returned "naturally" in the data.

5. **CUST00491 (Prob: 57.7%)**
   - **Why model failed:** Customer did not order for 97 days and had 1 support ticket. But their app engagement was very high (`sessions_30d` = 10). The model ignored the high session count and focused on high recency, predicting churn.
   - **Business Risk:** Sending a discount when they were already active on the app and browsing.

---

## 2. False Negative (FN) Examples
These are customers predicted as active/loyal (probability $< 0.40$), but they did not make any purchase in the 60 days post-snapshot.

### Detailed Table of 5 False Negatives:
| Customer ID | Recency (Days) | Last Visit (Days Ago) | Monetary (180d) | Frequency (180d) | Sessions (30d) | Tickets (90d) | Avg Rating | Churn Prob | Actual Churn |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| **CUST00067** | 63 | 5 | ₹1246.44 | 2 | 1 | 1 | 2.50 | 0.395 | 1 |
| **CUST00184** | 14 | 6 | ₹2456.91 | 3 | 6 | 0 | 3.33 | 0.092 | 1 |
| **CUST00247** | 57 | 14 | ₹937.32 | 2 | 5 | 0 | 3.00 | 0.369 | 1 |
| **CUST00414** | 3 | 0 | ₹311.44 | 1 | 5 | 1 | 4.00 | 0.336 | 1 |
| **CUST00438** | 64 | 22 | ₹2466.39 | 3 | 6 | 2 | 3.00 | 0.315 | 1 |

### Analysis of individual cases:

1. **CUST00067 (Prob: 39.5%)**
   - **Why model failed:** Their recency was 63 days and they visited 5 days ago, so they looked active. But their predicted probability (39.5%) was just below our 40% cutoff. They had a bad experience (average rating 2.50, and 1 support ticket), which the model didn't weigh heavily enough. They churned.
   - **Business Risk:** We miss sending them a win-back offer. Since they had a bad rating, a support resolution or apology email was needed, but we did nothing and lost them.

2. **CUST00184 (Prob: 9.2%)**
   - **Why model failed:** Extremely active customer on paper — ordered just 14 days ago, spent ₹2,456 (3 orders), 6 sessions. The model gave them only a 9.2% churn risk. Yet, they churned completely. This could be a customer who bought items for an event or moved to another brand suddenly.
   - **Business Risk:** We do not target them because they look highly active. This is a severe loss because they were a high-value customer.

3. **CUST00247 (Prob: 36.9%)**
   - **Why model failed:** Recency of 57 days and 5 sessions made them look safe (under 40%). But they had no orders in nearly two months and stopped buying.
   - **Business Risk:** Missing the window to re-engage them before they drifted into total dormancy.

4. **CUST00414 (Prob: 33.6%)**
   - **Why model failed:** Customer placed an order 3 days ago and visited on snapshot day, so the model saw zero recency risk. However, this was their first purchase (frequency = 1), and they had raised a support ticket. They had a bad first experience and churned immediately.
   - **Business Risk:** Losing a newly acquired customer right after their first order. First-time buyers with support issues must be handled with high priority.

5. **CUST00438 (Prob: 31.5%)**
   - **Why model failed:** Decent buyer with 3 orders and ₹2,466 spend. They looked safe, but they had raised 2 support tickets in the last 90 days and their rating was average (3.0). The model did not penalize the support tickets enough.
   - **Business Risk:** High-value customer lost due to unresolved or repeated service issues.

---

## 3. Overall Business Impact & Mitigation

### False Positives (Margins Loss vs. Customer Relationship)
* **Business Risk:** Sending discounts to customers who would have purchased anyway. This dilutes our profit margins.
* **Mitigation:** Instead of giving flat discounts to all predicted churners, we can use **non-monetary campaigns** (like early access to new launches, skin-care tips, or loyalty points) for customers with high recent sessions (like `sessions_30d` > 5) but high recency.

### False Negatives (Lost Revenue & Lifetime Value)
* **Business Risk:** Losing high-value customers (like CUST00184 and CUST00438) without making any retention effort. The cost of losing a customer (₹1,400+ spend) is much higher than sending a ₹25 coupon.
* **Mitigation:** We should set up a **trigger-based support flow**. Any customer who raises a support ticket and leaves a rating $\le 3.0$ should automatically get a customer service follow-up call, regardless of what the churn model predicts.
