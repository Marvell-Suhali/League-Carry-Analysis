<h1>League Carry Analysis – DSC 80 Final Project</h1>
<h3><em>Predicting carry roles in League of Legends esports</em></h3>
<p><strong>Author:</strong> Marvell Suhali</p>
<hr />

<h1>Introduction</h1>

<p>
This project analyzes professional League of Legends match data from the
<strong>Oracle’s Elixir 2025 dataset</strong>, containing approximately
<strong>118,932 player-game records</strong>. Each row corresponds to a single
player’s in-game performance for one match.
</p>

<h3>Guiding Question</h3>
<p><strong>Which role — Mid or Bot (ADC) — takes on the carry role more often in professional play?</strong></p>
<p>
A <em>carry</em> is defined as the player on a team who deals the highest
percentage of their team’s total damage.
</p>

<h3>Why This Matters</h3>
<ul>
  <li>Shows how teams allocate damage and gold resources.</li>
  <li>Reveals which roles drive match outcomes more often.</li>
  <li>Provides a base for predicting carry roles from player statistics.</li>
</ul>

<h3>Relevant Columns</h3>
<table>
  <thead>
    <tr>
      <th>Column</th>
      <th>Meaning</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>position</code></td>
      <td>Player role (Bot or Mid)</td>
    </tr>
    <tr>
      <td><code>damageshare</code></td>
      <td>Percentage of total team damage dealt by the player</td>
    </tr>
    <tr>
      <td><code>earnedgoldshare</code></td>
      <td>Percentage of total team gold earned by the player</td>
    </tr>
    <tr>
      <td><code>teamkills</code>, <code>kpm</code></td>
      <td>Team kill counts and kill pace (kills per minute)</td>
    </tr>
    <tr>
      <td><code>teamid</code></td>
      <td>Team identifier for grouping and carry detection</td>
    </tr>
  </tbody>
</table>

<hr />

<h1>Data Cleaning and Exploratory Data Analysis</h1>

<h2>Data Cleaning</h2>
<ul>
  <li>Filtered the dataset to include only <strong>Mid</strong> and <strong>Bot</strong> laners.</li>
  <li>Dropped rows missing <code>position</code>, <code>damageshare</code>, or <code>earnedgoldshare</code>.</li>
  <li>
    Constructed a new binary target column
    <code>is_carry</code>:
    <strong>1</strong> if a player had the highest damage share on their team,
    <strong>0</strong> otherwise.
  </li>
</ul>

<h2>Univariate Analysis</h2>
<h3>Damage Share Distribution</h3>
<p>
Most players deal between <strong>25–35%</strong> of their team’s damage.
A long right tail above 40% captures extreme carry performances where a single
player heavily dominates damage output.
</p>


<iframe
  src="assets/damage_share_distribution.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>


<h3>Earned Gold Share Distribution</h3>
<p>
Earned gold share is tightly centered around <strong>20%</strong> for both roles.
However, the ADC (Bot) distribution shows a heavier tail above 35%, indicating
that Bot laners more often command a larger share of their team’s total gold.
</p>

<iframe
  src="assets/dearned_gold_share_distribution.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>


<h2>Bivariate Analysis</h2>

<h3>Damage Share by Role</h3>
<p>
Comparing damage share by role, Bot laners exhibit both a higher mean damage share
and more extreme high-damage games compared to Mid laners, reinforcing the idea
that ADCs are traditional late-game carries.
</p>

<iframe
  src="assets/damage_share_by_role.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

<h3>Gold vs Damage Share</h3>
<p>
A scatter plot of <code>earnedgoldshare</code> versus <code>damageshare</code> shows a
strong positive relationship, especially for Bot players. Players with more gold
almost always convert that gold into higher damage share, which is consistent with
carry roles being resource-intensive.
</p>

<iframe
  src="assets/gold_vs_damage.html"
  width="800"
  height="600"
  frameborder="0">
</iframe>

<h2>Interesting Aggregates</h2>

<table>
  <thead>
    <tr>
      <th>Role</th>
      <th>Carry Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Bot (ADC)</strong></td>
      <td><strong>1.28%</strong></td>
    </tr>
    <tr>
      <td><strong>Mid</strong></td>
      <td><strong>0.69%</strong></td>
    </tr>
  </tbody>
</table>

<p>
Bot laners are almost <strong>twice as likely</strong> to be their team’s carry
compared to Mid laners, although the absolute probabilities are still very small
because each game has exactly one carry per team of five players.
</p>

<hr />

<h1>Assessment of Missingness</h1>

