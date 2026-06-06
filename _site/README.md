 
# League of Legends: Predicting Match Outcomes Before the Game Begins

Author: Andrew Colvin

This analysis of League of Legends professional matches was created as a project requirement for UCSD Data Science class DSC80. It aims to explore the data to try to find information that teams may find useful during their pre-match strategizing. I focus almost exclusively on the data that is attained prior to the start of the game.

## Introduction

League of Legends is massively popular MOBA style online video game that pits two groups of five competing to destroy the other team's nexus. Before the chaos of the match ensues, the teams must draft their champions and are able to restrict the champions their opponents are able to choose through a process called banning. This stage of the game is an incredible pivotal time, as this determines the caliber of champions that will be utilized. In every match, each team has five roles that are fulfilled by these champions: top, jungle, mid, bot, and support. While champions change from match to match, the roles stay constant and are central to the game balance and design.

The data used for this study is from professional League of Legends matches from 2022 and sourced from Oracle's Elixir database. The initial dataset had over 150,000 rows and more than 150 columns with information ranging from individual player names to amount of gold acquired throughout a given match.

By utilizing this data, my project is aims to answer the question: **How accurately can the winner of a League of Legends match be predicted using only information available before the match begins?**

I restricted my analysis to pre-match information to research if this data can give us an insight to how much a game's outcome may already be determined before players enter the Summoner's Rift.

The pre-match columns that will be utilized are:

- `gameid`: the unique id number shared between all players and teams that were apart of the game.

- `teamid`: the unique id number for each team that competed in the dataset.

- `firstPick`: a binary variable, `1` if the team was the first to select during the pick process and `0` if team did not have first pick.

- `pick1` - `pick5` the five champions selected by a team during the draft, in order of pick.

- `ban1`- `ban5`: the five champions banned by a team during the draft, in order of ban.

- `league`: professional league that the match was played in.

- `playoffs`: a binary variable, `1` if the match recorded was a playoff match and `0` if it was not.

- `patch`: the game version that the match was played on.

- `result`: the outcome variable, also binary; `1` if the team won the match and `0` if they lost. This is the only column that was not produced before the match started.

- `position`: the role fulfilled by each player, also denotes team level data. This column was vital in building the cleaned dataset.


## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
Given my research focus, I restricted my dataset heavily, only including the above columns. But prior to cutting down columns, I utilized the `datacompleteness` category to remove all of the games that had incomplete data, removing 22,000 of our 150,000 observations.

I recognized that I would be working at a team level, meaning large portions of the dataset had either redundant or unnecessary information. Each of the observations was either an individual player row or a team level aggregate row. In other words, each match had 12 rows sharing the same `gameid`, 10 rows for the players (5 for each team) and 2 for the teams. I utilized the `position` column to isolate only team level rows from the dataset, reducing the working dataset to 21,000 rows.

When verifying that my team level data was clean, I checked that each unique gameid had exactly one team marked as having the first pick of the game. I found that 2,000 matches were missing information in the `firstPick` category, but not only that, those same rows were also missing any information about the pick phase in general, meaning I was also missing data from `pick1` - `pick5` categories. Since my analysis leans so heavily on these categories, I decided to drop these observations, bringing my team level dataset to 17,000 rows.

Currently, the pick and ban columns store information at the champion level, meaning they record the specific champions picked or banned in each match. For my pre-match analysis, I wanted to analyze this information at the role level instead. Since each team must fill the same five roles in every game, role-based features provide a more consistent way to compare teams across matches. Because this information was not directly available in the dataset, I needed to engineer new features that converted champion pick and ban data into role-based data.

In order to build the new pick by role columns (named `pick1_role` through `pick5_role`), I matched each picked champion in the team level draft data to the corresponding player level row from the same game and team. Since the player level data included both the champion played and the player's role, I was able to replace each picked champion with the role that champion filled in that specific match. This allowed me to represent draft picks at the role level rather than the champion level.

