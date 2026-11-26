# 🛡️ FraudShield: Credit Card Fraud Detection Analysis

> **Which transaction patterns consistently signal fraud, and what triggers should a real-time alert system use?**

An end-to-end data analytics project for the **BFSI (Banking, Financial Services & Insurance)** domain. I audit and engineer features on 284,807 credit card transactions in Python, evaluate rule-based fraud alerts using precision and recall, and present the findings in a 3-page Power BI dashboard with a recommended layered alert policy for a bank.

**Tools:** Python (pandas, NumPy, matplotlib) · Power BI · DAX

---

## 📊 Dashboard Preview

| Overview | Fraud Patterns | Alert Rules |
|---|---|---|
| (images/01_overview.png) | (images/02_fraud_patterns.png) | (images/03_alert_rules.png) |

---

## 🎯 Business Problem

Fraud is rare but costly. Only **0.17%** of transactions are fraudulent, so a naive alert system either misses most fraud or buries analysts in false alarms. A bank needs to know:

1. **When** does fraud happen? (time-based patterns)
2. **How much** is typically stolen? (amount distributions)
3. **What behaviour** looks abnormal? (anomalous feature profile)
4. **Which rules** should trigger alerts, and at what cost in false alarms?

---

## 📁 Dataset

- **Source:** Credit Card Fraud Detection dataset (ULB / Worldline, via Kaggle), provided as `creditcard_compressed.zip`
- - **Size:** 284,807 transactions × 31 columns, covering two days of card activity
- **Fraud cases:** 492 (0.173%)
- **Features:** `Time` (seconds since first transaction), `Amount`, `Class` (0 = legit, 1 = fraud), and `V1–V28` (anonymised PCA components)
- **Data quality:** 0 null values, 1,081 duplicate rows

> The raw `creditcard.csv` is included in this repo. It is compressed because of its size. Download it from `creditcard_compressed.zip` and place it in the project folder.

**Note on "demographics":** the dataset has no age, gender or location. As a substitute, I profiled fraud using the deviation of the anonymised `V` features between fraud and legitimate transactions.

---

## 🔧 Methodology

1. **Load and audit:** checked shape, nulls, duplicates and class imbalance. I kept duplicates in the main analysis because identical amounts in the same second can be legitimate.
2. **Feature engineering:**
   - `hour` and `day` derived from `Time`
   - `time_bucket`: Night (0-5), Morning (6-11), Afternoon (12-17), Evening (18-23)
   - `amount_band`: ≤1, 1-10, 10-50, 50-100, 100-250, 250-500, 500-1000, 1000+
3. **Time-based analysis:** fraud rate by hour, time bucket and day × hour.
4. **Amount analysis:** fraud vs legit distributions (log scale) and fraud rate by amount band.
5. **Anomalous behaviour profile:** correlation of `V1–V28` with fraud, plus fraud vs legit feature means.
6. **Alert rule design and evaluation:** seven candidate rules scored on alerts raised, frauds caught, false alarms, **precision** and **recall**.
7. **Export and visualise:** summary tables exported as CSVs and built into a Power BI dashboard.

> `Time` is measured from the first transaction, not midnight, so "hour of day" is an approximation.

---

## 🔍 Key Findings

### When fraud happens
- Fraud rate is **0.52% at night (0-5 AM)**, about **3-4x higher** than any daytime bucket (0.12-0.17%).
- Fraud peaks at **2 AM (1.71%)**, when transaction volume is at its lowest.

### How much
- **37% of all frauds are transactions of $1 or less**, which is consistent with card-testing behaviour.
- Large amounts ($250-$1,000) also carry a higher fraud rate than mid-range amounts.
- Median fraud amount is **$9.25** vs **$22.00** for legit transactions, so fraud is not mainly about big-ticket purchases.

### What looks abnormal
- The features most associated with fraud are **V17, V14, V12, V10** (all strongly negative for fraud) and **V11, V4** (positive).
- Correlation with fraud: V17 (-0.33), V14 (-0.30), V12 (-0.26).

### Which rules work

| Rule | Alerts | Frauds caught | Precision | Recall |
|---|---:|---:|---:|---:|
| R1: Night hours (1-5 AM) | 16,239 | 118 | 0.73% | 23.98% |
| R2: Micro amount (≤ $1) | 30,492 | 181 | 0.59% | 36.79% |
| R3: High amount ($250-$1000) | 19,963 | 65 | 0.33% | 13.21% |
| R4: V14 < -5 | 541 | 324 | 59.89% | 65.85% |
| R5: V17 < -5 | 328 | 265 | **80.79%** | 53.86% |
| R6: V14 < -5 OR V17 < -5 OR V12 < -5 | 623 | 379 | 60.83% | **77.03%** |
| R7: Night AND V14 < -3 | 150 | 105 | 70.00% | 21.34% |

**Takeaway:** time and amount rules alone raise tens of thousands of false alarms. Behaviour-pattern rules (V14, V17, V12) catch up to **77% of fraud** with only a few hundred alerts.

---

## ✅ Recommended Layered Alert Policy

| Layer | Trigger | Why |
|---|---|---|
| 🔴 **Block / urgent alert** | `V17 < -5` | 81% precision, so very few false alarms |
| 🟠 **Review queue** | `V14 < -5` OR `V17 < -5` OR `V12 < -5` | Catches 77% of fraud at 61% precision |
| 🟡 **Escalate** | Night-time AND `V14 < -3` | 70% precision |
| ⚪ **Risk-score boosters only** | Night hours, amount ≤ $1, amount $250-$1000 | Add points to a risk score but never trigger an alert alone |

---

## 📂 Repository Structure

```
Credit-Card-Fraud-Detection/
│
├── 📊 dashboard/
│   ├── Credit_Card_Fraud_Detection.pbix   # Power BI dashboard (FraudShield)
│   ├── 01_overview.png                    # Page 1 – Overview
│   ├── 02_fraud_patterns.png              # Page 2 – Fraud Patterns
│   └── 03_alert_rules.png                 # Page 3 – Alert Rules
│
├── 🐍 cleaning_code.ipynb                 # Python (pandas) data cleaning & EDA
│
├── 🗂️ creditcard_compressed.zip           # Raw dataset (compressed)
├── 🗂️ fraud_clean-compressed.zip          # Cleaned dataset (compressed)
│
└── 📄 README.md                           # Project documentation
```

---

## 🚀 How to Run

1. Download `creditcard.csv` from `creditcard_compressed.zip` and place it in the project folder.
2. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib jupyter
   ```
3. Open and run `cleaning_code.ipynb`. It creates `charts/` and `powerbi/` folders with the PNG charts and summary CSVs.
4. Open `Credit_Card_Fraud_Detection.pbix` in **Power BI Desktop** to explore the dashboard.

---

## ⚠️ Limitations

- Only **two days** of data, so day-of-week and seasonal patterns cannot be assessed.
- `V1–V28` are anonymised, so the behavioural rules are statistically useful but cannot be explained in business terms.
- Rules were evaluated on the **same data they were designed on** (no train/test split), so real-world precision and recall will likely be lower.
- Hour of day is approximate because `Time` is relative to the first transaction.

## 🔭 Next Steps

- Add a train/test split to validate rule thresholds on unseen data.
- Compare rules against a machine learning model (e.g. logistic regression, random forest) using precision-recall curves.
- Estimate the financial cost of false alarms vs missed fraud to tune thresholds.

---

## 👤 Author

**Ayush Yadav**, Aspiring Data Analyst
🔗 [LinkedIn](https://www.linkedin.com/in/ayush-yadav-914246198) · 📧 ayush3112yadav@gmail.com
