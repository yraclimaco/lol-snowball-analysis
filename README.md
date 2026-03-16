# How Punishing is the Snowball Effect in Professional League of Legends?

Name: Yra Climaco

This is a comprehensive data science project completed through the UCSD DSC80 course.

## Introduction

In League of Legends, two teams of five compete to destroy each other's base. Each match progresses through distinct phases: the early game (first 15 minutes) focuses on farming gold, securing kills, and taking objectives like dragons and heralds. Teams that build advantages in these areas can "snowball," using their lead to gain even more resources, making it increasingly difficult for the losing team to catch up.

This project investigates how punishing the snowball effect is in 2025 professional League of Legends. Specifically: **do early game leads such as in gold, kills, and objectives reliably predict which team wins the match?**

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
2. Converted `firstblood`, `firstdragon`, `firstherald`, `firsttower`, and `firstbaron` to boolean types since they represent binary events.
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

<iframe src="assets/gamelength-distribution.html" width="100%" height="400" frameborder="0"></iframe>

Most professional games last between 25 and 35 minutes, with the distribution skewing slightly right. Very few games end before 20 minutes or extend past 45, suggesting that while snowball effects can end games early, most matches still play out to a mid-game conclusion.

### Bivariate Analysis

<iframe src="assets/golddiff15-by-result.html" width="100%" height="400" frameborder="0"></iframe>

Winning teams tend to have positive gold differentials at 15 minutes, while losing teams cluster in the negatives. The two distributions are clearly separated, reinforcing that early gold leads are strongly associated with match outcomes.

<iframe src="assets/winrate-by-goldbucket.html" width="100%" height="400" frameborder="0"></iframe>

Win rate increases sharply with gold differential at 15 minutes. Teams with large gold leads (5000+) win nearly every game, while teams far behind almost never recover. The steep curve confirms that early gold advantages snowball heavily into wins.

<iframe src="assets/side-winrate.html" width="100%" height="400" frameborder="0"></iframe>

Blue side holds a slight win rate advantage over Red side, consistent with the known structural advantage from draft order and map asymmetry. This motivated the fairness analysis at the end of the project to check whether our model performs differently across sides.

### Interesting Aggregates

| First Herald \ First Dragon | False | True |
|:---|---:|---:|
| False | 0.27 | 0.41 |
| True | 0.59 | 0.73 |

This pivot table shows win rates grouped by whether a team secured first herald (rows) and first dragon (columns). Teams that secured both objectives win 73% of the time, while teams that secured neither win only 27%. Notably, first herald appears to have a larger impact on win rate than first dragon, suggesting that early map control from herald contributes more to the snowball effect.

## Assessment of Missingness

### MNAR Analysis

I do not believe any columns in the cleaned dataset are **MNAR** (Missing Not At Random). The columns with missing values (`golddiffat25`, `firstbaron`, etc.) are missing because the game ended before those timestamps or events could occur, which is directly explained by `gamelength`, an observed column in the dataset. Since the missingness can be explained by other observed columns, these are better characterized as **MAR** (Missing At Random).

However, in the original `lol_df` (before filtering for relevant columns), the `url` column (110,832 missing values) is a strong candidate for **MNAR**. Whether a match page URL was published depends on decisions made by tournament organizers or data providers. these are factors that are not captured anywhere in the dataset. No observed column explains why some games have URLs and others do not. To make this MAR, we would need additional data such as tournament tier, broadcast rights, or data provider coverage records.

### Missingness Dependency

I tested whether the missingness of `golddiffat25` depends on other columns using permutation tests.

**Test 1: Missingness depends on `firstbaron`**

Games missing `golddiffat25` ended before 25 minutes, likely before baron even spawns at 20 minutes. We expect fewer first baron takes among the missing group. The permutation test yielded a p-value of approximately 0.0, so we reject the null hypothesis that the missingness of `golddiffat25` is independent of `firstbaron`.

<iframe src="assets/missingness-baron.html" width="100%" height="400" frameborder="0"></iframe>

**Test 2: Missingness does not depend on `firstdragon`**

First dragon spawns early (~5 minutes) and should not predict whether the game lasts to 25 minutes. The permutation test yielded a p-value of approximately 0.791, so we fail to reject the null. The missingness of `golddiffat25` likely does not depend on `firstdragon`.

<iframe src="assets/missingness-dragon.html" width="100%" height="400" frameborder="0"></iframe>

## Hypothesis Testing

Our EDA showed a strong visual relationship between early gold leads and winning. But is this difference statistically significant, or could it be due to random chance? To find out, lets run a permutation test.

**Null Hypothesis:** There is no significant difference in win rate between teams with a positive gold differential at 15 minutes and teams with a negative gold differential at 15 minutes.

**Alternative Hypothesis:** Teams with a positive gold differential at 15 minutes have a higher win rate than those with a negative gold differential at 15 minutes.

**Test Statistic:** Difference in mean win rate (positive gold group minus negative gold group). I chose this because we are comparing a numerical outcome (win rate) across two groups, and our alternative hypothesis is directional.