For the new ban by role columns, there was no player level row I could use to easily assign a role for each banned champion. I decided to build a champion to role map using the player level data. I assigned each champion a role based on the role it most commonly filled across the player level dataset. Since champions can fill multiple roles, this method may introduce some noise, but it gave a consistent way to convert champion bans to bans by role.

I then built those bans into columns counting how many of each role was banned by each team in each game. Those counts became the columns `top_bans`, `jng_bans`, `mid_bans`, `bot_bans`, and `sup_bans`. Finally, I added a `role_most_banned` column indicating which role each team targeted most often with its bans in that game.

After these cleaning steps, each observation in my dataset is represented by one team in one match, making it better for analyzing how different variables differ between winning and losing teams.

These are the first five rows of my cleaned team level dataset:

<style>
.table-wrapper {
  overflow-x: auto;
  margin: 1rem 0;
  text-align: center;
}

.table-wrapper table {
  display: inline-table !important;
  width: max-content !important;
  min-width: unset !important;
  border-collapse: collapse !important;
  margin: 1rem auto !important;
}

.table-wrapper th,
.table-wrapper td {
  border: 1px solid #e5e5e5 !important;
  padding: 8px 16px !important;
  text-align: center !important;
  white-space: nowrap !important;
}

.table-wrapper th {
  background-color: #f8f8f8 !important;
  font-weight: bold !important;
}
</style>


<table class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>gameid</th>
      <th>league</th>
      <th>playoffs</th>
      <th>patch</th>
      <th>teamid</th>
      <th>firstPick</th>
      <th>result</th>
      <th>pick1_role</th>
      <th>pick2_role</th>
      <th>pick3_role</th>
      <th>pick4_role</th>
      <th>pick5_role</th>
      <th>top_bans</th>
      <th>jng_bans</th>
      <th>mid_bans</th>
      <th>bot_bans</th>
      <th>sup_bans</th>
      <th>most_banned_role</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ESPORTSTMNT01_2690210</td>
      <td>LCKC</td>
      <td>0</td>
      <td>12.01</td>
      <td>oe:team:fa3d687a87bcae80362f784a7da571d</td>
      <td>1.0</td>
      <td>0</td>
      <td>top</td>
      <td>bot</td>
      <td>jng</td>
      <td>mid</td>
      <td>sup</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>sup</td>
    </tr>
    <tr>
      <td>ESPORTSTMNT01_2690210</td>
      <td>LCKC</td>
      <td>0</td>
      <td>12.01</td>
      <td>oe:team:7c64febcd5ccff13dcd035dc6867a00</td>
      <td>0.0</td>
      <td>1</td>
      <td>bot</td>
      <td>jng</td>
      <td>top</td>
      <td>mid</td>
      <td>sup</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>mid</td>
    </tr>
    <tr>
      <td>ESPORTSTMNT01_2690219</td>
      <td>LCKC</td>
      <td>0</td>
      <td>12.01</td>
      <td>oe:team:731b7a9fd004cdbe2bcb3da795bce47</td>
      <td>1.0</td>
      <td>0</td>
      <td>jng</td>
      <td>bot</td>
      <td>top</td>
      <td>sup</td>
      <td>mid</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>sup</td>
    </tr>
    <tr>
      <td>ESPORTSTMNT01_2690219</td>
      <td>LCKC</td>
      <td>0</td>
      <td>12.01</td>
      <td>oe:team:e7a7c6bf58eb268ed3f13aac4158aa8</td>
      <td>0.0</td>
      <td>1</td>
      <td>mid</td>
      <td>bot</td>
      <td>jng</td>
      <td>sup</td>
      <td>top</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>3</td>
      <td>sup</td>
    </tr>
    <tr>
      <td>ESPORTSTMNT01_2690227</td>
      <td>LCKC</td>
      <td>0</td>
      <td>12.01</td>
      <td>oe:team:b9733b8e8aa341319bbaf1035198a28</td>
      <td>1.0</td>
      <td>1</td>
      <td>top</td>
      <td>jng</td>
      <td>sup</td>
      <td>bot</td>
      <td>mid</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>mid</td>
    </tr>
  </tbody>
