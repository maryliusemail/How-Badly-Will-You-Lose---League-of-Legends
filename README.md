# How Badly Will You Lose – League of Legends  
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

Yes, I believe the column `firstherald` in my dataset is **Not Missing At Random (NMAR)**. When firstherald is 1, the team secured the Rift Herald; 0 means the opponent did. If the value is missing (NaN), it's likely because no team secured the Herald. The missingness of this column appears to be tied to how the game was played. Specifically, matches where `firstherald` is missing often have fewer kills by 15 minutes, suggesting a slower-paced and less aggressive early game. This implies that no team may have attempted to take the Rift Herald, and therefore the data was never recorded. 

Since the missingness seems related to in-game behavior—such as early-game tempo and agression and is not due to random error or chance, but rather tied to unobserved player decisions, which is why I classify it as NMAR.

To possibly treat this column as **Missing At Random (MAR)** instead, I would need more context. For instance, knowing whether the Herald spawned at all, whether teams moved toward the objective, or if the game ended before it became relevant to have a herald could help explain the missingness based on observed variables. With those extra information, we can potentially treat it as Missing At Random,

### Permutation Test: Kills at 15 Minutes vs. First Herald Missingness

- Null Hypothesis (H₀): The missingness in `firstherald` is not associated with `killsat15`; any observed difference is due to chance.

- Alternative Hypothesis (H₁): The missingness in `firstherald` is associated with `killsat15`; the observed difference is unlikely to be due to chance.

<iframe
  src="assets/firstherald_missingness_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above shows the null distribution of the difference in mean kills at 15 minutes between games where the `firstherald` value is missing and where it is not. The observed difference (red dashed line) is **-3.259**, and the computed p-value is **0.0000**. This result suggests that the missingness of `firstherald` is not random — it likely depends on early-game performance, specifically team kills by the 15-minute mark.

### Permutation Test: Result vs. First Herald Missingness

- Null Hypothesis (H₀): The missingness in `firstherald` is not associated with `result`; any observed difference in win rate is due to chance.

- Alternative Hypothesis (H₁): The missingness in `firstherald` is associated with `result`; the observed difference in win rate is unlikely to be due to chance.

<iframe
  src="assets/result_vs_firstherald_missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above shows the null distribution of the difference in mean match outcome (result) between games where the firstherald value is missing and where it is not. The observed difference (red dashed line) is 0.000, and the computed p-value is 1.0000. This result provides no evidence of an association between firstherald missingness and match outcome, suggesting that whether or not the Herald was recorded is not related to whether the team won or lost.

## Hypothesis Testing

**Null Hypothesis (H₀):**  
The distribution of game lengths is the same in games with low kill difference and high kill difference at 15 minutes.

**Alternative Hypothesis (H₁):**  
The distribution of game lengths is different in games with low kill difference and high kill difference at 15 minutes.


We used a permutation test to evaluate the difference in mean game length between two groups: games with low kill difference (< 3) and games with high kill difference (≥ 3) at 15 minutes. The test statistic is the difference in mean game length between the two groups, and we used a significance level of 0.05.

The observed difference in mean game length was -136.332 seconds, and the p-value was 0.0000. This suggests that, under the null hypothesis, such a large difference in game duration would be extremely unlikely to occur by chance

<iframe
  src="assets/gamelength_killdiff15_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We find strong evidence to reject the null hypothesis. Games with a high kill difference at 15 minutes tend to be significantly shorter than games with a low kill difference. This result suggests that early-game combat dominance is associated with faster game conclusions. However, as this is an observational analysis and not a randomized experiment, we cannot claim a causal relationship a strong association.

## Framing a Prediction Problem

The goal of this project is to predict whether a professional League of Legends game is a "stomp" based on early- and mid-game statistics. A **stomp** is defined as a game that lasts less than 25 minutes (1500 seconds) and where the winning team has a gold lead of at least 1500 at 15 minutes, or the losing team has a deficit of at least 1500 gold at the same time.

This is a **binary classification problem**, where the target variable is `is_stomp`:
- `1` indicates the game was a stomp
- `0` indicates it was not

I chose this variable because stomp games highlight major differences in early-game performance and can offer insight into dominant playstyles or major skill differences. Predicting them using early indicators could also help coaches or broadcasters understand which games are likely to snowball out of control.

To ensure that I am only using information available at the **time of prediction**, all features were limited to statistics recorded within the first 15 minutes of the match. 

## Baseline Model
For my baseline model, I used a **Logistic Regression classifier** with balanced class weights to account for the severe class imbalance in the binary target variable `is_stomp`. The model was trained on five features selected for their potential to capture early game advantages:

- **Quantitative features – 3**:  
  - `golddiffat10`  
  - `csdiffat15`  
  - `killsat15`  
  These continuous numerical features represent early economic and combat leads that could signal a snowballing game.

- **Nominal categorical features – 2**:  
  - `firstherald`  
  - `firstdragon`  
  These are binary indicators that reflect whether the team secured key early-game objectives.

There were no ordinal features used in this model.

