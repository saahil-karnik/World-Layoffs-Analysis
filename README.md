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
## Data Exploration Report: Global Layoffs Analysis

This report summarizes the findings from the Exploratory Data Analysis (EDA) conducted on the `world_layoffs` dataset. The analysis transitions from high-level metrics to granular insights regarding company performance, industry trends, and chronological patterns.

---

### 1. Key Performance Indicators (KPIs)

The initial exploration focused on identifying the scale of individual layoff events and the severity of company closures.

* **Maximum Single Event:** The largest single layoff event recorded in the dataset reached a peak that highlights the volatility of the tech sector during this period.
* **Total Liquidations:** Multiple companies reported a `percentage_laid_off` of **1 (100%)**, indicating complete shutdowns.
* **High-Stakes Failures:** Notable companies like **Quibi** and **BritishVolt** appear in the 100% layoff category despite having raised massive capital (upwards of $2 billion in Quibi's case), illustrating that high funding does not always guarantee sustainability.

---

### 2. Layoffs by Category

The data was aggregated to identify which sectors and regions bore the brunt of the economic shift.

#### **Top Entities and Locations**

* **Companies:** When aggregating total layoffs over the entire period, the "Big Tech" firms dominate the list, showing that while startups folded completely, large corporations shed the highest absolute number of employees.
* **Geography:** The **United States** consistently ranks as the country with the highest number of layoffs, with specific tech hubs (locations) showing concentrated job losses.
* **Industry:** The impact varied significantly across sectors, with industries like **Consumer** and **Retail** often appearing at the top of the list for total redundancies.

#### **Company Funding Stage**

The analysis shows a correlation between a company's "Stage" and its layoff volume. Post-IPO companies tend to have higher absolute numbers due to their size, while early-stage startups show higher frequency in the 100% layoff bracket.

---

### 3. Temporal Trends and Trajectory

Using advanced window functions and Common Table Expressions (CTEs), we identified how the layoff landscape evolved over time.

#### **Annual Leaders**

By ranking the top three companies for layoffs each year, we can see the shifting "epicenter" of the crisis:

* **Year-over-Year:** The companies appearing in the Top 3 change annually, suggesting that different sectors (e.g., Travel during the pandemic vs. Finance/AI shifts later) were impacted at different stages.

#### **The Rolling Total**

The most telling metric is the **Monthly Rolling Total**, which provides a "pulse" of the global economy.

* **Monthly Growth:** By extracting the month from the date strings, the analysis reveals specific "spike" months where layoffs accelerated globally.
* **Cumulative Impact:** The rolling total CTE demonstrates a steady, compounding increase in job losses, allowing stakeholders to see the aggregate human impact of the market downturn.

---

### 4. Summary of Methodology

The analysis was performed using several sophisticated SQL techniques to ensure data integrity and depth:

* **CTEs (Common Table Expressions):** Used to break down complex logic, such as calculating yearly rankings and monthly aggregations.
* **Window Functions:** `DENSE_RANK()` was utilized to compare companies within specific years, and `SUM() OVER()` was used to create the rolling progression of layoffs.
* **Data Cleaning Assumptions:** The script filters for `NULL` values in critical fields like `percentage_laid_off` and `years` to ensure the insights are based on actionable data.

---