</table>


### Exploratory Data Analysis

My initial intuition was to see if there was a difference in the outcome of a match based on who had the first pick of the draft.

<iframe
  src="team-win-rate-by-draft-order.html"
  width="650"
  height="425"
  frameborder="0"
  style="display: block; margin: 0 auto; border: 0;"
></iframe> 

This bivariate bar chart creates an informative visualization of the difference between having first and second picks. TTeams with the first pick had a win rate that was 4.08 percentage points higher than teams without the first pick. This leads me to believe there may be an association between first pick and game outcome.
<br>

---

<br>
I decided I wanted to explore how those teams with the first pick faired based on the role of the champion they picked first.

<iframe
  src="win-rate-by-first-picked-role.html"
  width= "650"
  height="425"
  frameborder="0"
  style="display: block; margin: 0 auto; border: 0;"
></iframe> 

This bivariate graph shows very consistent win rates for bot, jungle, support, and top. The mid role stands out because teams that first picked a mid champion had a win rate below 50%, while the other roles were around 52%.
<br>

---

<br>

Next I wanted to start looking to bans. I was initially curious how professional teams tend to distribute their bans.
<iframe
  src="distribution-of-bans-by-role.html"
  width="600"
  height="425"
  frameborder="0"
  style="display: block; margin: 0 auto; border: 0;"
></iframe> 

This univariate pie chart shows that the breakdown between ban roles is fairly even. The support role is the lowest at 15.2%. Support champions being a less common target for bans may be because bans are targeted towards roles that have a higher chance of "carrying" the team.

---

<br>

While ban distribution is interesting, it lead me to wonder if the role that teams targeted most often in their banning phase was associated with game outcome.

<iframe
  src="win-rate-by-most-frequently-targeted-ban-role.html"
  width="650"
  height="425"
  frameborder="0"
  style="display: block; margin: 0 auto; border: 0;"
></iframe> 

This bivariate bar chart again shows one role standing out above the rest. With team who's most targeted ban was a champion associated with the role of jungle led to a 52.79% win rate. 

**Note:** This chart was built from the `role_most_banned` column, which had some limitations. Since there were instances of ties for which role was most banned by a team (ex. each role was banned exactly once during the banning phase), I took the first value in that list of ties.

### Interesting Aggregates

To further investigate importance of role ban allocation, and given the limitations of the `role_most_banned` column, I built a data frame to try to assess how the outcome of the match really is associated with bans.


<table class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Role</th>
      <th>Wins</th>
      <th>Losses</th>
      <th>Difference</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Top</td>
      <td>1.016</td>
      <td>1.039</td>
      <td>-0.023</td>
    </tr>
    <tr>
      <td>Jungle</td>
      <td>0.964</td>
      <td>0.907</td>
      <td>0.057</td>
    </tr>
    <tr>
      <td>Mid</td>
      <td>1.226</td>
      <td>1.233</td>
      <td>-0.007</td>
    </tr>
    <tr>
      <td>Bot</td>
      <td>1.037</td>
      <td>1.064</td>
      <td>-0.027</td>
    </tr>
    <tr>
      <td>Support</td>
      <td>0.758</td>
      <td>0.758</td>
      <td>0.000</td>
    </tr>
  </tbody>
</table>

This table compares the average number of bans each team used on each role, separated by winning and losing teams. Aggregating the actual counts of team bans reduces the noise that limited my previous chart. By showing the difference, we can still see that the gap between winning and losing teams ban rates was largest for the role of jungle. 



## Assessment of Missingness
### NMAR Analysis

The column `teamid` could be NMAR. There are many rows where `teamid` is missing, but `teamname` exists. One possible explanation is that teams that were very recently created or newly registered at the time of their first match. So the missingness in that case would not be MAR but attached to a variable that we don't currently have access to, or NMAR. We could potentially explain the missing values if we had access to to team creation/registration date.

