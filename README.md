# How Punishing is the Snowball Effect in Professional League of Legends?

This project was for the UCSD DSC80 course.

## Introduction

This project investigates how punishing the snowball effect is in 2025 professional League of Legends. Specifically: **do early game leads in gold, kills, and objectives reliably predict which team wins the match?**

If small early advantages snowball into near-guaranteed wins, it means the game heavily punishes early mistakes and rewards aggressive early play. This matters for competitive balance, coaching strategy, and how fans understand what makes a team likely to win.

The dataset comes from [Oracle's Elixir](https://oracleselixir.com/) and contains match-level data from professional LoL esports in the 2025 season. After filtering to team-level rows and dropping games that ended before 15 minutes, the dataset contains **18,472 rows**. Each row represents one team's performance in a single game.

| Column | Description |
|---|---|
| `golddiffat15` | Gold differential at 15 minutes |
| `golddiffat25` | Gold differential at 25 minutes |
| `xpdiffat15` | Experience differential at 15 minutes |
| `csdiffat15` | Creep score differential at 15 minutes |
| `killsat15` | Number of kills the team has at 15 minutes |
| `firstblood` | Whether the team got first blood |
| `firstdragon` | Whether the team got the first dragon |
| `firstherald` | Whether the team got the first herald |
| `firsttower` | Whether the team got the first tower |
| `firstbaron` | Whether the team got the first baron |
| `gamelength` | Length of the game in seconds (converted to minutes during cleaning) |
| `side` | Blue or Red side |
| `result` | Match outcome (1 = win, 0 = loss) |

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

1. Selected only columns relevant to our snowball analysis from the original 165-column dataset.
2. Converted `playoffs`, `firstblood`, `firstdragon`, `firstherald`, `firsttower`, and `firstbaron` to boolean types since they represent binary events.
3. Converted `gamelength` from seconds to minutes for interpretability.
4. Filtered to only `position == 'team'` rows since we care about team-level performance, not individual player stats.
5. Dropped rows missing `golddiffat15`, `csdiffat15`, or `xpdiffat15` since games that ended before 15 minutes cannot be used to study the snowball effect at that timestamp.

Here is the head of the cleaned DataFrame:

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>gameid</th>
      <th>league</th>
      <th>gamelength</th>
      <th>side</th>
      <th>position</th>
      <th>result</th>
      <th>golddiffat15</th>
      <th>golddiffat25</th>
      <th>csdiffat15</th>
      <th>xpdiffat15</th>
      <th>killsat15</th>
      <th>firstblood</th>
      <th>firstdragon</th>
      <th>firstherald</th>
      <th>firsttower</th>
      <th>firstbaron</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>LOLTMNT03_179647</td>
      <td>LFL2</td>
      <td>26.53</td>
      <td>Blue</td>
      <td>team</td>
      <td>0</td>
      <td>-3837.0</td>
      <td>-6966.0</td>
      <td>-16.0</td>
      <td>-469.0</td>
      <td>0.0</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <td>LOLTMNT03_179647</td>
      <td>LFL2</td>
      <td>26.53</td>
      <td>Red</td>
      <td>team</td>
      <td>1</td>
      <td>3837.0</td>
      <td>6966.0</td>
      <td>16.0</td>
      <td>469.0</td>
      <td>3.0</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
    </tr>
    <tr>
      <td>LOLTMNT06_96134</td>
      <td>LFL2</td>
      <td>32.03</td>
      <td>Blue</td>
      <td>team</td>
      <td>1</td>
      <td>5069.0</td>
      <td>8377.0</td>
      <td>64.0</td>
      <td>2014.0</td>
      <td>10.0</td>
      <td>True</td>
      <td>False</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
    </tr>
    <tr>
      <td>LOLTMNT06_96134</td>
      <td>LFL2</td>
      <td>32.03</td>
      <td>Red</td>
      <td>team</td>
      <td>0</td>
      <td>-5069.0</td>
      <td>-8377.0</td>
      <td>-64.0</td>
      <td>-2014.0</td>
      <td>5.0</td>
      <td>False</td>
      <td>True</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <td>LOLTMNT06_95160</td>
      <td>LFL2</td>
      <td>29.70</td>
      <td>Blue</td>
      <td>team</td>
      <td>0</td>
      <td>118.0</td>
      <td>-5621.0</td>
      <td>-43.0</td>
      <td>1990.0</td>
      <td>10.0</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

### Univariate Analysis

<iframe src="assets/gamelength-distribution.html" width="800" height="600" frameborder="0"></iframe>

Most professional games last between 25 and 35 minutes, with the distribution skewing slightly right. Very few games end before 20 minutes or extend past 45, suggesting that while snowball effects can end games early, most matches still play out to a mid-game conclusion.

### Bivariate Analysis

<iframe src="assets/golddiff15-by-result.html" width="800" height="600" frameborder="0"></iframe>

Winning teams tend to have positive gold differentials at 15 minutes, while losing teams cluster in the negatives. The two distributions are clearly separated, reinforcing that early gold leads are strongly associated with match outcomes.

<iframe src="assets/winrate-by-goldbucket.html" width="800" height="600" frameborder="0"></iframe>

Win rate increases sharply with gold differential at 15 minutes. Teams with large gold leads (5000+) win nearly every game, while teams far behind almost never recover. The steep curve confirms that early gold advantages snowball heavily into wins.

### Interesting Aggregates

| First Herald \ First Dragon | False | True |
|:---|---:|---:|
| False | 0.27 | 0.41 |
| True | 0.59 | 0.73 |

This pivot table shows win rates grouped by whether a team secured first herald (rows) and first dragon (columns). Teams that secured both objectives win 73% of the time, while teams that secured neither win only 27%. Notably, first herald appears to have a larger impact on win rate than first dragon, suggesting that early map control from herald contributes more to the snowball effect.



