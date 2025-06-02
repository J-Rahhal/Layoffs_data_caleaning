# 🧹 Data Cleaning: Preparing a Raw Dataset

This project focuses solely on **cleaning a raw, messy dataset**. No analysis or modeling has been done—only data preparation.

---

## 🎯 Goals of Cleaning

The goal was to clean the dataset to make it usable for future analysis. Here's what we did:

1. **Remove Duplicates**
   - Checked for duplicate rows and removed them to ensure data integrity.

2. **Standardize the Data**
   - Trimmed leading/trailing spaces from text fields (e.g., company names, countries).
   - Unified similar category values (e.g., "Crypto Currency" → "Crypto").
   - Converted date columns from text format to proper `DATE` format.

3. **Handle Null or Blank Values**
   - Identified columns with missing or empty values.
   - Replaced blank strings with `NULL` values for consistency.
   - Attempted to populate missing values using information from similar rows (e.g., same company and location).

4. **Remove Unnecessary Columns or Rows**
   - Removed rows where key data (e.g., layoffs info) was completely missing.
   - Dropped or ignored any irrelevant columns for this cleaning phase.

---

## 📁 Output

The cleaned dataset is now consistent, standardized, and ready for further exploration or analysis.

---

> This cleanup process is essential to ensure the accuracy and reliability of any future data work.
