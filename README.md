This report outlines the data cleaning process performed on the `world_layoffs` dataset. The primary goal was to transform raw, unorganized data into a structured and reliable format suitable for Exploratory Data Analysis (EDA).

---

## 1. Project Overview

The data cleaning project followed a structured four-step methodology to ensure data integrity and consistency. A staging table (`layoffs_staging`) was created initially to protect the raw data, followed by a second staging table (`layoffs_staging2`) to facilitate row removal operations.

### Key Objectives:

* Identify and remove duplicate entries.
* Standardize inconsistent naming conventions.
* Fix data types (specifically for dates).
* Handle null and irrelevant data points.

---

## 2. Data Cleaning Process

### Phase 1: Duplicate Removal

Duplicates were identified using the `ROW_NUMBER()` window function, partitioning across all major columns (Company, Industry, Date, etc.).

* **The Problem:** Standard `DELETE` statements are often restricted when using CTEs in certain SQL environments.
* **The Solution:** A new table `layoffs_staging2` was created with an additional column, `row_num`. All rows where `row_num > 1` were deleted, ensuring each layoff event is recorded only once.

### Phase 2: Data Standardization

This phase addressed inconsistencies that would skew analysis results.

* **Industry Cleanup:** Various entries for "Crypto" (e.g., 'Crypto Currency', 'CryptoCurrency') were consolidated into a single 'Crypto' label.
* **Handling Blanks:** Empty strings in the Industry column were converted to `NULL` to allow for programmatic filling.
* **Self-Join Imputation:** A self-join was used to populate missing Industry values by matching companies that had the information in other rows (e.g., filling in "Travel" for missing Airbnb rows).
* **Country Names:** Trailing punctuation was removed from country names (specifically fixing "United States.").

### Phase 3: Date Formatting

The date column was originally stored as a `TEXT` format, which prevents time-series analysis.

* The `STR_TO_DATE()` function was applied to convert strings into a standard `YYYY-MM-DD` format.
* The column type was permanently altered from `TEXT` to `DATE`.

### Phase 4: Null Handling and Final Trimming

The final step involved removing data that provided no analytical value.

* **Row Deletion:** Rows where both `total_laid_off` and `percentage_laid_off` were `NULL` were deleted, as these rows did not provide actionable data regarding the scale of layoffs.
* **Column Cleanup:** The temporary helper column `row_num` was dropped to finalize the table schema.

---

## 3. Key Findings & Results

| Improvement Area | Action Taken | Benefit |
| --- | --- | --- |
| **Integrity** | Removed exact duplicates. | Prevents overcounting layoff figures. |
| **Consistency** | Standardized "Crypto" and "United States". | Ensures accurate grouping in aggregate queries. |
| **Usability** | Converted strings to `DATE` objects. | Enables trend analysis over months and years. |
| **Quality** | Removed rows with no layoff metrics. | Cleanses the dataset for more accurate statistical modeling. |

---

