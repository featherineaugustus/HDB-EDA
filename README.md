# 🏠 Singapore HDB Resale Price Prediction

## 📊 Project Overview

This project uses historical HDB resale transaction data (1990–2020) to build a regression model that predicts resale prices. The goal is to find effective preprocessing techniques and use a simple linear regression model to forecast prices accurately.

---

## 🗂️ Data Loading

- **Source**: Multiple CSV files with the same schema were concatenated.
- **Columns**:
  - *Text-based*: `town`, `flat_type`, `street_name`, `storey_range`, `flat_model`, `remaining_lease`
  - *Numeric*: other fields
- **Dropped Features**: `block`, `street_name` — too specific and not helpful for general trends.

---

## 🧹 Data Preprocessing

| Feature                 | Preprocessing                                                                 | Comments                                                                                           |
|------------------------|--------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| `resale_price`         | Divided by 10,000                                                               | Scales target values for better modeling                                                           |
| `flat_model`           | Lowercased; tested one‑hot & label encoding                                     | Encoding reduced performance; lowercasing kept                                                    |
| `storey_range`         | Converted to average; binned into Low (<5), Mid (5–9), High (≥10)               | Using min or max storey didn’t improve results                                                    |
| `remaining_lease`      | Converted to months; computed `flat_age` from `lease_commence_date`            | Normalization didn’t improve results                                                              |
| `flat_type`            | Removed stray hyphens for cleanliness                                           |                                                                                                    |
| `town`                 | Tested regional grouping (East, North, NE, Central, West)                       | Grouping worsened model performance                                                               |
| `year`, `month`        | Extracted from transaction date; tested cyclic month encoding                   | Did not significantly affect performance                                                          |

---

## 📆 Data Splitting

- Time-aware split: older data used for training, latest (2019–2020) for test.
- Training spans varied (e.g., 1990–2018, 2000–2018, 2010–2018, …).
- Finding: More recent training data leads to better test performance.

---

## 🔄 Model Pipeline

1. **Categorical Encoding**:
   - One-hot encode: `flat_type`, `flat_model`, `storey_range_avg`, `region`
   - Label encoding tested, but one-hot performed better
2. **Numerics**:
   - `month`, `floor_area_sqm`, `storey_range_avg`, `remaining_lease_month`, `flat_age`
   - Standard scaling applied
3. **Pipeline Construction**:
   - Numeric features → `StandardScaler`
   - Categorical features → `OneHotEncoder`

---

## 🤖 Model & Training

- Model: **Linear Regression** (fast and effective under time constraints)
- Alternative models considered: Random Forest, XGBoost — deemed too heavy for high-dimensional, one-hot encoded data
- Parameter tuning via `GridSearchCV`

---

## 📈 Performance Summary

| Training Range | Test Range     | R² (Train) | R² (Test) | MAE   | MSE   |
|----------------|----------------|------------|-----------|-------|-------|
| 1990–2018      | 2019–2020      | 0.666      | 0.490     | 0.770 | 1.196 |
| 2000–2018      | 2019–2020      | 0.618      | 0.562     | 0.724 | 1.028 |
| 2010–2018      | 2019–2020      | 0.784      | 0.821     | 0.488 | 0.420 |
| 2015–2018      | 2019–2020      | 0.864      | 0.854     | 0.446 | 0.343 |
| 2017–2018      | 2019–2020      | 0.868      | 0.857     | 0.444 | 0.336 |
| 2018           | 2019–2020      | 0.869      | 0.859     | 0.439 | 0.330 |

**Conclusion**: The more recent the training data, the better the model forecasts on newer trends.

---

## 🧠 Feature Importance

- **Location (`town`)**: most impactful predictor
  - Premium areas (e.g., Bukit Timah, Marine Parade) have high positive coefficients
  - Peripheral estates (e.g., Sembawang, Choa Chu Kang) have negative impact
- **Flat Model**: certain types (e.g., Terrace, Type S2) significantly increase price
- **Additional Factors**: floor area, lease age, storey do affect price, though less strongly

---

## ✅ Conclusion

- Latest data significantly enhances model performance
- **Location** is the strongest determinant of resale price
- Simple models with robust preprocessing can perform well
- Potential extensions:
  - Use more advanced regressors (e.g., RF, XGBoost)
  - Integrate new features (e.g., proximity to amenities)

---

## 📂 File Structure
```bash
Kaggle_HDB/
├── data/
│ └── *.csv # Raw and concatenated data files
├── model/
│ └── regressor_lr.pkl # Trained model file
├── notebook.html # Complete preprocessing & training pipeline
└── results/ # Performance visuals, logs, coefficients
```