<script type="text/javascript" async="" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# _League of Legends Analysis_
 *This is DSC 80 Project 4 where we clean, explore, and make predictions upon, League of Legends game data. This website stands as a report of our findings.*

**Names:** Rihui Ling and Minghan Wu


## Introduction

Our dataset is on all of the professional League of Legends games between 2014 and 2024 (inclusive).
The dataset contains one row per game per team (so two rows per game). The game data is comprehensive of most aspects of the game. Our cleaned game data has ```131414 rows × 27 columns```.
In this report, we discuss one question to perform hypothesis test on, and one prediction problem.

*Hypotheses:*

In the history of LoL, the choice of side (blue or red) is often believed to have an influence on the team winning rate, with the blue side having a greater chance of winning. For this reason, we put forth this question:

_Will the choice of side by a team in a game affect the team kills?_

* $$H_0$$: The blue side has the same expected team kills as the red side.

* $$H_1$$: The blue side has higher expected team kills than the red side.

*Prediction Problem:*
* Predict whether a team is winning/losing a game, given other data.

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

1 We only consider team data, so drop all player rows, only keeping the rows where ```position``` is ```team```

2 Drop all columns that are all na values, which is not associated with teams

3 Keep only the columns useful for our analysis

4 Convert ```patch``` to major patch

5 Drop all rows where ```gamelength``` is greater than 2 hrs (since the longest game in the history of LOL is about 1h30min), convert the unit of ```gamelength``` from s to min

6 drop rows that the earned gold is less than 0, it should only contain positive value

7 Convert all binary encoded columns to ```bool```

8 Convert all numerical column with integral values to ```int```

Here are the columns we decide to keep:

* Basic game stats: 
    * ```game```: the ordinal number of the game in the competition
    * ```gameid```: the game id
    * ```patch``` the major patch number
    * ```gamelength``` the time span of the game in minutes
* Basic team stats: 
    * ```side```: the side of the team, blue or red 
    * ```teamname```: the team name
    * ```teamid```: the team id
* Team game results: 
    * ```result```: result of the team in the game
    * ```kills```: the total kills of the team in the game
    * ```deaths```: the total deaths of the team in the game
    * ```assists```: the total assists of the team in the game
    * ```firstblood```: whether the team got first blood
    * ```damagetochampions```: the total damage made to the enemy team
    * ```towers```: the number of towers got by the team
    * ```opp_towers```: the number of towers got by the enemy team
    * ```firstmidtower```: whether the team got the first middle tower
    * ```firsttothreetowers```: whether the team got three towers before the enemy team
* General game resources: 
    * ```earnedgold```: the total number of gold earned by the team
    * ```firstdragon```: whether the team got the first dragon
    * ```dragons```: the total number of dragons got by the team
    * ```opp_dragons```: the total number of dragons got by the enemy team
    * ```firstherald```: whether the team got the first herald
    * ```elders```: the total number of elders got by the team
    * ```opp_elders```: the total number of elders got by the enemy team
    * ```firstbaron```: whether the team got the first baron
    * ```barons```: the total number of barons got by the team
    * ```opp_barons```: the total number of barons got by the enemy team

This table shows the first several rows of the dataframe after data cleaning:

<iframe 
src="table/teams_cleaned.html" 
width=800 
height=200
frameBorder=0>
</iframe>

### Univariate Analysis

Here is a histogram showing the distribution of ```gamelength```:

<iframe 
src="img/distribution_of_game_length_(min).html" 
width=800 
height=500
frameBorder=0>
</iframe>

We can see that the data is roughly normally distributed, centering around 32.5 min. Note that there are outliers on both ends of the graph (even after we remove the extreme values), although their density is too small to be visible.

### Bivariate Analysis

Here is a graph showing the distributions of gold earned by the team in the game conditional on game results (```win``` or ```lose```):

<iframe 
src="img/distribution_of_earned_gold_each_game_by_result.html" 
width=800
height=600
frameBorder=0>
</iframe>

Both distributions are roughly normally distributed, with the ```win``` distribution more concentrated than the ```lose``` distribution, and with higher mean.

### Interesting Aggregates

The following pivot table shows the average team kills for lost and won teams for different years.

<iframe 
src="table/avg_kills.html" 
width=800 
height=400
frameBorder=0>
</iframe>

We can see that for each year, the difference between average team kills for won teams and lost teams are around 10. Also, the average team kills for both won teams and lost teams increased significantly since 2021.

## Missingness Mechanisms

In this section, we analyze the missing mechanisms of several of the columns. To give an overview, here is a DataFrame showing how many rows are missing for each column:

<iframe 
src="table/col_missing.html" 
width=800
height=250
frameBorder=0>
</iframe>

### Not Missing at Random (NMAR)

We believe the missingness of ```game``` (which we dropped in the cleaning process) is NMAR. Upon researching online, we discover that the missingness results largely from the fact that there is only one game in the split in that league and year.

As such, the missing mechanism of ```game``` is NMAR, with value 1 (representing the first game) being significantly more prone to be missing than other values.

### Missing at Random (MAR)