<h2>NMAR Analysis</h2>
<p>
The <code>url</code> column, which originally pointed to match pages on external
websites, is plausibly <strong>NMAR</strong> (Not Missing At Random). The presence or
absence of a URL depends on external logging and publishing practices, such as
Oracle’s Elixir changing how match pages are linked. This mechanism is not fully
explained by any variables in the dataset itself, so additional metadata about
website logging practices would be required to turn this into MAR.
</p>

<h2>Missingness Dependency</h2>
<p>
We analyzed missingness for a column with non-trivial missing values
(e.g. <code>killsat25</code>) by performing permutation tests:
</p>
<ul>
  <li>
    Tested dependence of <code>killsat25</code> missingness on
    <code>teamkills</code>.
  </li>
  <li>
    Tested independence of <code>killsat25</code> missingness from
    <code>result</code> (win vs loss).
  </li>
</ul>

<p>
Permutation tests indicated that missingness in <code>killsat25</code>
<strong>depends on</strong> <code>teamkills</code> but <strong>does not depend on</strong>
<code>result</code>. This suggests a <strong>MAR</strong> mechanism where the missingness is tied
to game tempo statistics but not directly to the game outcome itself.
</p>

<hr />

<h1>Hypothesis Testing</h1>

<h2>Question</h2>
<p><strong>Do Bot laners carry more often than Mid laners?</strong></p>

<h2>Hypotheses</h2>
<ul>
  <li><strong>Null (H₀):</strong> Bot and Mid players have equal carry rates.</li>
  <li><strong>Alternative (H₁):</strong> Bot laners have a higher carry rate.</li>
</ul>

<h2>Test Statistic</h2>
<p>
Difference in proportions:
<code>p_bot − p_mid</code>, where <code>p_role</code> is the proportion of players in that role
with <code>is_carry = 1</code>.
</p>

<h2>Results</h2>
<ul>
  <li>Observed difference: <strong>0.00594</strong></li>
  <li>Right-tailed permutation test with 2000 repetitions.</li>
  <li>p-value: <strong>&lt; 0.001</strong></li>
</ul>

<h2>Conclusion</h2>
<p>
We reject the null hypothesis. There is strong statistical evidence that
<strong>Bot laners are more likely to be carries than Mid laners</strong>, though the absolute
difference in probabilities is small because each team has only one carry per
five players.
</p>

<hr />

<h1>Framing a Prediction Problem</h1>

<h2>Prediction Task</h2>
<p>
We formulate a prediction problem: given a player’s in-game statistics, can we
predict whether they will be their team’s <strong>carry</strong>?
</p>

<ul>
  <li><strong>Task type:</strong> Binary classification</li>
  <li><strong>Response variable:</strong> <code>is_carry</code> (1 = carry, 0 = not carry)</li>
</ul>

<h2>Evaluation Metric</h2>
<p>
We use <strong>accuracy</strong> as the primary evaluation metric. The dataset is not severely
imbalanced, and accuracy provides a simple, interpretable baseline for comparing
models. Other metrics (precision/recall) are considered in the fairness analysis.
</p>

<hr />

<h1>Baseline Model</h1>

<h2>Model Description</h2>
<ul>
  <li>Model: <strong>LogisticRegression</strong> in a single sklearn <code>Pipeline</code></li>
  <li>
    Features:
    <ul>
      <li><code>damageshare</code> (quantitative)</li>
      <li><code>earnedgoldshare</code> (quantitative)</li>
      <li><code>position</code> (nominal, encoded using <code>OneHotEncoder</code>)</li>
    </ul>
  </li>
  <li>Train-test split: 80/20</li>
</ul>

<h2>Performance</h2>
<p>
The baseline model achieves an accuracy of
<strong>0.99185</strong> on the test set.
</p>
<p>
This high accuracy is expected because damage share is extremely predictive of
the carry label by construction: the carry is defined precisely as the highest
damage share on the team.
</p>

<hr />

<h1>Final Model</h1>

<h2>Feature Engineering</h2>
<p>To move beyond the baseline, we engineered two additional features:</p>
<ol>
  <li>
    <strong><code>gold_damage_ratio</code></strong> –
    damage share divided by gold share. This measures
    how efficiently a player converts gold into damage; carries tend to turn
    resources into damage more effectively.
  </li>
  <li>
    <strong><code>kpm_share</code></strong> –
    a normalized measure of how involved a player is in kills relative to team
    tempo. Players with high kill participation in fast-paced games are more
    likely to be central carries.
  </li>
</ol>

