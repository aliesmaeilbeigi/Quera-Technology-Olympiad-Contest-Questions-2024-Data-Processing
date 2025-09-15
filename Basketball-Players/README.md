# NBA Players Analysis (2000–2009) 🏀📊

## 📄 Other Languages
- 🇮🇷 [نسخه فارسی (Persian)](README_fa.md)

## Overview

This project analyzes player performance in the NBA across ten seasons (2000–2009). The goal is to explore individual and team statistics, identify standout players, and derive insights from the dataset provided.

The dataset contains **1,830 rows** and **18 columns**, with each row representing a player's performance in a given season.

---

## Dataset Description

The dataset `NBA2000-2009.csv` includes the following columns:

| Column | Description                            |
| :----: | :------------------------------------- |
| PLAYER | Player's name                          |
|  TEAM  | Team abbreviation                      |
|  YEAR  | Season year                            |
|   GP   | Games played                           |
|   MIN  | Average minutes played                 |
|   PTS  | Average points scored                  |
|   FGM  | Average successful 2-point field goals |
|   FGA  | Average attempted 2-point field goals  |
|   3PM  | Average successful 3-point field goals |
|   3PA  | Average attempted 3-point field goals  |
|   FTM  | Average successful free throws         |
|   FTA  | Average attempted free throws          |
|  OREB  | Average offensive rebounds             |
|  DREB  | Average defensive rebounds             |
|   AST  | Average assists                        |
|   STL  | Average steals                         |
|   BLK  | Average blocks                         |
|   TOV  | Average turnovers                      |

The dataset is imported using Python’s `pandas` library:

```python
import pandas as pd
df = pd.read_csv("../data/NBA2000-2009.csv")
df.head()
```

---

## Part 1: Double-Doubles

**Objective:** Identify players who achieved a "double-double" in at least **2 of 5 key statistics** during a season.
**Key Statistics:** Points (PTS), Rebounds (REB = OREB + DREB), Assists (AST), Blocks (BLK), Steals (STL).

**Output:** A dataframe `double_double` containing:

* PLAYER
* YEAR
* PTS
* AST
* REB
* BLK
* STL

The dataframe is sorted by `YEAR` and `PLAYER` in ascending order. Example rows:

| PLAYER          | YEAR |  PTS | AST |  REB | BLK | STL |
| --------------- | :--: | :--: | :-: | :--: | :-: | :-: |
| Antonio Davis   | 2000 | 13.7 | 1.4 | 10.1 | 1.9 | 0.3 |
| Antonio McDyess | 2000 | 20.8 | 2.1 | 12.0 | 1.5 | 0.6 |

**Code:**

```python
# create total rebounds and a boolean count for >=10 in each stat
df['REB'] = df['DREB'] + df['OREB']
df['mb10'] = ((df["PTS"] >= 10).astype(int) +
              (df["REB"] >= 10).astype(int) +
              (df["AST"]>=10).astype(int) +
              (df["BLK"] >= 10).astype(int) +
              (df["STL"]>=10).astype(int))

# select players with at least 2 categories >=10
double_double = (df[df['mb10'] >= 2][['PLAYER','YEAR','PTS','AST','REB','BLK','STL']]
                 .sort_values(['YEAR','PLAYER'])
                 .reset_index(drop=True))
```

---

## Part 2: Most Valuable Players

**Objective:** Identify the most valuable players (MVPs) over the ten seasons.

**Player Value Formula:**

```
VALUE = (PTS + OREB + DREB + AST + STL + BLK) - (TOV + Missed FG + Missed FT)
```

Where:

* `Missed FG` = FGA − FGM
* `Missed FT` = FTA − FTM

The overall value of a player is the **average value across all seasons**.

**Output:** `best_player` dataframe with:

* PLAYER
* VALUE (rounded to 2 decimals)

Sorted by VALUE descending and then PLAYER ascending. Example:

| PLAYER        | VALUE |
| ------------- | ----- |
| Kevin Garnett | 29.74 |
| LeBron James  | 27.90 |

**Code:**

```python
df["Missed FG"] = df["FGA"] - df["FGM"]
df["Missed FT"] = df["FTA"] - df["FTM"]
df["VALUE"] = (df["PTS"] + df["OREB"] + df["DREB"] + df["AST"] + df["STL"] + df["BLK"]) - (df["TOV"] + df["Missed FG"] + df["Missed FT"])

best_player = (df.groupby('PLAYER')['VALUE']
               .mean()
               .reset_index()
               .round(2)
               .sort_values(['VALUE','PLAYER'], ascending=[False,True])
               .reset_index(drop=True))
```

---

## Part 3: Highest Scoring Teams Per Year

**Objective:** Identify the team with the highest average points per season.

**Definition:** Team points = sum of all players' average points in that season.

**Output:** `max_PTS_of_year` dataframe with:

* YEAR
* TEAM
* PTS (rounded to 2 decimals)

Example:

| YEAR | TEAM | PTS   |
| ---- | ---- | ----- |
| 2000 | SAC  | 100.2 |
| 2001 | LAL  | 95.9  |

**Code:**

```python
team_pts = df.groupby(['YEAR','TEAM'])['PTS'].sum().reset_index()
max_PTS_of_year = team_pts.loc[team_pts.groupby('YEAR')['PTS'].idxmax()]
max_PTS_of_year['PTS'] = max_PTS_of_year['PTS'].round(2)
max_PTS_of_year = max_PTS_of_year.sort_values('YEAR').reset_index(drop=True)
```

---

## Summary

* Identified **players achieving double-doubles** in at least 2 key stats per season.
* Calculated **player values** to determine MVPs over 2000–2009.
* Determined **highest scoring teams** per season.

This analysis provides insights into player performance trends, team scoring strengths, and identifies standout performers across a decade of NBA seasons.