We believe the missingness of ```elders``` is dependent on ```patch```. To confirm this observation, we run a permutation test on the two columns (using Total Variation Distance (tvd) as statistic).

This is the observed conditional distribution:

<iframe 
src="img/distribution_of_patch_by_missingness_of_elders.html" 
width=800
height=600
frameBorder=0>
</iframe>

This is the DataFrame showing the same distribution:

<iframe 
src="table/elder_missingness.html" 
width=800
height=400
frameBorder=0>
</iframe>

After running the permutation and computing Total Variation Distance (TVD) 500 times, this is the distribution we get:

<iframe 
src="img/permutation_test_of_the_missingness_of_elders_depend_on_patch.html" 
width=800
height=600
frameBorder=0>
</iframe>

We see that the observed tvd 1.2765 is significantly greater than most of the tvd values resulted from distribution. The p-value for the permutation test is 0.0. 

Hence we conclude that the missingness of ```elders``` is dependent on ```patch```, making the former missing at random.

### Missing Completely at Random (MCAR)

We believe the missing mechanism of ```barons``` is completely at random. To confirm this, we run permutation tests against each every one of other columns. 

For numerical columns, we compute the absolute differences between the means of the missing group and the non-missing group; 

For categorical columns, we compute the tvds between the missing group and the non-missing group.

Here is the DataFrame showing our result:

<iframe 
src="table/barons_missingness.html" 
width=800
height=400
frameBorder=0>
</iframe>

From this, we can see that the missingness of ```barons``` is independent from the values of other columns, with 95% confidence level.

### Handling Missingness

* Use ```patch``` to conditionally impute missing ```elders```. We use ```np.random.choice``` to randomly select $$n$$ ```elders``` values from the rows with the same ```patch```, where $$n$$ is the number of ```elders``` missing corresponding to that ```patch``` (we regard na value in ```patch``` as a valid category). Note that for those patches where ```elders``` do not exist, the non-missing values are all 0, and hence the imputed values must be 0.

* Listwise delete rows where ```barons``` is missing (delete all rows where ```barons``` is missing), since ```barons``` is missing completely at random.

## Hypothesis Testing

Question: _Will the choice of side by a team in a game affect the team kills?_

* $$H_0$$: The blue side has the same expected team kills as the red side.

* $$H_1$$: The blue side has higher expected team kills than the red side.

Here is a graph showing the overlaying distributions of red-side kills and blue-side kills:

<iframe 
src="img/distribution_of_kills_by_side.html" 
width=800
height=600
frameBorder=0>
</iframe>

We test the two hypotheses by conducting a permutation test by permuting ```kills``` 1000 times and compute the test statistic _average kills blue - average kills red_.

Below is a graph showing the distribution of the test tatistic according to $$H_0$$ and our observed statistic:

<iframe 
src="img/distribution_of_difference_in_mean_(kills).html" 
width=800
height=600
frameBorder=0>
</iframe>

Since p-value we obtain is 0.0, we reject $$H_0$$. This test makes us conclude that more likely that not, the choosing the blue side does increase the expected kills of the team.

## Framing a Prediction Problem

We try to obtain a model that predicts whether a team is winning/losing a game, given the data we kept.

This is a binary classification problem.

We will use (test set) accuracy to evaluate our model, since 
* It is a more direct metric for the success of the model
* We do not have a preference towards false positive / false negative
* The ```result``` we get after cleaning and imputation has about equal numbers of ```1``` and ```0``` (the percentage of winning is 50.01%)

## Baseline Model

Our baseline model uses logistic regression to predict whether the team will win or lose given the features we select. 

This dataframe shows the first several rows of ```result```, which we are predicting:

<iframe 
src="table/model_y.html" 
width=800 
height=200
frameBorder=0>
</iframe>

Here is a description of the features we use in our baseline model and what transformations we apply. For this model, all features come from the original dataframe.

#### Nominal

We one-hot-encode every categorical column (each time, drop one of the columns generated to avoid colinearity).

This dataframe shows the categorical columns before one-hot-encoding:

<iframe 
src="table/baseline_nominal.html" 
width=800 
height=200
frameBorder=0>
</iframe>

#### Ordinal

_We do not have any ordinal feature in our selection._

#### Numerical

We standardize every numerical column.

That is, for every column $$X=(X_1, X_2, ..., X_n)$$, for every $$i=1, 2, ..., n$$, we let $$X_{std, i}=\frac{X_i-\bar{X}}{SD_X}$$.

This dataframe shows the categorical columns before standardizing:

<iframe 
src="table/baseline_quant.html" 
width=800 
height=200
frameBorder=0>
</iframe>

### Model Parameters

For this model, we use default parameters of sklearn.LogisticRegression.

* Penalty: ```'l2'```
* tol (tolerance for stopping iteration): ```1e-4```
* max_iter: ```100```

### Model Result

The following dataframe shows the model accuracy:

<iframe 
src="table/baseline_model_score.html" 
width=800 
height=200
frameBorder=0>
</iframe>

The following graph is the confusion matrix on the test set:

<iframe 
src="img/confusion_matrix_of_test_data_(baseline_model).html" 
width=800 
height=400
frameBorder=0>
</iframe>
