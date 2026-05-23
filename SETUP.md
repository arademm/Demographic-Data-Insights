# Setup

Tested on Python 3.10+. Should work on any modern Python 3 environment.

## Quick start

```bash
git clone https://github.com/<your-username>/census-demographic-analysis.git
cd census-demographic-analysis
pip install -r requirements.txt
jupyter notebook Census_Data_Analysis.ipynb
```

Run all cells in order from top to bottom.

---

## Dependencies

Install everything at once:

```bash
pip install pandas numpy matplotlib seaborn scipy word2number jupyter
```

Or using the requirements file:

```bash
pip install -r requirements.txt
```

| Package | Purpose |
|---|---|
| `pandas` | Data loading, cleaning, manipulation |
| `numpy` | Numerical operations, random sampling for imputation |
| `matplotlib` | Base plotting |
| `seaborn` | Statistical visualisations (heatmaps, distribution plots) |
| `scipy` | Statistical testing (Z-tests, T-tests) |
| `word2number` | Converting word-format entries like `"seventy one"` to integers |
| `jupyter` | Running the notebook |

`word2number` is the non-obvious one. The notebook installs it automatically if it isn't found, but installing it upfront avoids a restart mid-run.

---

## Virtual environment (recommended)

Using `venv`:

```bash
python -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows
pip install -r requirements.txt
jupyter notebook Census_Data_Analysis.ipynb
```

Using `conda`:

```bash
conda create -n census-analysis python=3.10
conda activate census-analysis
pip install -r requirements.txt
jupyter notebook Census_Data_Analysis.ipynb
```

---

## Data files

The full dataset is not included in this repository. The `data/` folder contains:

- `sample_raw.csv` — 25 representative rows showing the original structure and data quality issues
- `data_dictionary.md` — full column reference including types, issues, and cleaning actions taken

To run the notebook against the sample only, update the filename in the data loading cell:

```python
# Change this line in the notebook:
filename = 'T1_A25census-4.csv'

# To this:
filename = 'data/sample_raw.csv'
```

The cleaning pipeline will run on the sample data. Visualisations and projection outputs will differ from the full analysis given the reduced row count.

---

## Repo structure

```
census-demographic-analysis/
├── Census_Data_Analysis.ipynb   ← Full pipeline with outputs
├── README.md
├── SETUP.md                     ← You are here
├── requirements.txt
├── data/
│   ├── sample_raw.csv           ← 25-row sample of the original dataset
│   └── data_dictionary.md       ← Column reference and quality issue log
└── .gitignore
```

---

## Troubleshooting

**`ModuleNotFoundError: No module named 'word2number'`**
The notebook handles this automatically, but if it fails: `pip install word2number` then restart the kernel before re-running.

**`FileNotFoundError: T1_A25census-4.csv not found`**
The full dataset isn't included. Either point the notebook at `data/sample_raw.csv` (see above) or place your own copy of the CSV in the root directory.

**Plots not displaying inline**
Add `%matplotlib inline` to the top of the first code cell, or launch with `jupyter notebook` rather than `jupyter lab` if rendering looks off.