To prepare the data:
- I **dropped rows with missing values in categorical features** to ensure clean encodings.
- I applied **mean imputation and standard scaling** (`StandardScaler`) to the quantitative features to normalize their ranges.
- For the categorical features, I used **one-hot encoding** with `handle_unknown='ignore'` to convert them into a machine-readable format.

I split the dataset using an 80/20 **stratified train-test split** to preserve class proportions. The model was evaluated using both **accuracy** and **F1 score**:

- **Accuracy**: 0.8867  
- **F1 Score**: 0.0082

While the model achieved high accuracy, this is misleading due to the imbalance of the target variable. The extremely low F1 score indicates that the model performs poorly at identifying the minority class (`is_stomp = 1`). The F1 score is used to validate it because it balances precision and recall, making it ideal for evaluating models on imbalanced classification tasks. Therefore, I do **not consider this model to be good**, but it serves as a baseline for further improvement.



## Final Model

For my final model, I used a **Random Forest Classifier** with balanced class weights to account for the strong class imbalance in the binary target variable `is_stomp`. This model is well-suited for classifications, especially in cases with complex feature interactions and imbalanced classes. It was selected due to its ability to capture nonlinear patterns and dependencies between features, which is important in this case.


### Features Added and Why

To improve the model’s predictive performance, I engineered two new features:

- `gold_per_minute`: This is the gold differential at 15 minutes divided by 900 (seconds), capturing the pace at which a team builds a gold lead. Since stomps are often characterized by rapid economic dominance, this metric reflects a team’s speed and efficiency.

- `kda_15`: Calculated as `(killsat15 + assistsat15 + 1) / (deathsat15 + 1)`, this kill-death-assist ratio represents early combat success and survivability. High values here typically correlate with stronger early performance and often leads a stomp.


### Feature Types and Encodings

The final model used was a `RandomForestClassifier` with a  total of **20 features**, including:

- **Quantitative features (17)**:
  - These include standard early-game metrics such as `golddiffat10`, `xpdiffat10`, `visionscore`, as well as the two engineered features: `gold_per_minute` and `kda_15`.
  - `StandardScaler` was applied to common early-game features to normalize them.
  - `QuantileTransformer` was used on the engineered features to reduce skewness.
  - Missing values in numeric features were imputed using the mean.

- **Nominal categorical features (3)**:
  - `firstblood`, `firstdragon`, and `firstherald` were imputed using the most frequent value and encoded using one-hot encoding.

There were no ordinal features in this dataset.

---

### Hyperparameters

I used **GridSearchCV** with 5-fold cross-validation to select the best combination of hyperparameters:

- `n_estimators`: [50, 100]
- `max_depth`: [10, 20, None]
- `min_samples_split`: [2, 5]

**Best Parameters Found**:
- `n_estimators = 50`
- `max_depth = 10`
- `min_samples_split = 5`

These hyperparameters offered the best trade-off between underfitting and overfitting based on the cross-validation F1 score.


### Performance Comparison

- **Baseline Model (Logistic Regression)**  
  - Accuracy: 0.8867  
  - F1 Score: 0.0082  

- **Final Model (Random Forest)**  
  - Accuracy: **0.9999**  
  - F1 Score: **0.8333**

The final model significantly improved upon the baseline, especially the **F1 score**. The F1 score is used because it balances precision and recall, making it ideal for evaluating models on imbalanced classification tasks. The substantial improvement in F1 score indicates that the final model is far more effective at identifying “stomp” games, achieving both high precision and recall.


## Fairness Analysis

To evaluate the fairness of my Final Model, I examined whether the model performs differently for two groups based on early-game objective control:

- **Group X**: Teams that did *not* secure the first dragon (`firstdragon = 0`)
- **Group Y**: Teams that *did* secure the first dragon (`firstdragon = 1`)

I chose **precision** as the evaluation metric, as it reflects how often the model is correct when it predicts a "stomp" — which is especially important when such predictions may influence decision-making or further analysis.

### Hypotheses

- **Null Hypothesis (H₀)**: The model is fair. Precision is equal for both groups, and any observed difference is due to chance.
- **Alternative Hypothesis (H₁)**: The model is unfair. Precision is lower for teams that did **not** secure first dragon (Group X).

### Method

Using the final fitted Random Forest model (`n_estimators = 50`, `max_depth = 10`), I predicted outcomes on the held-out test set. I computed the **precision** separately for Group X and Group Y, and then ran a **permutation test with 1000 iterations**. In each iteration, I shuffled the group assignments and recalculated the difference in precision between the two groups.

### Results and Conclusion

- **Observed Precision Difference (Y - X)**: **-0.0004**  
- **P-value**: **0.5400**

The observed precision difference between the two groups was extremely small and not statistically significant, with a p-value of 0.5400. As a result, we **fail to reject the null hypothesis**, meaning that we **do not find evidence of unfairness** in model performance between teams that secured the first dragon and those that did not.

This suggests that the model performs **equally well across both groups**, at least in terms of precision. While fairness should always be considered in deployment, the results here indicate no immediate concern based on this dimension of in-game performance.







