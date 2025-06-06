# Predicting if the game was a STOMP
### DSC 80 Final Project – Spring 2025  
By: Ziyan Liu

## Introduction

This project uses professional League of Legends match data from Oracle’s Elixir for the 2022 season. The dataset includes 150,588 rows, each representing a team’s performance in a single match from major competitive leagues such as the LCS, LEC, LCK, and LPL. The central question this project explores is: Can we predict whether a game will be a "stomp" based on early and mid-game performance metrics? A stomp is defined here as a match that ends in under 25 minutes where one team had a gold lead of at least 1500 at 15 minutes and won, or was behind by that much and lost. I created a custom binary target column, is_stomp, to capture this outcome. This question matters because stomp games highlight extreme early-game dominance and can uncover important patterns or mistakes in gameplay which could be interesting to coaches and fans. To answer it, the analysis focuses on early indicators such as gold, XP, and CS differences at 10 and 15 minutes (golddiffat10, xpdiffat15, etc.); early combat stats (killsat15, assistsat15, deathsat15); objective control (firstblood, firstdragon, firstherald, firsttower); and impact metrics like visionscoreat15, dpm, and damagetakenperminute. (total 22 features)

## Data Cleaning and Exploratory Data Analysis

To prepare the dataset for analysis, I first reduced the original 163 columns to a focused set of 22 features relevant to early and mid-game performance, objective control, and combat metrics. These were selected intentionally to ensure that only pre-outcome features were used when building predictive models, preserving the temporal order of in-game events and avoiding data leakage.

Next, I filtered the data to include only rows where the datacompleteness column was marked as 'complete'. This column is included by Oracle’s Elixir to flag whether full match data was successfully collected, which depends on the success of automated scraping and logging tools. Removing incomplete records ensured that all retained rows had consistent and trustworthy in-game stats.

I then checked for and confirmed there were no placeholder values (e.g., 'NULL', 'NA', or '-') beyond standard NaN entries. Since none were found, I didnt do any conversion. I also verified that the dataset contained no duplicated rows, and none were found.

Finally, I created a custom binary target column, is_stomp, which flags whether a game ended in under 25 minutes and involved a gold difference of at least 1500 at 15 minutes, where the leading team won or the trailing team lost. This variable is the core outcome used for modeling. 

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

This plot shows the distribution of game lengths (in seconds) across all matches in the dataset. Most games last between 1500 and 2200 seconds (approximately 25 to 36 minutes), forming a slightly right-skewed bell curve that reflects the typical pacing of professional League of Legends matches, with fewer extremely short or long games.

### Gold vs. Kills at 15 Minutes – Stomp Comparison

<iframe
  src="assets/gold_kills_by_stomps.html"
  width="800"
  height="500"
  frameborder="0">
</iframe>
This scatter plot shows the relationship between gold difference and kills at 15 minutes, with color indicating whether the match was a stomp. There is a strong positive correlation between early gold and kill leads, and stomp games (in yellow) tend to cluster at the extremes.

### Average Combat Stats at 15 Minutes by Stomp Label
The table below shows the average number of kills, deaths, and assists at 15 minutes, grouped by whether the game was classified as a stomp or not.


|   is_stomp |   assistsat15 |   deathsat15 |   killsat15 |
|-----------:|--------------:|-------------:|------------:|
|          0 |          2.09 |         1.28 |        1.27 |
|          1 |          4.73 |         3.16 |        3.37 |

Teams in stomp games average significantly more kills (3.37 vs. 1.27), assists (4.73 vs. 2.09), and deaths (3.16 vs. 1.28) by the 15-minute mark. This suggests that stomp games are characterized by agression and combat, reinforcing the idea that winning early flights strongly contribute to snowballing advantages then winning.

## Assessment of Missingness

### NMAR Column

I do not believe that any columns in the dataset are Not Missing At Random (NMAR). NMAR would imply that the likelihood of a value being missing is directly related to the missing value itself, which doesn’t appear to be the case in this dataset.

However, the column firstherald is a good example of data that is Missing by Design (MD). Its values are often missing because neither team secured the Rift Herald, so no value was recorded. However, it may also be considered Missing At Random (MAR) since the missingness could depend on other observed variables like killsat15, which is an indication of agressive playstyle that includes grabing the Rift Herlad. For example, games with missing firstherald values tend to have fewer early kills, suggesting a less aggressive playstyle or slower early tempo that results in teams not tryign to grab the Herald.