### Missingness Dependency

To find data to analyze for missingness I explored the original dataset. I found that there were multiple columns that took measurements of a game variable at a certain time, meaning if a game does not extend to that time, the data would be missing. This is an example of MAR. One of these columns is `goldat25`. I decided to look into `goldat25`'s missingness relation ship to `gamelength` to see if there might be an association. I explored it further by grouping rows by `goldat25` missing vs not missing and ploted the `gamelength` for each of these groups.

<iframe
  src="game-length-by-gold-at-25.html"
  width="850"
  height="425"
  frameborder="0"
  style="display: block; margin: 0 auto; border: 0;"
></iframe> 

This box plot gives us a good indication that the missingness of `goldat25` may be associated with `gamelength`.

### Testing Dependency: Dependent
To test this I performed a permutation test with the following hypothesis:

**Null Hypothesis:** The missingness of `goldat25` is independent of `gamelength`.

**Alternative Hypothesis:** The missingness of `goldat25` depends on `gamelength`.

**Test Statistic:** Absolute difference between the averages of game length between games that are missing `goldat25` data and those that are not missing. 

**Significance Level:** α =  0.05

The observed statistic for our data for the absolute difference in mean game length between the two groups, was 598 seconds, or close to 10 minutes.

After running a permutation test with 10,000 simulations, I found not a single permutation had a more extreme value than our observed statistic. Meaning the resulting **p-value** was less than 0.0001, much less than our **significance level** of α = 0.05, therefore we *reject the null hypothesis*. This is strong evidence that the missingness of `goldat25` depends on `gamelength`.

To further visualize the strength of this association we can look at the histogram of simulations with the observed statistic plotted:

<iframe
  src="permutation-distribution-for-gold-at-25-missingness.html"
  width="850"
  height="425"
  frameborder="0"
  style="display: block; margin: 0 auto; border: 0;"
></iframe> 

### Testing Dependency: Independent
That same column of `goldat25`'s missingness may be dependent on `gamelength` but was it independent from any of the columns in our dataset? I decided to check how it was associated with the column `position`, which remember just holds the role the competitors are playing. I hypothesized that the missingness of `goldat25` should have no association with `position`.

<table class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>position</th>
      <th>bot</th>
      <th>jng</th>
      <th>mid</th>
      <th>sup</th>
      <th>team</th>
      <th>top</th>
    </tr>
    <tr>
      <th>goldat25_missing</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>False</th>
      <td>0.166667</td>
      <td>0.166667</td>
      <td>0.166667</td>
      <td>0.166667</td>
      <td>0.166667</td>
      <td>0.166667</td>
    </tr>
    <tr>
      <th>True</th>
      <td>0.166667</td>
      <td>0.166667</td>
      <td>0.166667</td>
      <td>0.166667</td>
      <td>0.166667</td>
      <td>0.166667</td>
    </tr>
  </tbody>
</table>

By grouping the existence of missingness for `goldat25` by the `position`, we can see the proportion of each position is the same. This leads me to hypothesize that indeed `goldat25`'s missingness is not explained by `position`.


**Null Hypothesis:** The missingness of `goldat25` is independent of `position`.

**Alternative Hypothesis:** The missingness of `goldat25` depends on `position`.

**Test Statistic:** TVD between distributions of positions between games that are missing `goldat25` data and those that are not missing. 

**Significance Level:** α = 0.05

**Observed statistic:** TVD = 0.0

After running a permutation test with 10,000 simulations, I found not a single permutation had a less extreme value than our observed statistic. Meaning the resulting **p-value** was 1.0, which is much greater of our **significance level** of α = 0.05, therefore we *fail to reject the null hypothesis*. This is strong evidence that the missingness of `goldat25` is independent of  `position`.

