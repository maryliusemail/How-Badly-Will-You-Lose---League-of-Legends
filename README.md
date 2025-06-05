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

For my baseline model, I used a **Logistic Regression** classifier with balanced class weights to address the class imbalance in the target variable. The model was trained on seven features: `golddiffat15`, `killsat15`, `deathsat15`, `firstdragon`, `firstherald`, `firsttower`, and `gamelength`. The first four features are quantitative and were standardized using `StandardScaler` to ensure consistent scaling, particularly since game durations can vary significantly. The last three features are binary categorical variables that capture key in-game objectives, and they were one-hot encoded after imputing missing values with the most frequent category.

To improve upon this baseline, I plan to experiment with different types of models such as **Random Forests**, **Gradient Boosting**, and other **ensemble methods** to better capture non-linear relationships and improve overall predictive performance.


## Final Model

In my final model, I used a **Random Forest Classifier** to improve upon the baseline logistic regression model. I kept the same set of seven features: `golddiffat15`, `killsat15`, `deathsat15`, `firstdragon`, `firstherald`, `firsttower`, and `gamelength`. These features were selected because they reflect both early and mid-game performance, as well as objective control, all of which are important indicators of whether a match is a “stomp.” The quantitative features were standardized using `StandardScaler`, and the categorical features were one-hot encoded after imputation.

Compared to logistic regression, Random Forest offers the ability to model complex, non-linear relationships and interactions between features, making it a strong choice for capturing the dynamics of League of Legends matches. To ensure robustness, I used class-balanced weighting and left the default number of trees at 100, with a fixed random state to ensure reproducibility.

After training the model on an 80/20 stratified split and evaluating it on the test set, my final model achieved an **F1 score of 0.9907**. This represents a substantial improvement over the baseline model’s F1 score of **0.5295**, indicating that the Random Forest classifier is highly effective at capturing the underlying patterns in the data. Both precision and recall are near perfect, suggesting that the model performs exceptionally well in identifying “stomp” games with very few false positives or false negatives.

This performance demonstrates that switching to a more powerful model, even without adding new features or tuning hyperparameters extensively, can yield dramatic gains in predictive power.












