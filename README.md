# How Badly Will You Lose – League of Legends  
### DSC 80 Final Project – Spring 2025  
By: Ziyan Liu

## Introduction

This project uses professional League of Legends match data from [Oracle’s Elixir](https://oracleselixir.com/match-data/) from 2022, which contains detailed post-game statistics from major competitive leagues such as the LCS, LEC, LCK, and LPL. The dataset includes **150,588 rows and 163 columns**, with each row representing a single player's performance in a match.

The goal of this project is to predict whether a game was a “stomp” — defined here as a short match (less than 25 minutes) in which one team had a gold lead of at least 1500 at 15 minutes and went on to win, or was behind by that much and lost. Stomps are meaningful because they showcase overwhelming early-game dominance and can reveal patterns in team strategy and execution.

To answer this question, we focus on a subset of early and mid-game features, such as gold, XP, and CS differences at 10 and 15 minutes; kills, assists, and deaths at the same intervals; early objective control (`firstblood`, `firstdragon`, `firstherald`, `firsttower`); and combat impact metrics like `visionscore`, `dpm` (damage per minute), and `damagetakenperminute`. The target column, `is_stomp`, is a binary label derived from game length, result, and gold differential at 15 minutes.

## Data Cleaning and Exploratory Data Analysis

To get the dataset ready for analysis, I started by loading the full 2022 Oracle’s Elixir League of Legends dataset, which originally had 150,588 rows and 163 columns. From there, I narrowed it down to 23 columns that focused on early- and mid-game performance, objective control, and combat stats — the kinds of features that would matter most when trying to identify one-sided, fast-paced games (or “stomps”).

Since the data came from multiple match logs, there were some inconsistencies in how missing values were recorded — things like `'NULL'`, `'na'`, or even empty strings. I cleaned these up by converting them all into proper missing values using pandas. After that, I filtered the dataset to include only rows marked as `'complete'` in the `datacompleteness` column, so I knew I was working with reliable data. I also removed any duplicates to avoid skewing results with repeated entries.

The last step was creating a new column called `is_stomp`, which flags whether a match qualifies as a stomp. I defined a stomp as a game that ended in under 25 minutes and where one team either had a gold lead of 1500+ at 15 minutes and won, or was behind by 1500+ and lost. This new column became the label I’m trying to predict in the rest of the project.

|   golddiffat10 |   xpdiffat10 |   csdiffat10 |   golddiffat15 |   xpdiffat15 |   csdiffat15 |   killsat10 |   assistsat10 |   deathsat10 |   killsat15 |   assistsat15 |   deathsat15 |   firstblood |   firstdragon |   firstherald |   firsttower |   visionscore |   damagetakenperminute | datacompleteness   |   gamelength |   result |   is_stomp |
|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|------------:|--------------:|-------------:|-------------:|--------------:|--------------:|-------------:|--------------:|-----------------------:|:-------------------|-------------:|---------:|-----------:|
|             52 |          -44 |            8 |            391 |          345 |           14 |           0 |             0 |            0 |           0 |             1 |            0 |            0 |           nan |           nan |          nan |            26 |               1072.4   | complete           |         1713 |        0 |          0 |
|            485 |          432 |           -5 |            541 |         -275 |          -11 |           1 |             2 |            0 |           2 |             3 |            2 |            1 |           nan |           nan |          nan |            48 |                944.273 | complete           |         1713 |        0 |          0 |
|            162 |           71 |            0 |           -475 |          153 |            1 |           0 |             1 |            0 |           0 |             3 |            0 |            0 |           nan |           nan |          nan |            29 |                581.646 | complete           |         1713 |        0 |          0 |
|            296 |          265 |          -12 |           -793 |        -1343 |          -34 |           1 |             1 |            0 |           2 |             1 |            2 |            1 |           nan |           nan |          nan |            25 |                463.853 | complete           |         1713 |        0 |          0 |
|            528 |         -587 |            1 |            443 |         -497 |            7 |           1 |             1 |            0 |           1 |             2 |            2 |            1 |           nan |           nan |          nan |            69 |                475.026 | complete           |         1713 |        0 |          0 |


### Game Length Distribution

<iframe
  src="assets/game_length.html"
  width="800"
  height="500"
  frameborder="0">
</iframe>

This plot shows the distribution of game lengths across all matches.

### Gold vs. Kills at 15 Minutes – Stomp Comparison

<iframe
  src="assets/gold_kills_by_stomps.html"
  width="800"
  height="500"
  frameborder="0">
</iframe>

This plot shows how gold and kill counts at 15 minutes vary between stomp and non-stomp games. Stomps tend to have higher gold and kill values early on.
### Average Combat Stats at 15 Minutes by Stomp Label

|   is_stomp |   assistsat15 |   deathsat15 |   killsat15 |
|-----------:|--------------:|-------------:|------------:|
|          0 |          2.09 |         1.28 |        1.27 |
|          1 |          4.73 |         3.16 |        3.37 |

This pivot table shows that stomp games have significantly more kills and assists, and fewer deaths at 15 minutes. This indicates strong early momentum and dominance, consistent with the definition of a stomp.

## Assessment of Missingness

### NMAR Column

Yes, I believe the column `firstherald` in my dataset is **Not Missing At Random (NMAR)**. The missingness of this column appears to be tied to how the game was played. Specifically, matches where `firstherald` is missing often have fewer kills by 15 minutes, suggesting a slower-paced or less aggressive early game. This implies that no team may have attempted to take the Rift Herald, and therefore the data was never recorded.

Since the missingness seems related to in-game behavior—such as early-game tempo or team strategy—it is not due to random error or chance, but rather tied to unobserved player decisions. This makes the missingness dependent on the underlying (and unrecorded) game flow, which is why I classify it as NMAR.

To possibly treat this column as **Missing At Random (MAR)** instead, I would need more contextual data. For instance, knowing whether the Herald spawned at all, whether teams moved toward the objective, or if the game ended before it became relevant could help explain the missingness based on observed variables. With that information, a predictive model could account for the missing values in a MAR framework.

### Permutation Test: Kills at 15 Minutes vs. First Herald Missingness

<iframe
  src="assets/firstherald_missingness_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above shows the null distribution of the difference in mean kills at 15 minutes between games where the `firstherald` value is missing and where it is not. The observed difference (red dashed line) is **-3.259**, and the computed p-value is **0.0000**. This result suggests that the missingness of `firstherald` is not random — it likely depends on early-game performance, specifically team kills by the 15-minute mark.

## Hypothesis Testing

**Null Hypothesis (H₀):**  
The distribution of game lengths is the same in games with low kill difference and high kill difference at 15 minutes.

**Alternative Hypothesis (H₁):**  
The distribution of game lengths is different in games with low kill difference and high kill difference at 15 minutes.

We used a permutation test comparing the mean game length of two groups: games with a **low kill difference (< 3)** and those with a **high kill difference (≥ 3)** at 15 minutes. The observed test statistic (difference in means) was **-136.331 seconds**, and the resulting **p-value was 0.0000**.

<iframe
  src="assets/gamelength_killdiff15_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This p-value is well below the standard significance threshold of 0.05, so we **reject the null hypothesis**. This provides strong statistical evidence that early-game dominance (measured by kill difference at 15 minutes) is associated with significantly shorter game durations. In other words, early aggression often leads to quicker wins in League of Legends.

## Framing a Prediction Problem

The goal of this project is to **predict whether a professional League of Legends game is a "stomp"** based on early- and mid-game statistics. A **stomp** is defined as a game that lasts less than 25 minutes (1500 seconds) and where the winning team has a gold lead of at least 1500 at 15 minutes, or the losing team has a deficit of at least 1500 gold at the same time.

This is a **binary classification problem**, where the target variable is `is_stomp`:
- `1` indicates the game was a stomp
- `0` indicates it was not

We chose this variable because stomp games highlight major disparities in early-game performance and can offer insight into dominant playstyles. Predicting them using early indicators could help coaches, analysts, or broadcasters understand which games are likely to snowball out of control.

To ensure that we are only using information available at the **time of prediction**, all features were limited to statistics recorded within the first 15 minutes of the match. 

## Baseline Model

For my baseline model, I used a **Logistic Regression classifier** with balanced class weights to account for the imbalance in the binary target variable `is_stomp`. The model was trained on seven features, listed as follows:

- **Quantitative features - 4 **:  
  - `golddiffat15`  
  - `killsat15`  
  - `deathsat15`  
  - `gamelength`  
  These are continuous numerical values that capture early-to-mid game combat statistics and game duration.

- **Nominal categorical features - 3 **:  
  - `firstdragon`  
  - `firstherald`  
  - `firsttower`  
  These are binary indicators representing whether the team secured key objectives.

There were **no ordinal features** in this dataset.

To prepare the data for modeling:
- I applied standard scaling to the quantitative features using `StandardScaler`, to account for varying scales, especially due to different game lengths.
- For categorical features, I used **most frequent imputation** followed by **one-hot encoding** to convert them into a numeric format suitable for the model.

I trained the model on an 80/20 stratified train-test split and evaluated it using the **F1 score**, which is a better metric than accuracy for imbalanced classification problems. The **baseline model achieved an F1 score of 0.5303**. While the model shows some ability to predict “stomp” games, the relatively low F1 score suggests limited recall or precision. Therefore, I do **not consider this model to be strong**, but it provides a reasonable foundation to build upon.

---

## Final Model

To improve upon the baseline, I engineered two new quantitative features based on domain knowledge from League of Legends:

1. **`gold_per_min`**: Calculated as `golddiffat15 / gamelength`, this feature reflects how quickly a team accumulates a gold lead, regardless of game duration. This gives insight into tempo and their efficiency.

2. **`kda15`**: A kill-death ratio, defined as `(killsat15 + 1) / (deathsat15 + 1)`, which captures combat efficiency while avoiding division-by-zero errors. A higher `kda15` often corresponds to a stronger early game dominance.

These features were chosen because they reflect capture meaningful elements of the game. Faster gold accumulation and better combat stats are both key indicators of a game being a "stomp."

For the final model, I used a **Random Forest Classifier**, which is well-suited to capturing non-linear interactions between variables. I used a pipeline that applied:

- **Standard scaling** for the original quantitative features (`golddiffat15`, `killsat15`, `deathsat15`)
- **Quantile transformation** (to normalize skew) for the engineered features (`gold_per_min`, `kda15`) and `gamelength`
- **One-hot encoding** for the categorical features (`firstdragon`, `firstherald`, `firsttower`) after imputing missing values

### Hyperparameter Tuning

To optimize the model, I used GridSearchCV with 5-fold cross-validation, testing combinations of:

- `n_estimators`: [50, 100, 150]
- `max_depth`: [5, 10, 20, None]

The best hyperparameters found were:
- `n_estimators = 50`
- `max_depth = 10`

### Final Model Performance

Using these hyperparameters, the final model achieved an **F1 score of 0.9902** on the same test set as the baseline. This is a big improvement over the baseline score of 0.5303!









