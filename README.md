# SocialAnxietyPrediction
**Authors:** Sreejony Sengupta, Brendan Hendricks, Nandini Anand Kumar, Niharika Pappu, Christian Herpin  

## Overview
This project analyzes the **Enhanced Anxiety Dataset** (*n = 11,000; 19 variables*), profiling demographic, behavioral, health, and psychological factors to predict a person’s **self‑reported Anxiety Level (1–10)** and identify the strongest drivers of higher anxiety. Although the dataset is synthetic, we treat it as a representative sample to illustrate how everyday signals can support **early identification** and **faster intervention**.

**Why this matters.** Social anxiety affects millions, and care is often delayed. Robust screening can speed referrals, target resources, and shape supportive programs. Our holistic dataset and modeling demonstrate how to surface actionable effect sizes and ranked risk factors for clinicians, counselors, program planners, and product teams.

---

## Data
- **Source:** Enhanced Anxiety Dataset (synthetic).  
- **Size:** 11,000 rows, 19 variables.  
- **Target:** `Anxiety Level (1–10)` (continuous).  
- **Link:** Kaggle (inputs): https://www.kaggle.com/code/enesfiliz/social-anxietyand-lifestyle-analysis/input

> *Note:* Because this is synthetic, findings are **indicative**, not clinical evidence.

---

## Exploratory Analysis (EDA)
1. **Data hygiene.** Standardized categorical values; verified no missing data; dropped **`Stress Level (1–10)`** to prevent it overshadowing other predictors (potential proxy/leakage with the target).  
2. **Target distribution.** Histogram + KDE of **Anxiety Level** showed a center around ~4 with a modest high-end tail → sufficient variation.  
3. **Numeric predictors.** Histograms (with mean line and KDE where appropriate) and boxplots of anxiety vs. predictors. Ordinal discrete variables (e.g., **Sweating Level (1–5)**, **Diet Quality (1–10)**) were plotted without KDE. Patterns aligned with intuition: lower sleep and higher caffeine consumption associate with higher anxiety; therapy sessions showed a non‑monotonic pattern at the extreme end (likely due to small counts in 9–10.5 bins).  
4. **Categorical comparisons.** Boxplots across categories (e.g., family history, recent major life event, smoking, medication, dizziness, gender). Medians were higher for **family history**, **recent major life event**, **smoking**, **medication use**, **dizziness**, and **“other”** gender.  
5. **Correlations.** A heatmap highlighted moderate relationships clustering in lifestyle/physiology (sleep, physical activity, heart rate, caffeine, screen time, diet), motivating multivariable modeling while monitoring collinearity.

---

## Feature Engineering
Guided by correlation patterns and domain logic, we created:

- **Composite index:**  
  - **`Cardio Stress Index`** = (Heart Rate × Breathing Rate) / Sleep Hours, then **MinMax‑scaled** (fitted on train).  
- **Binary risk flags:**  
  - `is_low_sleep`, `is_high_caffeine`, `is_high_alcohol`, `is_low_activity`, and a cumulative **`multiple_lifestyle_risks`**.  
- **Interactions:**  
  - `PhysicalActivity_AgeGroup` (Physical Activity × Age Group),  
  - `therapy_sweating` (Therapy Sessions × Sweating Level),  
  - others such as `therapy_diet_ratio`, `alcohol_x_smoking`, and simple AND‑style indicators (e.g., `med_and_dizzy`).  
- **Encodings:**  
  - Categorical text → numeric encodings (one‑hot or ordinal where meaningful).  
- **Leakage control:**  
  - Dropped raw categorical duplicates after encoding and removed **`Stress Level (1–10)`** as a likely proxy for the target.

---

## Modeling
- **Split:** 70/30 train–test (random_state=42).  
- **Metric:** Root Mean Squared Error (RMSE); we also report R².  
- **Models evaluated:**
  - **Linear:** Multiple Linear Regression, **Ridge**, **Lasso** (α tuned via CV).  
  - **Tree:** Decision Tree (with pruning).  
  - **Ensembles:** **Random Forest** (tuned over `n_estimators`, `max_depth`) and **XGBoost**.

> Cross‑validation used 5 folds; scaling/encoding was fit on **train only** (or via pipelines) to prevent leakage.

---

## Results (Test Set)
| Model              | RMSE | Notes|
|--------------------|:----:|------|
| Linear Regression  | 1.52 | Baseline linear fit |
| Ridge              | 1.50 | CV over α |
| Lasso              | 1.49 | CV over α |
| Decision Tree (pruned) | 1.46 | — | Single interpretable non‑linear model |
| **Random Forest (depth 7)** | **1.39** | **0.57** | Best RMSE; robust to non‑linearities/interactions |
| XGBoost            | 1.41 | —     | Competitive with RF; higher compute than shallow RF |

**Efficiency note.** A **Random Forest (depth 5)** achieved RMSE ≈ **1.41**, on par with XGBoost but with lower computational cost.  

**Top signals** (feature importance, indicative): `cardio_stress_index`, `therapy_sweating`, `therapy_diet_ratio`, `caffeine_intake`.

---

## Limitations & Next Steps
- **Synthetic data** limits clinical generalization; treat results as illustrative.  
- Extend to **real‑world longitudinal data** (time‑varying habits → anxiety trajectories).  
- Refine interaction terms; add **demographic/biological context**; evaluate fairness.  
- Consider **calibration**, **uncertainty**, and **cost‑sensitive** thresholds for screening.

## Citation / Data
- Kaggle input page: https://www.kaggle.com/code/enesfiliz/social-anxietyand-lifestyle-analysis/input  
- Please note usage terms on the Kaggle page.
