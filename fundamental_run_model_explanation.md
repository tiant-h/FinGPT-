# Code Structure Explanation: fundamental_run_model.py

## Overview
This script is a **machine learning pipeline** for stock market prediction using fundamental data. It trains multiple ML models to predict stock returns for a specific sector.

---

## Code Flow (Step by Step)

### **Step 1: Setup & Imports** (Lines 1-10)
- Suppresses warnings
- Imports necessary libraries (pandas, numpy, time, etc.)
- Imports the `ml_model` module which contains the actual ML training functions

### **Step 2: Command Line Arguments** (Lines 15-38)
The script accepts parameters when you run it:
- **Required:**
  - `-sector_name`: Name of the sector (e.g., "sector10")
  - `-fundamental`: Path to fundamental data CSV file
  - `-sector`: Path to sector-specific Excel file
  
- **Optional (with defaults):**
  - `-first_trade_index`: Starting point for backtesting (default: 20)
  - `-testing_window`: Size of testing window in quarters (default: 4)
  - Column names for date, ticker, label, etc.

### **Step 3: Load Data** (Lines 39-54)
- Loads **fundamental data** from CSV file
- Loads **sector-specific data** from Excel file
- Extracts:
  - Unique dates (sorted)
  - Unique stock tickers in the sector

### **Step 4: Configure Rolling Window** (Lines 56-67)
- Sets up **rolling window** for time-series backtesting:
  - Training: 4 years (16 quarters)
  - Testing: 1 year (4 quarters)
  - First trade date starts at index 20 (quarter 20)
- Creates list of all trade dates for backtesting

### **Step 5: Feature Selection** (Lines 68-75)
- Identifies which columns are **features** (predictors):
  - Excludes non-feature columns (like ticker names, dates, IDs)
  - Only includes numeric columns
  - Excludes columns with NaN values

### **Step 6: Run ML Models** (Lines 80-94)
- Calls `ml_model.run_4model()` which:
  1. Trains 3 ML models (Random Forest, XGBoost, LightGBM)
  2. Uses rolling window approach (train on past, test on recent, predict for trading)
  3. Selects the best model for each time period based on test performance
  4. Generates predictions for all stocks at each trade date
- Measures execution time
- Saves results to files

### **Step 7: Error Handling** (Lines 95-97)
- Catches and prints any errors that occur during execution

---

## Key Concepts

### **Rolling Window Strategy**
```
Time:  [Q1] [Q2] ... [Q16] [Q17] [Q18] [Q19] [Q20] [Q21] [Q22] [Q23] [Q24] ...
       └────────── Train ──────────┘ └─ Test ─┘ └─ Trade ─┘
       
At Q20: Train on Q1-Q16, Test on Q17-Q19, Predict for Q20
At Q21: Train on Q2-Q17, Test on Q18-Q20, Predict for Q21
...and so on
```

### **What the Script Does**
1. **Input**: Historical fundamental data for stocks in a sector
2. **Process**: 
   - Trains ML models using rolling window
   - Tests model performance
   - Selects best model
   - Predicts future returns
3. **Output**: CSV files with predicted returns for each stock at each trade date

---

## Example Usage
```bash
python3 fundamental_run_model.py \
  -sector_name sector10 \
  -fundamental Data/fundamental_final_table.xlsx \
  -sector Data/1-focasting_data/sector10_clean.xlsx
```

---

## Output Files
Results are saved in `results/{sector_name}/`:
- `df_predict_rf.csv`: Random Forest predictions
- `df_predict_xgb.csv`: XGBoost predictions  
- `df_predict_gbm.csv`: LightGBM predictions
- `df_predict_best.csv`: Best model predictions (selected dynamically)
- `df_best_model_name.csv`: Which model was best at each date
- `df_model_score.csv`: Model evaluation scores

---

## Summary
This is a **backtesting framework** that:
- Uses fundamental financial data to predict stock returns
- Trains multiple ML models using time-series cross-validation
- Automatically selects the best-performing model
- Generates trading signals (predicted returns) for portfolio construction
