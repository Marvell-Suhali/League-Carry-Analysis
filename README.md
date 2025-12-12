# League Carry Analysis
DSC 80 Final Project – Predicting carry roles in League of Legends.

Introduction

This project analyzes professional League of Legends match data from the Oracle’s Elixir 2025 dataset, containing 118,932 player-game entries. Each row represents a single player's statistics from a match.

Guiding Question

Which role — Mid or Bot (ADC) — carries more often in professional play?
A carry is defined as the player who deals the highest share of team damage in a game.

Why this matters

Understanding carry behavior reveals:

how teams allocate resources,

how different roles influence game outcomes,

how we can predict carries from performance statistics.

Relevant Columns
Column	Description
position	Player role (Bot / Mid)
damageshare	Share of team damage dealt
earnedgoldshare	Share of team gold earned
teamkills, kpm, teamid	Team context statistics
Data Cleaning and Exploratory Data Analysis
Cleaning Steps

Filtered dataset to include only Bot and Mid players.

Removed rows missing key fields (position, damageshare, earnedgoldshare).

Created a new label:
is_carry = 1 if the player had the highest damage share on their team.

Insert your df.head() screenshot or table here.

Univariate Analysis
Distribution of Damage Share

(Embed Plotly histogram)
Most players deal 25–35% of their team’s damage. A long tail above 40% represents extreme carry games.

Distribution of Earned Gold Share

(Embed Plotly histogram)
Gold share is centered near 20%, but ADCs exhibit a heavier upper tail — consistent with teams funneling resources into them.

Bivariate Analysis
Damage Share by Role (Boxplot)

(Embed Plotly boxplot)
Bot laners show:

higher median damage share,

more variability,

suggesting they take on the primary carry role more frequently.

Gold Share vs Damage Share (Scatterplot)

(Embed Plotly scatter)
Gold share strongly correlates with damage share, especially for ADCs.

Interesting Aggregates
Role	Carry Rate
Bot (ADC)	1.28%
Mid	0.69%

ADCs carry nearly twice as often as mids.

Assessment of Missingness
NMAR Column

The url column is NMAR — URLs disappeared from Oracle’s Elixir due to a change in data publishing policy. This cannot be explained by any in-dataset feature.

Missingness Dependency

Chosen column: killsat25

Permutation tests showed:

Missingness depends on team kill count (teamkills)

Missingness does NOT depend on game result (result)

Interpretation: killsat25 is MAR, because it depends on another observed variable (teamkills).

Hypothesis Testing
Question

Do ADCs carry more often than Mid laners?

Hypotheses

H₀: ADC and Mid laners have equal carry rates.

H₁: ADC players carry more often.

Test Statistic

Difference in proportions: p_bot − p_mid

Results

Observed difference: 0.00594

p-value: < 0.001

Conclusion

We reject the null hypothesis.
Bot laners carry significantly more often than Mid laners, though the absolute difference is small.

Framing a Prediction Problem
Prediction Task

Predict whether a player will be their team’s carry.

Task Type

Binary classification.

Response Variable

is_carry

Evaluation Metric

Accuracy, chosen because:

dataset is not extremely imbalanced,

accuracy provides intuitive model comparison.

Baseline Model
Model

Logistic Regression

OneHotEncoding for position

Features:

damageshare

earnedgoldshare

position

Performance

Accuracy: 0.99185

The baseline already performs extremely well due to damage share being highly predictive.

Final Model
Engineered Features

gold_damage_ratio
Measures efficiency converting gold into damage.

kpm_share
Normalizes kill participation to team kill tempo.

Selected Model

RandomForestClassifier, because it captures nonlinear relationships logistic regression cannot.

Hyperparameters Tuned

n_estimators

max_depth

min_samples_split

Best Parameters
max_depth = 5
min_samples_split = 5
n_estimators = 100

Final Performance
Model	Accuracy
Baseline	0.99185
Final	0.99277

Accuracy improves slightly, but meaningfully, given the near-deterministic relationship between damage share and carry status.

Fairness Analysis
Comparison Groups

High gold share vs. Low gold share (median split)

Metric

Recall (ability to detect true carries)

Results
Group	Recall
High gold share	0.404
Low gold share	0.000
Hypotheses

H₀: Recall is equal across groups.

H₁: High-gold players have higher recall.

Permutation Test

Observed difference: 0.404

p-value: 0.000

Conclusion

We reject H₀.
The model is unfair — it performs dramatically better on high-gold players.

This reflects real gameplay: players with more gold naturally behave like carries, so the model is biased toward predicting them as such.

End of Report

Thanks for reading!
This project shows how competitive gaming statistics can be modeled, analyzed, and used to reveal strategic patterns in esports.



“Make this into a polished GitHub Pages site with styling.”

I can generate everything instantly.
