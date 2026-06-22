# End-to-End Customer Churn Predictor Pipeline

An enterprise-grade data science and machine learning pipeline that predicts customer attrition (churn) for a modern direct-to-consumer (D2C) e-commerce brand. This project demonstrates full-stack software capabilities by joining automated relational database management, data-warehouse level feature engineering, and a production-ready machine learning pipeline.

---

## Core Architecture & Pipeline Flow
The project is decoupled into 4 distinct phases:
1. **Database Layer (SQLite):** Implements a relational star schema enforcing data integrity constraints, foreign keys, multi-value check constraints, and cascading deletions.
2. **Feature Engineering Layer (Advanced SQL):** Computes complex behavioral transforms directly inside the database engine. Leverages Common Table Expressions (CTEs), safe division checking, formatting casting, and Julian day date math to construct behavioral metrics.
3. **Preprocessing Pipeline (Pandas & Scikit-Learn):** Implements an airtight `ColumnTransformer` assembly line to scale numeric dimensions and encode multi-class categorical traffic channels.
4. **Optimization Layer (GridSearchCV):** Combines parallel processing cross-validation and hyperparameter tuning to select an optimal `RandomForestClassifier` tuned specifically to combat class imbalance.

---

## Relational Database Schema

The infrastructure tracks customer behavior across three distinct tables interconnected by primary and foreign keys:

### 1. `users` (Parent Table)
* `user_id`: `INTEGER PRIMARY KEY AUTOINCREMENT` — Unique identification fingerprint for each customer.
* `signup_date`: `TEXT NOT NULL` — Registration timestamp (ISO8601 string).
* `country`: `TEXT` — Geographic country location of the user.
* `traffic_source`: `TEXT` — Marketing acquisition channel (`Organic`, `Google Ads`, `Facebook`, `Affiliate`, `Instagram`, `YouTube`).

### 2. `orders` (Child Table)
* `order_id`: `INTEGER PRIMARY KEY AUTOINCREMENT` — Unique identifier for each order.
* `user_id`: `INTEGER` — Foreign Key pointing back to `users(user_id)` with `ON DELETE CASCADE`.
* `order_date`: `TEXT NOT NULL` — Transaction timestamp.
* `order_amount`: `REAL NOT NULL` — Monetary cost of the order.
* `status`: `TEXT CHECK(status IN ('Completed', 'Returned', 'Cancelled'))` — Hard integrity guardrail tracking transaction validity.

### 3. `web_logs` (Child Table)
* `log_id`: `INTEGER PRIMARY KEY AUTOINCREMENT` — Unique behavioral log row identifier.
* `user_id`: `INTEGER` — Foreign Key pointing back to `users(user_id)` with `ON DELETE CASCADE`.
* `activity_date`: `TEXT NOT NULL` — Event day tracking string.
* `page_views`: `INTEGER DEFAULT 0` — Continuous number of site clicks.
* `session_duration_secs`: `INTEGER DEFAULT 0` — Total time spent on site.

---

## Engineered Machine Learning Features

Rather than passing raw rows directly to the model, we perform **Database-Level Feature Engineering** to derive strong predictive signals:

* **Target Variable (`churned`):** Binary classification marker ($1$ for churned, $0$ for active). If a customer has never placed a completed order, or if their last completed transaction was $> 60$ days relative to our assessment date, they are categorized as **Churned**.
* **Average Order Value (AOV):** Total monetization spend divided by successful transactions. Handled via conditional `CASE` checks to avoid division-by-zero crashes.
* **Views Per Second (Engagement Velocity):** Total page views divided by accumulated session seconds, exposing fast-browsing transactional behaviors. 
* **Account Tenure:** The age span of the user record measured in total days between system registration and calculation date (`julianday`).

---

## Execution Flow

### 1. Project Prerequisites
Ensure your local Python environment has the necessary packages installed:
```bash
pip install pandas scikit-learn faker