To further visualize the strength of this association we can look at the histogram of simulations with the observed statistic plotted:

<iframe
  src="permutation-distribution-for-gold-at-25-position.html"
  width="860"
  height="425"
  frameborder="0"
  style="display: block; margin: 0 auto; border: 0;"
></iframe> 

## Hypothesis Testing

During my EDA phase, I found that there were marginal differences in the ban rate of certain roles between winning and losing teams. To assess if this difference is meaningful enough to explore as a feature in my prediction model, we'll perform a permutation test on the following hypothesis.

**Null Hypothesis:** The game outcome is independent of the distribution of bans by role.

**Alternative Hypothesis:** The game outcome is dependent on the distribution of bans by role.

**Test Statistic:** Sum of the absolute differences between the mean ban rates of winning and losing teams across all roles.

**Significance Level:** α = 0.05

After running a permutation test with 10,000 simulations, I found a **p-value** of about 0.01, which is less than our **significance level** of α = 0.05, and therefore we *reject the null hypothesis*. This is evidence that this difference is unlikely to be due to random chance, and that the game outcome is associated with distribution of bans by role.

Again, plotting the observed statistic on our permutation test to visualize where our observed statistic lands within permutation test.

<iframe
  src="hypothesis-testing.html"
  width="850"
  height="425"
  frameborder="0"
  style="display: block; margin: 0 auto; border: 0;"
></iframe> 

## Framing a Prediction Problem

I set out to predict whether pre-match information can predict the outcome of a League of Legends match. This means my model will only use information known before the game begins, such as draft order, pick roles, ban roles, patch, league, and team. I wont have access to any information generated during the match, such as kills or gold earned.

My response variable, the outcome of the match, is represented by the `result` column of the dataset. Therefore since there can be no ties, only winners and losers, we are dealing with a binary classification. 

To evaluate my model I used accuracy, as it will measure how often(by proportion) my model predicts the correct result. Since there is a clear distinction between outcomes and the outcomes are balanced, accuracy is the most natural metric to use. F1-score would be more suited if my response variable was not balanced. For instance, if I was trying to predict if the team with first pick won. In my dataset there is an unequal number of teams that had first pick that won and lost.

The task of predicting a winner based only on pre-match data is ambitious, but if the model can perform even marginally better than a coin flip, it suggests that pre-match strategy contains meaningful information about the eventual outcome of a game.


## Baseline Model
For the baseline model I decided to test the number of bans allocated to each role. This specifically interested me because of the findings from my hypothesis test, where I investigated if ban allocation by role was associated with game outcomes. I used five features in my bare bones baseline model.

Baseline Model Features (type):
- `top_bans` (quantitative)
- `jng_bans` (quantitative)
- `mid_bans` (quantitative)
- `bot_bans` (quantitative)
- `sup_bans` (quantitative)

I chose to use logistic regression, it naturally works well for my response variable (`result`) since it is binary, and used 10 fold cross-validation for prediction training.

My initial model had an average cross-validated accuracy score of 50.7%. Not a very strong model, and only slightly better than a coin flip. So while, the outcome of the match is likely dependent on ban allocation by role, it is not solely able to predict outcome.

<br>

---
<br>

**Intermediate Model**

I decided to try and improve upon the base model by adding more pre-match features. It occurred to me that the most predictive pre-match feature is most likely team strength. For my final model I will implement a way to measure team strength, but first I wanted to see how the model faired with less impactful features. My intermediate model used 13 features.

Intermediate Model Features (type):
- `top_bans` (quantitative)
- `jng_bans` (quantitative)
- `mid_bans` (quantitative)
- `bot_bans` (quantitative)
- `sup_bans` (quantitative)
- `pick1_role` (nominal)
- `pick2_role` (nominal)
- `pick3_role` (nominal)
- `pick4_role` (nominal)
- `pick5_role` (nominal)
- `firstPick` (nominal)
- `patch` (nominal)
- `league` (nominal)

