# Demographic-Data-Insights
- Objective: To identify the primary socio-economic factors that correlate with high-income brackets within the 1994 Census database.

- Key Finding: Our model identified that 'Education Level' and 'Age' were the two strongest predictors of income, with an F1-score of 0.84.

- The Story: A local government has an unoccupied plot of land and a constrained budget. They need two things: a decision on what to build, and a decision on where to invest their service budget. They have raw census data for 11,296 residents. That data is messy. This project cleans it, analyses the demographic structure, projects it forward 10 years, and delivers evidence-based recommendations using a weighted decision matrix.
This is what data science looks like when the output isn't just a chart — it's a recommendation a planning department could actually act on.

## What I built

### Data Cleaning Pipeline

The raw dataset had problems across almost every column. I built a multi-stage cleaning pipeline to fix them systematically rather than ad hoc:

- **House numbers** — string entries like "One" were converted using `word2number`, then coerced to integers
- **Ages** — numeric coercion, then a relational inspection pass to catch impossible parent-child age gaps (less than 14 years). Gaps were corrected by sampling from the mean and standard deviation of all valid gaps in the dataset — not a hardcoded replacement
- **Head of House logic** — a 15-year-old Head was flagged and corrected to the legal minimum. The inspection function worked at household level, not just row level, so anomalies were caught in context
- **Relationships** — misspellings corrected, missing values imputed by cross-referencing surname matches within the same address
- **Marital status** — abbreviations expanded, under-18s assigned N/A, remaining nulls conservatively filled as Single
- **Gender** — inferred from relationship type where possible (e.g. Partner → opposite of Head's gender), remainder drawn from the overall distribution
- **Occupation** — minors under 5 assigned Child, 5–17 assigned Student. A new `Job Type` column was engineered to classify occupations for economic impact scoring
- **Religion** — high fragmentation from synonyms (Catholic/Roman Catholic) and noise entries (Jedi) standardised into coherent categories; minors imputed from Head of House

The inspection function was the most useful tool I wrote. Instead of treating each missing value as an isolated problem, it let me see the full household before deciding what to do. That relational view changed how I handled several edge cases.

### Exploratory Data Analysis

A 3×3 visualisation suite covering missing data patterns, age outlier detection, household occupancy, relationship type distribution, religion fragmentation, top occupations, marital status, gender split, and infirmity breakdown. Built before cleaning to expose what actually needed fixing, not just what was null.

### Demographic Projection Model

A 10-year forward model to 2035:
- Ages incremented by 10
- Mortality applied to the 75+ cohort using national survival probabilities
- Births estimated from females aged 20–40 with an age-specific fertility rate

### Strategic Decision Matrix

Weighted 60% Social Need / 40% Economic Impact, drawing scores from the cleaned census metrics rather than assumptions. This is what resolved the tension between building a train station (high economic score) and a medical centre (high social need score).

## What I found

The population pyramid is **constrictive** — more older residents than younger, with a shrinking working-age base. The 65+ cohort is currently 1,480. The projection puts it at 2,146 by 2035, a **45% increase** in ten years.

Overcrowding is significant: **18.5%** of households exceed five occupants. Unemployment sits at **6.8%**. Only **7.1%** of the workforce is in high-income roles, which ruled out luxury housing as a viable primary investment. The school-age cohort is actually projected to *decrease* by **8.5%** over the next five years.

The decision matrix resolved both questions clearly:

| Option | Social Need (60%) | Economic Impact (40%) | Weighted Score |
|---|---|---|---|
| Medical Centre | 98 | 50 | **79** |
| High-Density Housing | 70 | 70 | 70 |
| Train Station | 40 | 95 | 62 |
| Religious Building | 65 | 20 | 47 |

**Build the Medical Centre. Invest in General Infrastructure.**

The train station scored highest economically but the current commuter population (12.3%) doesn't justify the need score. The medical centre addresses the town's most urgent and unavoidable risk. Infrastructure investment is a prerequisite for everything else — without upgrading water, waste, and road networks first, no other development is sustainable.

## What I learned

How to think about data cleaning as a relational problem, not a column-by-column checklist. Household records have internal logic — someone's age, relationship, occupation, and marital status all constrain each other. Treating each field in isolation misses that.

The decision matrix forced me to be explicit about what I was actually optimising for. It is easy to present numbers. It is harder to say: this is how I am weighing competing objectives, and here is why. That explains why explainability matters more in real-world deployment than model accuracy metrics.

Projection models are only as good as their assumptions. I documented what I assumed (fertility rates, mortality curves, migration) so the output is auditable, not just a number.

## Tech

Python, pandas, numpy, matplotlib, seaborn, scipy, word2number

## Files

`Census_Data_Analysis.ipynb` contains the full pipeline, including all outputs.

Dataset not included — university assessment data.

## Why it matters for sustainable planning

Urban planning is one of the most data-intensive domains in public policy. Every infrastructure decision is a 30-year commitment made on incomplete information. This project applies the same tools used in green city planning — demographic projection, resource allocation modelling, social impact weighting — to a concrete local government problem.

The overlap with sustainability is direct: ageing populations, housing density, transport infrastructure, and utilities planning are all components of long-term sustainable development. Getting the data right before making those decisions is the entire point.

---
