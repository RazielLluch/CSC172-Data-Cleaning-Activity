# StartUp Investments (Crunchbase) — Data Cleaning and Visualization

**Course:** CSC172 Data Mining and Analysis

**Institution:** Mindanao State University – Iligan Institute of Technology

**Student:** Josiah Raziel S. Lluch (2022-0834)

**Semester:** AY 2025–2026, Semester 1

---

## Overview

This project performs data cleaning and exploratory data visualization on the **Crunchbase startup VC investments dataset** (`investments_VC.csv`) from Kaggle. The dataset contains records of venture capital-funded startups with information on funding amounts, funding rounds, industries, geography, founding dates, and company status.

**Dataset Link** https://www.kaggle.com/datasets/arindam235/startup-investments-crunchbase/data

The notebook is split into two major phases:
1. **Data Cleaning** — preparing raw data for reliable analysis
2. **Data Visualization** — exploring patterns and insights through charts

---

## Dataset

| Property | Detail |
|---|---|
| File | `investments_VC.csv` |
| Encoding | `latin1` |
| Rows | ~54,000+ startup records |
| Columns | 38 (before cleaning) |

### Key Columns

| Column | Description |
|---|---|
| `name` | Startup name |
| `market` | Primary industry/market |
| `category_list` | All industry tags (pipe-delimited) |
| `funding_total_usd` | Total funding raised (USD) |
| `funding_rounds` | Number of funding rounds |
| `status` | Operating, acquired, or closed |
| `country_code` | Country of the startup |
| `state_code` | US state (where applicable) |
| `founded_at` | Date company was founded |
| `first_funding_at` | Date of first funding event |
| `last_funding_at` | Date of most recent funding event |
| `seed`, `round_A`–`round_H` | Amount raised per funding stage |

---

## Dependencies

**use uv sychronization to install dependencies from the pyrpoject.toml file**

```bash
uv sync
```

---

## Part 1: Data Cleaning

### Steps Performed

#### 1. Load Dataset
```python
df = pd.read_csv("investments_VC.csv", encoding="latin1")
```

#### 2. Strip Column Name Whitespace & Drop Useless Columns
The `permalink` and `homepage_url` columns were dropped as they hold no analytical value.
```python
df.columns = df.columns.str.strip()
df = df.drop(columns=["permalink", "homepage_url"])
```

#### 3. Drop Rows with Missing Values
All rows with blank values are dropped — **except** `state_code`, which is intentionally sparse since many non-US countries don't use a state system.
```python
df = df.dropna(subset=[c for c in df.columns if c != "state_code"])
```

#### 4. Fix `funding_total_usd`
Raw values are stored as formatted strings (e.g., `" 17,50,000 "`). These are stripped of whitespace and commas, dashes are replaced with `0`, and the column is cast to numeric.
```python
df['funding_total_usd'] = (
    df['funding_total_usd']
    .str.strip()
    .str.replace(',', '', regex=False)
    .str.replace('-', '0', regex=False)
    .str.strip()
)
df['funding_total_usd'] = pd.to_numeric(df['funding_total_usd'], errors='coerce')
```

#### 5. Strip Whitespace from `market`
```python
df['market'] = df['market'].str.strip()
```

#### 6. Process `category_list`
Categories are stored as a pipe-delimited string (e.g., `|Software|SaaS|`). Leading/trailing pipes are stripped, then the field is split into a Python list.
```python
df['category_list'] = df['category_list'].str.strip('|')
df['category_list'] = df['category_list'].str.split('|')
```

#### 7. Parse Date Columns
Three date columns are converted from strings to proper `datetime` objects.
```python
df['founded_at'] = pd.to_datetime(df['founded_at'], errors='coerce')
df['first_funding_at'] = pd.to_datetime(df['first_funding_at'], errors='coerce')
df['last_funding_at'] = pd.to_datetime(df['last_funding_at'], errors='coerce')
```

#### 8. Flag Corrupt Dates
Rows where `founded_at` is before the year 1900 are identified as likely corrupted data.
```python
df[df['founded_at'].dt.year < 1900]
```

#### 9. Handle Remaining Missing Values
`status` NaNs are filled with `'unknown'`; rows missing `funding_total_usd` are dropped.
```python
df['status'].fillna('unknown', inplace=True)
df.dropna(subset=['funding_total_usd'], inplace=True)
```

#### 10. Summary Check
```python
df.info()
df.isnull().sum()
df.describe()
```

---

## Part 2: Data Visualization

All visualizations use **Matplotlib** and **Seaborn** with the `whitegrid` theme.

### Chart 1 — Distribution of Total Funding (Histogram)
Displays the spread of funding amounts on a **log10 scale** to handle the extreme range of values (from thousands to hundreds of millions of dollars).

### Chart 2 — Top 10 Markets by Startup Count (Horizontal Bar)
Shows which industries have the highest number of VC-backed startups in the dataset.

### Chart 3 — Top 10 Countries by Startup Count (Bar Chart)
Compares startup counts across the top 10 countries by `country_code`.

### Chart 4 — Total VC Funding by Year Founded (Line Chart)
Plots the sum of total funding grouped by the year a startup was founded, filtered to 1990–2015 to remove outliers.

### Chart 5 — Startup Status Distribution (Pie Chart)
Breaks down the share of startups by their current status: operating, acquired, or closed.

### Chart 6 — Total Funding per Round Type (Bar Chart)
Compares total capital raised across funding stages: Seed, Round A, B, C, D, E, and F.

### Chart 7 — Top 15 Markets by Average Funding (Horizontal Bar)
Highlights which markets attract the largest *average* investment, as opposed to the most startups.

### Chart 8 — Funding Rounds vs. Total Funding (Scatter Plot)
Explores the relationship between the number of funding rounds and total funding raised (log scale), using a 2,000-row random sample to reduce overplotting.

---

## File Structure

```
├── investments_VC.csv       # Raw dataset
├── data_cleaning.ipynb      # Notebook (source, no outputs)
└── README.md                # This file
```