<h2>Model Choice</h2>
<p>
We switch to a <strong>RandomForestClassifier</strong> in a single sklearn
<code>Pipeline</code> that:
</p>
<ul>
  <li>One-hot encodes <code>position</code>,</li>
  <li>Scales numeric features,</li>
  <li>Trains a random forest on the transformed data.</li>
</ul>

<h2>Hyperparameter Tuning</h2>
<p>
We tuned the following hyperparameters using <code>GridSearchCV</code>:
</p>
<ul>
  <li><code>n_estimators</code> – number of trees (model stability)</li>
  <li><code>max_depth</code> – tree depth (controls complexity and overfitting)</li>
  <li><code>min_samples_split</code> – minimum samples to split a node (regularization)</li>
</ul>

<p><strong>Best hyperparameters:</strong></p>
<pre><code>{
  "max_depth": 5,
  "min_samples_split": 5,
  "n_estimators": 100
}
</code></pre>

<h2>Performance Comparison</h2>
<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>Accuracy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Baseline (Logistic Regression)</td>
      <td>0.99185</td>
    </tr>
    <tr>
      <td>Final Model (Random Forest)</td>
      <td><strong>0.99277</strong></td>
    </tr>
  </tbody>
</table>

<p>
The final model yields a slightly higher accuracy than the baseline while using
richer features and a nonlinear classifier. The improvement is small but
consistent, demonstrating that engineered features capturing efficiency and
kill involvement can add incremental predictive power beyond raw damage and
gold shares.
</p>

<hr />

<h1>Fairness Analysis</h1>

<h2>Groups and Metric</h2>
<p>
We examine whether the final model is fair with respect to a player’s resource
level. We define:
</p>
<ul>
  <li><strong>Group X (High-gold players):</strong> players with <code>earnedgoldshare</code> above the median.</li>
  <li><strong>Group Y (Low-gold players):</strong> players with <code>earnedgoldshare</code> at or below the median.</li>
</ul>

<p>
To assess fairness, we use <strong>recall</strong> (true positive rate) for the
<code>is_carry = 1</code> class within each group. Recall measures how often the model
correctly identifies true carries.
</p>

<h2>Group Recalls</h2>
<table>
  <thead>
    <tr>
      <th>Group</th>
      <th>Recall</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>High-gold players</td>
      <td><strong>0.404</strong></td>
    </tr>
    <tr>
      <td>Low-gold players</td>
      <td><strong>0.000</strong></td>
    </tr>
  </tbody>
</table>

<h2>Hypotheses</h2>
<ul>
  <li>
    <strong>Null (H₀):</strong> The model is fair. Recall for high-gold and low-gold
    players is roughly equal; any difference is due to random chance.
  </li>
  <li>
    <strong>Alternative (H₁):</strong> The model is unfair. Recall for high-gold
    players is higher than recall for low-gold players.
  </li>
</ul>

<h2>Permutation Test</h2>
<p>
We computed the observed difference in recall:
</p>
<p><strong>Observed statistic:</strong> recall_high − recall_low = 0.404</p>
<p>
Then we performed a one-sided permutation test by shuffling the carry labels
(<code>y_test</code>) 2000 times, recomputing recall for high-gold and low-gold groups
on each shuffled dataset, and recording the differences.
</p>
<p>
The resulting p-value was approximately <strong>0.000</strong>, meaning none of the
permuted differences were as large as the observed difference.
</p>

<h2>Fairness Conclusion</h2>
<p>
We reject the null hypothesis. The model is <strong>not fair</strong> across gold groups:
high-gold players are detected as carries much more often than low-gold players.
</p>
<p>
Some of this disparity reflects the underlying game reality – players with more
gold truly are more likely to be carries – but it also means the model almost
never flags low-gold players as carries, even in the rare games where they
heavily overperform their resources.
</p>

<hr />

<h1>Conclusion</h1>

<p>
This project explored how professional League of Legends data can be used to
understand and predict carry behavior.
</p>

<ul>
  <li>Bot laners are statistically more likely to be carries than Mid laners.</li>
  <li>Damage share and gold share are extremely strong predictors of carry status.</li>
  <li>
    A Random Forest model with engineered features (<code>gold_damage_ratio</code>,
    <code>kpm_share</code>) slightly improves accuracy over a baseline logistic regression.
  </li>
  <li>
    Fairness analysis reveals that the model performs much better for high-gold
    players than for low-gold players, highlighting a resource-driven bias.
  </li>
</ul>

<p>
Overall, this project demonstrates how data science can provide strategic
insights into esports while also emphasizing the importance of evaluating
models beyond raw accuracy, especially when different groups may experience
very different prediction quality.
</p>


