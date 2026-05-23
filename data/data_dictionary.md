# Data Dictionary — Yranigami Town Census 2025

This document describes the structure, data types, and quality issues present in the raw census dataset used for this project. The full dataset contains **11,296 records** across **12 columns**. The `sample_raw.csv` file in this folder contains 25 rows drawn from the original, preserving the exact structure and representative examples of each data quality issue.

> **Note:** All data is entirely synthetic. Yranigami Town is fictional. No real individuals are represented.

---

## Column Reference

### `Unnamed: 0`
- **Type:** Integer
- **Description:** Auto-generated row index from the CSV export. No analytical value.
- **Action taken:** Dropped immediately on load (`df.drop("Unnamed: 0", axis=1)`).

---

### `House Number`
- **Type:** Mixed — mostly integer, some string
- **Description:** The street number of the property.
- **Issues found:**
  - Some entries recorded as English words rather than digits (e.g. `"Five"`, `"seventy one"`).
  - No missing values, but word-format entries caused the column to be read as `object` dtype.
- **Action taken:** Attempted integer coercion first; word entries converted using `word2number` (`w2n.word_to_num()`), then cast to `int`.

---

### `Street`
- **Type:** String
- **Description:** Name of the street. Used in combination with `House Number` to identify unique households.
- **Issues found:** None. Clean and consistent across the dataset.

---

### `First Name`
- **Type:** String
- **Description:** Individual's first name.
- **Issues found:** None analytical. Not used in analysis — retained for household-level inspection only.

---

### `Surname`
- **Type:** String
- **Description:** Individual's surname. Critical for household grouping — members of the same household sharing a surname were used to infer relationships, impute missing gender, and validate age gaps.
- **Issues found:** None structural. A small number of lodgers shared a surname with the Head of House by coincidence, which required careful handling in the relationship imputation step.

---

### `Age`
- **Type:** Mixed — mostly integer, some string, some null
- **Description:** Individual's age in years.
- **Issues found:**
  - Some entries recorded as English words (e.g. `"seventy one"`).
  - Null values present.
  - **Logical anomalies:** One Head of House recorded as 15 years old (below legal minimum). Multiple parent-child pairs with a gap of less than 14 years (biologically implausible).
- **Action taken:**
  - Numeric coercion applied first; word-format entries converted via `word2number`.
  - 15-year-old Head corrected to 18 (legal minimum).
  - Impossible parent-child gaps corrected by sampling a new age from the mean and standard deviation of all *valid* parent-child age gaps in the dataset — not a fixed replacement value.
  - Remaining nulls imputed by household median where available.

---

### `Relationship to Head of House`
- **Type:** String (categorical)
- **Description:** The individual's relationship to the Head of household.
- **Expected values:** `Head`, `Wife`, `Husband`, `Partner`, `Son`, `Daughter`, `Grandson`, `Granddaughter`, `Lodger`, `Visitor`, `Adopted Son`, `Adopted Daughter`, `Nephew`, `Niece`
- **Issues found:**
  - Misspellings present (e.g. `"neice"` → `"Niece"`).
  - Missing values for some household members.
- **Action taken:** Misspellings corrected via direct mapping. Missing values inferred by cross-referencing surname matches within the same address and age relative to the Head; defaulted to `"Lodger"` for unresolvable adult non-family members.

---

### `Marital Status`
- **Type:** String (categorical), mixed format
- **Description:** Individual's marital status.
- **Expected values:** `Married`, `Single`, `Divorced`, `Widowed`, `N/A` (for under-18s)
- **Issues found:**
  - Abbreviations used inconsistently throughout (e.g. `"M"` for Married, `"S"` for Single, `"D"` for Divorced, `"W"` for Widowed).
  - Some individuals under 18 recorded with adult marital statuses.
  - Null values present.
- **Action taken:** Abbreviations expanded via dictionary mapping. Individuals under 18 reassigned `"N/A"`. Remaining nulls conservatively filled as `"Single"`.

---

### `Gender`
- **Type:** String (categorical), mixed format
- **Description:** Individual's gender.
- **Expected values:** `Male`, `Female`
- **Issues found:**
  - Abbreviations present (`"M"` for Male, `"F"` for Female).
  - Null values present.
- **Action taken:** Abbreviations expanded. Missing values inferred from `Relationship to Head of House` where possible (e.g. `"Wife"` → `Female`, `"Husband"` → `Male`; `"Partner"` → opposite of Head's gender). Remaining nulls filled by random sampling constrained to the dataset's overall gender distribution.

---

### `Occupation`
- **Type:** String (free text)
- **Description:** Individual's stated occupation. Highly varied — includes specific job titles, `"Student"`, `"Unemployed"`, `"Retired [role]"`, and blank entries.
- **Issues found:**
  - Missing values for minors, who were not assigned occupations in the raw data.
  - High cardinality — hundreds of distinct values — making direct grouping impractical.
- **Action taken:**
  - Individuals under 5 assigned `"Child"`.
  - Individuals aged 5–17 assigned `"Student"`.
  - A new engineered column `Job Type` was created, classifying occupations into categories (`High Income`, `Manual/High Risk`, `Public Sector`, `Student`, `Unemployed`, `Retired`, etc.) for use in the economic impact scoring matrix.

---

### `Infirmity`
- **Type:** String (categorical), near-entirely null
- **Description:** Any recorded physical or mental health condition.
- **Null rate:** ~99.3% of records (11,214 of 11,296 rows null in the raw data). This is expected — most individuals have no recorded infirmity.
- **Issues found:** None beyond sparsity. The non-null values were used in the social needs scoring for the decision matrix.
- **Action taken:** No imputation — null correctly represents the absence of a recorded condition.

---

### `Religion`
- **Type:** String (categorical), high fragmentation
- **Description:** Individual's stated religious affiliation.
- **Null rate:** ~59% of records (6,688 of 11,296 rows null in the raw data).
- **Issues found:**
  - Synonym fragmentation (e.g. `"Catholic"` and `"Roman Catholic"` treated as distinct categories in raw data).
  - Noise entries representing joke responses (e.g. `"Jedi"`, consistent with the phenomenon documented in the 2001 UK census).
  - Null values especially prevalent among children and young adults.
- **Action taken:** Synonyms consolidated into canonical categories. Noise entries flagged and excluded from infrastructure analysis. Missing values for minors imputed from the religion of the Head of House, or from another household member sharing the same surname.

---

## Summary of Data Quality Issues

| Column | Issue Type | Frequency |
|---|---|---|
| House Number | Word-format entries | Low |
| Age | Word-format, nulls, logical anomalies | Moderate |
| Relationship to Head | Misspellings, nulls | Low |
| Marital Status | Abbreviations, nulls, logical mismatches | Moderate |
| Gender | Abbreviations, nulls | Low |
| Occupation | Nulls (minors) | Moderate |
| Infirmity | Structural sparsity | High (by design) |
| Religion | Synonym fragmentation, noise entries, nulls | High |

---

## Dataset Provenance

This dataset was provided as part of a university assessment for the Fundamentals of Data Science module, MSc Artificial Intelligence & Data Science, University of Hull (2025). All data is entirely synthetic. The town of Yranigami is fictional and any resemblance to real places or individuals is coincidental.
