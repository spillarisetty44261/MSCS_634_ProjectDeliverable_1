# MSCS 634 – Project Deliverable 1: Data Collection, Cleaning, and Exploration

## 1. Dataset summary

The IBM Telco Customer Churn dataset describes **7,043 customers** of a fictional California phone and internet provider. Each customer has **21 attributes**:

- **Demographics:** `gender`, `SeniorCitizen`, `Partner`, `Dependents`
- **Account:** `tenure` (months), `Contract`, `PaperlessBilling`, `PaymentMethod`, `MonthlyCharges`, `TotalCharges`
- **Services:** `PhoneService`, `MultipleLines`, `InternetService`, and six add-ons (`OnlineSecurity`, `OnlineBackup`, `DeviceProtection`, `TechSupport`, `StreamingTV`, `StreamingMovies`)
- **Target:** `Churn`, whether the customer left within the last month
- **Identifier:** `customerID`

Source: IBM's public GitHub repository [`IBM/telco-customer-churn-on-icp4d`](https://github.com/IBM/telco-customer-churn-on-icp4d) (also on Kaggle as `blastchar/telco-customer-churn`).

### Why this dataset
- **Exceeds the size requirements:** 7,043 records (500+ required) and 21 attributes (8–10 required).
- **Addresses a real business problem.** Churn directly drives revenue loss in subscription businesses.
- **Mixes data types** (numeric, binary, multi-class categorical), so it exercises the full preprocessing toolkit.
- **Contains genuine data-quality issues** (see Section 3), which makes the cleaning phase meaningful.
- **Supports every later deliverable:** regression (predict `MonthlyCharges`), classification (predict `Churn`), clustering (customer segments), and association-rule mining (service combinations linked to churn).

---

## 2. Key insights from EDA

| # | Insight | Evidence |
|---|---|---|
| 1 | **Churn is moderately imbalanced:** 26.5% Yes vs 73.5% No | Figure 3 |
| 2 | **Contract type is the strongest driver:** month-to-month 43% churn, one-year 11%, two-year 3% (Cramér's V = 0.41) | Figures 5, 10 |
| 3 | **Churn is concentrated in the first year:** 47% in months 0–12, falling to 10% after month 48. The relationship is non-linear and interacts with contract type | Figure 6 |
| 4 | **Fiber-optic customers churn at 42%**, more than double DSL (19%), despite being the premium product | Figure 5 |
| 5 | **Electronic-check payers churn at 45%**, compared with 15–17% for automatic payment methods | Figure 5 |
| 6 | **Lacking TechSupport or OnlineSecurity roughly doubles churn** (~31% vs ~15%) | Figure 5 |
| 7 | **Simpson's-paradox pricing effect:** churners pay more *overall*, but *less within each internet tier*, because they hold fewer add-ons. Price is therefore a confounded signal | Figures 4, 7 |
| 8 | **`TotalCharges` ≈ `tenure` × `MonthlyCharges`** (r = 0.83 with tenure), a multicollinearity risk for linear models | Figures 2, 8, 9 |
| 9 | **`gender` and `PhoneService` are not significantly associated with churn** (p > 0.05), so they are candidates to drop | Figure 10 |

---

## 3. Data cleaning steps

The notebook first **audits** the data for each problem type and only then **cleans** it, so every change traces back to a documented finding. **No records were dropped**: all 7,043 customers are retained.

| Step | Issue found | Action and rationale |
|---|---|---|
| 1 | Potential whitespace or capitalisation variants in labels | Stripped whitespace from all text columns. The audit found no variants; this step is defensive |
| 2 | `TotalCharges` stored as **text** | Converted to `float` |
| 3 | **11 hidden missing values:** blank strings `" "` in `TotalCharges`, which `isna()` does not detect | All 11 rows have `tenure = 0`, and they are the only such rows: new customers with no completed billing cycle. **Imputed 0** because `TotalCharges ≈ tenure × MonthlyCharges = 0`. Dropping the rows would remove every new customer; mean imputation would be factually wrong |
| 4 | **Inconsistent binary encoding:** `SeniorCitizen` uses 0/1, while all other binary columns use Yes/No | Recoded to Yes/No |
| 5 | **Redundant levels:** "No internet service" (6 columns) and "No phone service" (`MultipleLines`) | Verified as *perfectly* equivalent to `InternetService == "No"` / `PhoneService == "No"`, then collapsed to "No" (9,838 cells). No information is lost, and redundant items that would create trivial association rules are removed |
| 6 | Duplicates | 0 exact duplicate rows and 0 repeated IDs. 22 rows share a profile with another customer under a different ID; all are 1-month, month-to-month customers on simple plans. These were **kept** as legitimate distinct customers |
| 7 | Noise and outliers | **IQR and |z| > 3 tests flagged 0 outliers.** A business-rule check (`TotalCharges` vs `tenure × MonthlyCharges`) has a median ratio of 1.000; the 27 records (0.38%) that deviate by more than 25% are all short-tenure customers, consistent with plan changes or prorated first bills. They were kept |
| 8 | Helper features | Added `ChurnFlag` (0/1), `TenureGroup` (4 bins), and `NumAddOnServices` (0–6) to support EDA and later modelling |

---