This model used categorical (nominal) features, so I incorporated `OneHotEncoder` to be able to use logistic regression on those features.

My intermediate model achieved an average cross-validated accuracy of 51.2%, so while it did improve it is still not much better than a coin flip.

## Final Model
For the final model I added `teamid` to the same features used in my intermediate model. Since `teamid` is also categorical, I added it to the `OneHotEncoder` transformer. By including `teamid` I can get a indirect measure of team strength. This enables the model to learn patterns for teams in the training data, which can start to measure a team's likeliness to win. I continued using logistic regression used in my previous models. My final model used 14 features.

Final Model Features (type):
- `top_bans` (quantitative)
- `jng_bans` (quantitative)
- `mid_bans` (quantitative)
- `bot_bans` (quantitative)
- `sup_bans` (quantitative)
- `pick1_role` (nominal)
- `pick2_role` (nominal)
- `pick3_role` (nominal)
- `pick4_role` (nominal)
- `pick5_role` (nominal)
- `firstPick` (nominal)
- `patch` (nominal)
- `league` (nominal)
- `teamid` (nominal)



The final model achieved an average prediction accuracy, prior to tuning, of 59.1%, which is by far the most meaningful increase seen while modeling. All from just adding one additional feature. This suggests that team strength captures important information about match outcome that draft related features alone do not fully explain.
<br>

---

<br>
Finally I tested some regularization parameters on my model. I tested six values of `C`, increasing by factors of 10 (seen below) as my hyperparamters to tune the model. The value of `C` controls how strongly the model tries to fit the training data, so I want to see if there is some value of `C` that balances simplicity and flexibility for the model.

The resulting data frame shows that the regularization parameter reaches a maximum for prediction accuracy at around `C = 100` where its accuracy is at 59.7%. 

<table class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>C</th>
      <th>Accuracy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0.1</td>
      <td>0.587755</td>
    </tr>
    <tr>
      <td>1.0</td>
      <td>0.590917</td>
    </tr>
    <tr>
      <td>10.0</td>
      <td>0.593721</td>
    </tr>
    <tr>
      <td>100.0</td>
      <td>0.597361</td>
    </tr>
    <tr>
      <td>1000.0</td>
      <td>0.596764</td>
    </tr>
    <tr>
      <td>10000.0</td>
      <td>0.595989</td>
    </tr>
  </tbody>
</table>

## Fairness Analysis

During the EDA phase, one of the variables that stood out as an interesting grouping was playoff matches vs non-playoff matches. In this section, I evaluated if there is a difference in accuracy for my model when trying to predict these two groups. Since my model's strongest feature was `teamid`, which indirectly measured team strength. I hypothesized that non-playoff games would have a higher prediction accuracy than playoff matches. Playoff teams should generally be closer in team strength to one another, weakening the ability for the model to predict the outcome.

**Playoff Groups**

X: Non-playoff matches 

Y: Playoff matches

**Metric:** Accuracy

**Null Hypothesis:** Model's accuracy is the same for both playoff and non-playoff matches.

**Alternative Hypothesis:** Model's accuracy is higher for non-playoff matches than for playoff matches.

**Test Statistic:** Difference in accuracy between non-playoff and playoff matches. (Accuracy of non-playoff matches - accuracy of playoff matches)

**Significance Level:** α = 0.05

The observed statistic was 0.0546. The model correctly predicted the outcome of non-playoff matches around 5 percentage points higher than playoff matches.

After running a permutation test with 10,000 simulations, I found a **p-value** of less than 0.0001, which is less than our **significance level** of α = 0.05, and therefore we *reject the null hypothesis*. This is strong evidence that the the model's accuracy is higher for non-playoff matches than playoff matches.

I created a visual representation of my analysis by plotting the observed statistic on a histogram of the permutations created during our permutation test.

<iframe
  src="playoff.html"
  width="850"
  height="425"
  frameborder="0"
  style="display: block; margin: 0 auto; border: 0;"
></iframe> 