**Significance Level:** α = 0.05

**Conclusion:**
The observed difference in win rate was approximately 0.458. After 1,000 permutations, the p-value was approximately 0.0. Since this is well below our significance level, so we reject the null hypothesis. The data suggests that teams with a positive gold differential at 15 minutes tend to win at a significantly higher rate. This is further evidence that the snowball effect is real in professional play.

<iframe src="assets/hypothesis-test.html" width="100%" height="400" frameborder="0"></iframe>

## Framing a Prediction Problem

Now that we've established that early gold leads are strongly associated with winning, the natural next question is: **can we actually predict the match winner using only 15-minute data?**

This is a **binary classification** problem. The response variable is `result` (1 = win, 0 = loss).

If early game state is highly predictive of the final outcome, it confirms that the snowball effect is punishing in professional play. The stronger the prediction, the more punishing the snowball.

**Metric:** Accuracy. Since every game produces exactly one winner and one loser, the classes are perfectly balanced (50/50), so accuracy is an appropriate metric. There is no class imbalance that would require F1-score.

**Time of prediction:** We only use features available at the 15-minute mark. We would not yet know `golddiffat20`, `golddiffat25`, `firstbaron`, `gamelength`, or the final result.

## Baseline Model

Our baseline model is a **Logistic Regression** implemented in a single sklearn Pipeline with three features:

- `golddiffat15` (quantitative) -- scaled with StandardScaler
- `xpdiffat15` (quantitative) -- scaled with StandardScaler
- `firstblood` (nominal, boolean) -- encoded with OneHotEncoder

The model achieved a **training accuracy of 0.7352** and a **test accuracy of 0.7288**. Since these values are close, the model generalizes well to unseen data. At ~73%, it performs well above the 50% random guessing baseline, suggesting early game stats do carry meaningful predictive signal. However, there is room for improvement by incorporating more early-game features and using a more flexible model.

## Final Model

To improve on the baseline, we switched to a **Random Forest Classifier** and engineered new features that capture more dimensions of the snowball effect.

**New engineered features:**

1. `abs(golddiffat15)`: Captures how lopsided the game is regardless of which side holds the lead. A team down 3k gold faces the same snowball pressure as one up 3k. This helps the model distinguish close games from stomps.

2. `golddiffat15 * killsat15`: Interaction term. A gold lead built through kills is more snowbally than one from farming, because kills also deny enemy XP and map pressure. This captures the quality of the lead, not just its size.

**Additional features beyond baseline:**
- `csdiffat15`: CS (creep score) difference reflects farming efficiency, a different dimension of early advantage than raw gold.
- `killsat15`: Kill count captures aggression and fight outcomes, which contribute to snowballing through denied enemy XP and map pressure.
- `firstdragon`, `firstherald`, `firsttower`: Objective control signals that indicate which team is dictating the pace of the game. As we saw in the EDA, securing these objectives is strongly associated with winning.
- `side`: Blue side has a known structural win rate advantage in professional play due to draft order and map asymmetry.

**Hyperparameter tuning:** We used GridSearchCV with 5-fold cross-validation over `max_depth` [5, 10, 15, 20, None] and `n_estimators` [100, 200, 300]. The best parameters were **max_depth = 5** and **n_estimators = 200**, with a best CV accuracy of 0.7508.

**Performance comparison:**

| Model | Train Accuracy | Test Accuracy |
|:---|---:|---:|
| Baseline (Logistic Regression) | 0.7352 | 0.7288 |
| Final (Random Forest) | 0.7556 | 0.7364 |

The final model improves test accuracy by about 0.8 percentage points. The modest `max_depth` of 5 prevents overfitting while still capturing nonlinear thresholds in the snowball effect such as the difference between a recoverable 1k gold lead and a decisive 3k lead. The small gap between train and test accuracy confirms the model generalizes well.

<iframe src="assets/confusion-matrix.html" width="100%" height="400" frameborder="0"></iframe>

## Fairness Analysis

Finally, we want to check whether our model is fair across different groups. Since Blue side has a known structural advantage in professional play, we test whether our model performs differently depending on which side a team is on.

**Groups:** Blue side (Group X) vs. Red side (Group Y)

**Evaluation Metric:** Accuracy

**Null Hypothesis:** The model is fair. Its accuracy for Blue side and Red side teams are roughly the same, and any differences are due to random chance.

**Alternative Hypothesis:** The model is unfair. Its accuracy for Blue side is different from its accuracy for Red side.

**Test Statistic:** Difference in accuracy (Blue - Red)

**Significance Level:** α = 0.05

**Result:** The observed Blue side accuracy was 0.7327 and Red side accuracy was 0.7399, giving an observed difference of -0.0071. After 1,000 permutations, the p-value was 0.633. Since this is well above our significance level of 0.05, we fail to reject the null hypothesis. The data suggests that our model performs similarly for both Blue and Red side teams.

<iframe src="assets/fairness-test.html" width="100%" height="400" frameborder="0"></iframe>