If additional context—like whether the Herald spawned or how teams moved around objectives—were available, it could further support treating the missingness in firstherald as MAR. But as for NMAR, there's no evidence that the missingness depends on the unobserved value itself, which is why I do not classify it as NMAR.

### Permutation Test: Does First Herald Missingness depend on Kills at 15 Minutes?

- `Null Hypothesis` (H₀): The missingness in `firstherald` is not associated with `killsat15`; any observed difference is due to chance.

- `Alternative Hypothesis` (H₁): The missingness in `firstherald` is associated with `killsat15`; the observed difference is unlikely to be due to chance.

<iframe
  src="assets/firstherald_missingness_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above shows the null distribution of the difference in mean kills at 15 minutes between games where the `firstherald` value is missing and where it is not. The observed difference (red dashed line) is **-3.259**, and the computed p-value is **0.0000**. This result suggests that the missingness of `firstherald` is not random — it likely depends on early-game performance, specifically team kills by the 15-minute mark.

### Does First Herald Missingness depend on the result of the game>

- `Null Hypothesis` (H₀): The missingness in `firstherald` is not associated with `result`; any observed difference in win rate is due to chance.

- `Alternative Hypothesis` (H₁): The missingness in `firstherald` is associated with `result`; the observed difference in win rate is unlikely to be due to chance.

<iframe
  src="assets/result_vs_firstherald_missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above shows the null distribution of the difference in mean match outcome (result) between games where the firstherald value is missing and where it is not. The observed difference (red dashed line) is 0.000, and the computed p-value is 1.0000. This result provides no evidence of an association between firstherald missingness and match outcome, suggesting that whether or not the Herald was recorded is not related to whether the team won or lost.

## Hypothesis Testing

`Null Hypothesis (H₀):` 
The distribution of game lengths is the same in games with low kill difference and high kill difference at 15 minutes.

`Alternative Hypothesis (H₁):` 
The distribution of game lengths is different in games with low kill difference and high kill difference at 15 minutes.


We used a permutation test to evaluate the difference in mean game length between two groups: games with low kill difference (< 3) and games with high kill difference (≥ 3) at 15 minutes. The test statistic is the difference in mean game length between the two groups, and we used a significance level of 0.05.

The observed difference in mean game length was -136.332 seconds, and the p-value was 0.0000. This suggests that, under the null hypothesis, such a large difference in game duration would be extremely unlikely to occur by chance

<iframe
  src="assets/gamelength_killdiff15_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We find strong evidence to reject the null hypothesis. Games with a high kill difference at 15 minutes tend to be significantly shorter than games with a low kill difference. This result suggests that early-game combat dominance is associated with faster game conclusions. However, as this is an observational analysis and not a randomized experiment, we cannot claim a causal relationship.

## Framing a Prediction Problem

The goal of this project is to build a binary classification model that predicts whether a League of Legends game is a stomp, based on early- and mid-game statistics. A stomp is defined as a game that ends quickly (under 25 minutes) where one team decisively wins or loses horribly—characterized by both a significant gold differential at the 15-minute mark and a short game duration.

The response variable is `is_stomp`, a binary label:

  - 1 indicates the game was a stomp
  - 0 indicates it was not a stomp

I chose is_stomp as the target because it captures extreme, one-sided games that can be used to see important large skill differences or lack of preparedness, making easy to see if there might be an unfair match up between a strong and expereince pro team vs a beginner or weak pro team.

To evaluate the model, I want to use the F1 score, which balances precision and recall. F1 is particularly suitable for this task because stomps are relatively rare in the dataset. Unlike accuracy, which can be misleading when classes are imbalanced, F1 ensures that the model is effectively identifying both stomp and non-stomp games.


## Baseline Modele
For my baseline model, I used a Logistic Regression classifier. This is a binary classification task where the goal is to predict whether a game is a stomp, defined as a game that ends within 25 minutes and involves a significant gold differential at the 15-minute mark.

Dataset Balancing
To ensure the model had equal exposure to both outcomes, I constructed a balanced dataset by sampling an equal number of stomp (is_stomp = 1) and non-stomp (is_stomp = 0) games—5,346 of each. This prevents the model from being biased toward the majority class and enables better evaluation using metrics like the F1 score.

