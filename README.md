# 🐼 Pandas Learning Journey

## Preface

This is a concise, practical reference for learning **Pandas** — Python's go-to library for working with tabular data. Each section covers one core concept with clean, minimal examples. No fluff, no duplicates — just the essential patterns you'll actually use when reading, exploring, cleaning, and visualizing data.

---

## 📋 Table of Contents

1. [Reading in Files](#1-reading-in-files)
2. [Filtering and Ordering](#2-filtering-and-ordering)
3. [Indexes in Pandas](#3-indexes-in-pandas)
4. [Group By and Aggregating](#4-group-by-and-aggregating)
5. [Merge, Join & Concatenate](#5-merge-join--concatenate)
6. [Data Cleaning](#6-data-cleaning)
7. [EDA in Pandas](#7-eda-in-pandas)
8. [Pandas Visualization](#8-pandas-visualization)

---

## 1. Reading in Files

> Pandas can read almost any file format. Use `pd.read_*` functions matching your file type. Use `set_option` to control how much data is displayed.

```python
import pandas as pd

# CSV
df = pd.read_csv("countries.csv")

# TXT with tab separator
df = pd.read_csv("countries.txt", sep='\t')

# JSON
df = pd.read_json("data.json")

# Excel — specify sheet
df = pd.read_excel("workbook.xlsx", sheet_name='Sheet1')

# Display settings
pd.set_option('display.max.rows', 235)
pd.set_option('display.max.columns', 40)

# Quick inspection
df.shape        # (rows, columns)
df.head(7)      # first 7 rows
df.tail(10)     # last 10 rows
df['Rank']      # single column
df.loc[224]     # row by label/index
df.iloc[224]    # row by position number
```

---

## 2. Filtering and Ordering

> Filter rows by condition, list membership, or string match. Sort by one or multiple columns with independent directions.

```python
import pandas as pd
df = pd.read_csv("world_population.csv")

# Filter by condition
df[df['Rank'] <= 10]

# Filter by list of values
countries = ['Bangladesh', 'Brazil']
df[df['Country'].isin(countries)]

# Filter by string pattern
df[df['Country'].str.contains('United')]

# Select specific columns or rows
df.filter(items=['Continent', 'CCA3'])           # keep 2 columns
df.filter(like='United', axis=0)                 # rows containing 'United'

# Sort — single column
df[df['Rank'] < 10].sort_values('Rank', ascending=False)

# Sort — multiple columns with mixed directions
df.sort_values(by=['Country', 'Rank'], ascending=[False, True])
```

---

## 3. Indexes in Pandas

> Indexes are row labels. You can set any column as the index, reset it back to default, and use multi-level (hierarchical) indexes for complex data.

```python
# Set index on load
df = pd.read_csv("world_population.csv", index_col='Country')

# Reset to default integer index
df.reset_index(inplace=True)

# Set index after loading
df.set_index('Country', inplace=True)

# Access rows
df.loc['Albania']     # by label (index name)
df.iloc[1]            # by position (always integer)

# Multi-level index
df.set_index(['Country', 'Continent'], inplace=True)
df.sort_index(ascending=[False, True])

# Multi-level lookup
df.loc['South America', 'Argentina']
```

> 💡 `loc` uses **labels** · `iloc` uses **integer positions** — always.

---

## 4. Group By and Aggregating

> Group rows by a category column, then apply aggregate functions to summarize each group.

```python
# Basic groupby + aggregate
df.groupby('Base Flavor').mean(numeric_only=True)
df.groupby('Base Flavor').count()
df.groupby('Base Flavor').sum(numeric_only=True)
df.groupby('Base Flavor').min()

# Multiple aggregations on one column
df.groupby('Base Flavor').agg({
    'Flavor Rating': ['mean', 'max', 'count', 'sum']
})

# Multiple aggregations on multiple columns
df.groupby('Base Flavor').agg({
    'Flavor Rating':  ['mean', 'max', 'count', 'sum'],
    'Texture Rating': ['mean', 'max', 'count', 'sum']
})

# Group by multiple columns
df.groupby(['Base Flavor', 'Liked']).mean(numeric_only=True)

# Full statistical summary per group
df.groupby(['Base Flavor', 'Liked']).describe()
```

---

## 5. Merge, Join & Concatenate

> Three ways to combine DataFrames. Use **merge** for SQL-style joins on columns, **join** for index-based joins, **concat** to stack DataFrames together.

```python
df1 = pd.read_csv("LOTR.csv")
df2 = pd.read_csv("LOTR2.csv")

# --- MERGE (column-based) ---
df1.merge(df2, how='inner', on=['FellowshipID', 'FirstName'])  # matching rows only
df1.merge(df2, how='left')    # all of df1, matched df2
df1.merge(df2, how='right')   # all of df2, matched df1
df1.merge(df2, how='outer')   # all rows from both
df1.merge(df2, how='cross')   # every combination

# --- JOIN (index-based) ---
df1.set_index('FellowshipID').join(
    df2.set_index('FellowshipID'),
    lsuffix='_Left', rsuffix='_Right', how='outer'
)

# --- CONCAT (stack side by side) ---
pd.concat([df1, df2], join='outer', axis=1)
```

| Type | Best for |
|---|---|
| `merge` | Joining on shared columns (like SQL) |
| `join` | Joining on indexes |
| `concat` | Stacking DataFrames vertically or horizontally |

---

## 6. Data Cleaning

> Real-world data is messy. This section covers the full cleaning pipeline: removing duplicates, stripping bad characters, splitting columns, standardizing values, handling nulls, and dropping unwanted rows.

```python
import pandas as pd
df = pd.read_excel("Customer Call List.xlsx")

# 1. Remove duplicate rows
df.drop_duplicates(inplace=True)

# 2. Drop a useless column
df.drop(columns='Not_Useful_Column', inplace=True)

# 3. Clean string columns — strip unwanted characters
df["Last_Name"] = df["Last_Name"].str.strip("123._/")

# 4. Fix phone numbers
df["Phone_Number"] = df["Phone_Number"].str.replace('[^a-zA-Z0-9]', '', regex=True)
df["Phone_Number"] = df["Phone_Number"].apply(
    lambda x: str(x)[0:3] + '-' + str(x)[3:6] + '-' + str(x)[6:10]
)
df["Phone_Number"] = df["Phone_Number"].str.replace('nan--', '').str.replace('N/a--', '')

# 5. Split one column into multiple
df[["Street_Address", "State", "Zip_Code"]] = df["Address"].str.split(',', n=2, expand=True)

# 6. Standardize values
df["Paying Customer"]  = df["Paying Customer"].str.replace('Yes', 'Y').str.replace('No', 'N')
df["Do_Not_Contact"]   = df["Do_Not_Contact"].str.replace('Yes', 'Y').str.replace('No', 'N')

# 7. Fill remaining nulls
df = df.fillna('')

# 8. Drop rows flagged as Do Not Contact
for x in df.index:
    if df.loc[x, "Do_Not_Contact"] == 'Y':
        df.drop(x, inplace=True)

# 9. Drop rows with no phone number
for x in df.index:
    if df.loc[x, "Phone_Number"] == '':
        df.drop(x, inplace=True)
# Alternative: df.dropna(subset=["Phone_Number"], inplace=True)

# 10. Reset index after dropping rows
df.reset_index(drop=True, inplace=True)

# 11. Export cleaned data
df.to_csv("Customer_Call_List_CLEANED.csv", index=False)
df.to_excel("Customer_Call_List_CLEANED.xlsx", index=False, sheet_name="Cleaned")
print("✅ CSV and Excel saved!")
```

---

## 7. EDA in Pandas

> Exploratory Data Analysis (EDA) means understanding your dataset before doing anything else — its shape, data types, missing values, distributions, and relationships between columns.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

df = pd.read_csv("world_population.csv")
pd.set_option('display.float_format', lambda x: '%.2f' % x)

# --- Core inspection ---
df.info()                        # column names, dtypes, non-null counts
df.describe()                    # count, mean, std, min, max per column
df.isnull().sum()                # count of nulls per column
df.nunique()                     # count of unique values per column

# --- Sort & explore ---
df.sort_values(by="World Population Percentage", ascending=False).head(10)

# --- Correlation matrix ---
df.corr(numeric_only=True)

# --- Heatmap of correlations ---
plt.rcParams['figure.figsize'] = (20, 7)
sns.heatmap(df.corr(numeric_only=True), annot=True)
plt.show()

# --- Group & compare ---
df.groupby('Continent').mean(numeric_only=True).sort_values(by="2022 Population", ascending=False)

# --- Trend over time (transpose for plotting) ---
df2 = df.groupby('Continent')[
    ['1970 Population','1980 Population','1990 Population',
     '2000 Population','2010 Population','2015 Population',
     '2020 Population','2022 Population']
].mean().sort_values(by="2022 Population", ascending=False)

df2.transpose().plot()
plt.show()

# --- Type-based selection & boxplot ---
df.select_dtypes(include='float')
df.boxplot(figsize=(20, 10))
plt.show()
```

---

## 8. Pandas Visualization

> Pandas has built-in plotting powered by Matplotlib. Set a style, then call `.plot()` with a `kind` argument — or use shorthand like `.plot.bar()`.

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("Ice Cream Ratings.csv")
df = df.set_index('Date')

# See all available styles
print(plt.style.available)
plt.style.use('classic')

# Line chart
df.plot(kind='line', title='Ice Cream Ratings', xlabel='Date', ylabel='Score')

# Horizontal bar (stacked)
df.plot.barh(stacked=True)

# Scatter plot
df.plot.scatter(x='Texture Rating', y='Overall Rating', s=500, c='yellow')

# Histogram
df.plot.hist(bins=10)

# Box plot
df.boxplot()

# Area chart
df.plot.area(figsize=(10, 5))

# Pie chart (single column)
df.plot.pie(y='Flavor Rating', figsize=(10, 10))

plt.show()
```

| Chart | Use when |
|---|---|
| `line` | Trends over time |
| `barh` | Comparing categories horizontally |
| `scatter` | Relationship between two numeric columns |
| `hist` | Distribution of a single column |
| `boxplot` | Spread & outliers across columns |
| `area` | Cumulative trends over time |
| `pie` | Part-to-whole for a single column |

---

## 💡 Quick Reference

| Task | Code |
|---|---|
| Read CSV | `pd.read_csv("file.csv")` |
| Read Excel | `pd.read_excel("file.xlsx", sheet_name='Sheet1')` |
| Filter rows | `df[df['col'] > value]` |
| Multi-filter | `df[df['col'].isin(['A','B'])]` |
| Sort | `df.sort_values(by='col', ascending=False)` |
| Set index | `df.set_index('col', inplace=True)` |
| Group & aggregate | `df.groupby('col').agg({'col2': ['mean','max']})` |
| Merge DataFrames | `df1.merge(df2, how='inner', on='key')` |
| Fill nulls | `df.fillna('')` |
| Drop column | `df.drop(columns='col')` |
| Reset index | `df.reset_index(drop=True)` |
| Export CSV | `df.to_csv("out.csv", index=False)` |
| EDA summary | `df.info()` · `df.describe()` · `df.isnull().sum()` |
| Plot | `df.plot(kind='line')` |

---

> 💬 *Each section is self-contained — jump to whichever concept you need. For real projects, you'll typically go: Read → Inspect → Clean → Explore → Visualize.*