### Features Used
The model uses a total of 6 features, including:

3 quantitative (numerical):

  - `golddiffat10`: Gold difference at 10 minutes
  - `csdiffat15`: CS difference at 15 minutes
  - `killsat15`: Kills at 15 minutes

3 nominal (categorical):

  - `side`: Whether the team was Blue or Red
  - `patch`: Version of the game patch
  - `playoffs`: Whether the match occurred during playoffs

There are no ordinal features in this setup.

### Encodings and Preprocessing
  - `Numerical features` were standardized using StandardScaler.
  - `Categorical features` were encoded using OneHotEncoder.

### Model Performance
  - Accuracy: 0.6844
  - F1 Score: 0.6653

### Good/Bad?
This baseline model serves as a good starting point. While Logistic Regression is a simple linear model, it performs reasonably well here. The use of a balanced dataset and F1 score for evaluation helps avoid misleading accuracy metrics and gives a clearer view of performance on both classes.


## Final Model

For my final model, I used a **Random Forest Classifier**. I created a **balanced dataset** by sampling the same number of stomp and non-stomp games (5,346 each) to make the model focus equally on both classes.

### Features Added and Why

I added two new features based on game knowledge:

- `gold_per_minute_10`: Measures how quickly a team gains gold in the first 10 minutes. Faster gold gain often means early dominance, which can lead to a stomp. This helps the model in predicting how fast a stomped vs non-stomped team is accumlating gold within the first ten mins.
- `kda_15`: Combines kills, assists, and deaths at 15 minutes to show how well a team is doing in early fights. High KDA usually means strong early advantage. This ratio is important in giving more context to the kills or deaths recieved. For exmaple, a high death isn't bad if they get the same or greater number of kills/assists.

These features help identify early leads and team strength, which are useful for predicting stomps.

### Features and Encoding

The model used 12 features:

- `Numerical features` (9):
  - `StandardScaler` was used for common stats that could have different scales (in the thousands while kills are not).
  - `QuantileTransformer` was applied to engineered features to reduce possilby skewness (there mgiht be teams without kills or without deaths).

- `Categorical features` (3):
  - `side`, `patch`, and `playoffs`, encoded with `OneHotEncoder`.

### Model and Hyperparameters

I used `GridSearchCV` with 5-fold cross-validation to find the best settings within:
    `classifier__n_estimators`: [50, 100],
    `classifier__max_depth`: [10, 20, None],
    `classifier__min_samples_split`: [2, 5]
    
The final parameters were:
  - `n_estimators = 100`
  - `max_depth = 20`
  - `min_samples_split = 5`

### Model Performance vs Basline

  - `Accuracy`: 0.8139  
  - `F1 Score`: 0.8188

Compared to the baseline model, this is a significant improvement! The added features better capture early dominance patterns that lead to stomps, and the Random Forest model capitalizes on nonlinear interactions across features. The higher F1 score also means that the model is better at correctly identifying both classes, better balancing precision (how many predicted positives are actually correct) and recall (how many actual positives the model catches). 

## Fairness Analysis

### Group Definitions
- **Group X**: Games where `playoffs == 0` (regular season)
- **Group Y**: Games where `playoffs == 1` (playoff matches)

### Evaluation Metric
We used `precision` as the evaluation metric to assess how accurately the model predicts stomp games for each group.

### Hypotheses
- `Null Hypothesis` (H₀): There is no difference in precision between playoff and non-playoff games.
- `Alternative Hypothesis` (H₁): There is a difference in precision between the two groups.

### Test Statistic and Method
We calculated the  `observed precision difference` between Group Y and Group X, then used a permutation test (1,000 permutations) where the `playoffs` labels were shuffled. The test statistic is the `difference in precision` between the two groups.

- `Observed Precision Difference`: 0.0000  
- `P-value`: 0.4930  
- `Significance Level`: α = 0.05

### Conclusion
Since the p-value (0.4930) is greater than the significance level (0.05), we **fail to reject the null hypothesis**. This suggests that there is **no statistically significant difference** in the model’s precision between playoff and non-playoff games. Meaning that the model is fair for these groups.







