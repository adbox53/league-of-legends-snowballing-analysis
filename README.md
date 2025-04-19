# Can Early Leads Guarantee Wins? A Snowballing Analysis in League of Legends

## Introduction

**League of Legends (LoL)** is one of the most popular competitive games in the world, boasting millions of players and a massive professional esports scene. In a standard match, two teams of five players each compete to destroy the opposing team’s base. Over the course of a game, teams earn gold and experience by defeating enemies and securing map objectives—advantages that allow them to purchase powerful items and gain momentum.

A key strategic concept in League is the **snowballing effect**—the idea that an early lead can rapidly spiral into a larger, often insurmountable advantage as the game progresses. Once a team gets ahead in gold, experience, or objectives, they can more easily secure future advantages, often leading to victory.

This project investigates the following central question:

> **To what extent do early-game leads predict a team’s advantage later in the match—and ultimately, their chance of winning?**

Understanding this relationship is important not just for players and coaches, but also for game designers and analysts seeking to maintain competitive balance and gain deeper insights into professional-level play.

### About the Dataset

The dataset used in this project consists of **117,600 rows** and **161 columns**, with each row representing one team or player’s performance in a 2024 professional match, and each column or "feature" representing a certain aspect of the game. To explore how advantages accumulate over time, the project focuses on a subset of features that capture key metrics at different game intervals.

### Relevant Features

#### At 10 Minutes
- `deathsat10`: Total number of deaths for the team by the 10-minute mark (quantitative)  
- `firstdragon`: Binary indicator for whether the team secured the first dragon (nominal: 0 or 1)  
- `golddiffat10`: Total team gold differential at the 10-minute mark (quantitative)  
- `csdiffat10`: Total team creep score (CS) differential at 10 minutes (quantitative)  
- `killsat10`: Total number of team kills by 10 minutes (quantitative)  

#### At 15 Minutes
- `golddiffat15`: Total team gold differential at the 15-minute mark (quantitative)  
- `csdiffat15`: Total team creep score (CS) differential at 15 minutes (quantitative)  
- `killsat15`: Total number of team kills by 15 minutes (quantitative)  
- `deathsat15`: Total number of team deaths by 15 minutes (quantitative)  

#### At 20 Minutes
- `golddiffat20`: Total team gold differential at the 20-minute mark (quantitative)  
- `csdiffat20`: Total team creep score (CS) differential at 20 minutes (quantitative)  
- `killsat20`: Total number of team kills by 20 minutes (quantitative)  
- `deathsat20`: Total number of team deaths by 20 minutes (quantitative)  

#### Post-20 Minutes
- `firstbaron`: Binary indicator for whether the team secured the first Baron Nashor (nominal: 0 or 1)  

By analyzing how these early and mid-game indicators relate to key late-game outcomes, the project aims to illustrate how early leads can snowball into larger advantages and ultimately drive a team to victory.

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

 The primary focus was on high-level competitive gameplay, specifically Tier 1 professional leagues and the World Championship (Worlds). The goal was to filter and clean the data in a way that would enable meaningful exploration of how early-game advantages in League of Legends can snowball into later-game success. Below are the main data cleaning steps:


1. **Filtering for Complete Games**  
   The first step was to filter out rows where the `datacompleteness` column indicated incomplete games.

    ```python
    LOL24_clean = LOL24[LOL24['datacompleteness'] == 'complete']
    ```

2. **Selecting One Row Per Team Per Game**  
   Since each match can contain multiple rows corresponding to individual players, only rows where the individual `playername` was missing (i.e., rows representing whole teams) were included in the final dataset.

    ```python
    LOL24_team = LOL24_clean[LOL24_clean['playername'].isna()]
    ```

3. **Filtering for Major Leagues and Worlds Events**  
   To ensure the dataset focused on high-level competitive matches, the data was filtered to include only matches from major leagues such as LCK, LPL, LEC, LTA, LCP, and Worlds events.

    ```python
    LOL24_team_major = LOL24_team[LOL24_team['league'].isin(['LCK', 'LPL', 'LEC', 'LTA', 'LCP', 'Worlds'])]
    LOL24_team_major.shape
    ```

    The shape of the dataset after this step was `(1790, 161)`.

### Time-Based Feature Extraction

To analyze how early-game performance affects the outcome of the match, the data was divided into key time intervals at 10, 15, 20 and post-20 minutes. These time intervals are critical for understanding how early leads might snowball into larger advantages later in the game.

1. **Defining Baseline Features**  
   The initial set of features was defined to capture key aspects of gameplay at the 10-minute mark.

    ```python
    baseline_features = ['golddiffat10', 'csdiffat10', 'killsat10']
    baseline_data = LOL24_team_major[baseline_features + ['result']]
    ```

2. **Extracting Features at 10, 15, and 20 Minutes**  
   Next, features were extracted for the 10, 15, 20, and post 20 minute time intervals.

    ```python
    features_10 = ['deathsat10', 'firstdragon']  # 10 min features
    features_15 = ['golddiffat15', 'csdiffat15', 'killsat15', 'deathsat15']  # 15 min features
    features_20 = ['golddiffat20', 'csdiffat20', 'killsat20', 'deathsat20']  # 20 min features
    features_baron = ['firstbaron']  # Post-20 min features
    ```

3. **Combining Features into Datasets**  
   The final step involved combining the baseline features with the features extracted at the various time intervals to form datasets for analysis.

    ```python
    baseline_data = LOL24_team_major[baseline_features + ['result']]  # Base
    data_10 = LOL24_team_major[baseline_features + features_10 + ['result']]  # Base + 10 min features
    data_15 = LOL24_team_major[baseline_features + features_10 + features_15 + ['result']]  # Base + 10 + 15 min features
    data_20 = LOL24_team_major[baseline_features + features_10 + features_15 + features_20 + ['result']]  # Base + 10 + 15 + 20 min features
    data_first_baron = LOL24_team_major[baseline_features + features_10 + features_15 + features_20 + features_baron + ['result']]  # Base + 10 + 15 + 20 + Baron
    ```

    The shapes of the prepareddatasets used in the analysis were as follows:

    - `baseline_data.shape`: (1790, 4)  
    - `data_10.shape`: (1790, 6)  
    - `data_15.shape`: (1790, 10)  
    - `data_20.shape`: (1790, 14)  
    - `data_first_baron.shape`: (1790, 15)


    Below is a preview of `data_first_baron`, the most complete form of the final dataset used in this project. It includes early- and mid-game features as well as the first Baron objective:

    **Early-game stats**

    | golddiffat10 | csdiffat10 | killsat10 | deathsat10 | firstdragon | golddiffat15 | csdiffat15 | killsat15 | deathsat15 |
    |--------------|------------|-----------|------------|-------------|---------------|-------------|------------|-------------|
    | 882          | -3         | 5         | 4          | 0           | 1729          | -5          | 6          | 5           |
    | -882         | 3          | 4         | 5          | 1           | -1729         | 5           | 5          | 6           |
    | -883         | 5          | 0         | 2          | 1           | -331          | 28          | 1          | 3           |
    | 883          | -5         | 2         | 0          | 0           | 331           | -28         | 3          | 1           |
    | -392         | 19         | 0         | 1          | 1           | -461          | 14          | 1          | 2           |

    **Mid- to late-game stats and result**

    | golddiffat20 | csdiffat20 | killsat20 | deathsat20 | firstbaron | result |
    |--------------|-------------|------------|-------------|-------------|--------|
    | 2416         | 7           | 6          | 5           | 0           | 0      |
    | -2416        | -7          | 5          | 6           | 1           | 1      |
    | -786         | 11          | 3          | 5           | 0           | 0      |
    | 786          | -11         | 5          | 3           | 1           | 1      |
    | 179          | 20          | 6          | 5           | 1           | 0      |




### Exploratory Data Analysis

#### Univariate Analysis

##### Gold Difference Over Time by Match Result

<iframe  
src="assets/gold_difference_plot.html"  
width="900"  
height="530"  
frameborder="0"  
></iframe>  

This plot illustrates how winning teams tend to hold a small gold advantage at the 10-minute mark, which progressively increases by 15 and 20 minutes. This trend highlights the snowballing effect: teams that gain an early lead often maintain and build upon it as the game progresses.

---

##### Win Rate by First Blood

<iframe  
src="assets/first_blood_plot.html"  
width="550"  
height="530"  
frameborder="0"  
></iframe>  
This plot shows that teams securing first blood exhibit a significantly higher win rate. Gaining an early kill can grant tempo and map control, often translating into a gold lead and reinforcing the snowballing effect observed in professional League of Legends games.

#### Bivariate Analysis

##### KDE: Gold Diff at 10 vs. Kills at 20

<iframe  
src="assets/kill20_vs_gold10_plot.html"  
width="820"  
height="630"  
frameborder="0"  
></iframe>  

This plot highlights how a gold lead at 10 minutes leads to more kills at 20 minutes. As the game progresses, the impact of an early lead becomes even more significant, with the gap in kills widening. This further reinforces the idea that early advantages often snowball into greater success later in the game.

---

#### Interesting Aggregates

##### Win Rate Based on First Dragon and First Baron

<iframe  
src="assets/baron_dragon_plot.html"  
width="600"  
height="430"  
frameborder="0"  
></iframe>  

This heatmap displays win rates based on whether teams secured the first Dragon and/or the first Baron. When **neither objective** was secured, the win rate was just **13%**. Securing **only the first Dragon** increased the win rate to **23%**, while **only the first Baron** resulted in an impressive **80%** win rate. Teams that secured **both objectives** surged to a **90%** win rate. These results strongly emphasize the snowballing nature of League of Legends—particularly how securing pivotal neutral objectives like Baron can decisively shift the momentum in a team's favor.

#### Imputation

After filtering and cleaning the dataset to obtain `data_first_baron` (the most complete version of the dataset), a check for missing values was performed using:

```python
data_first_baron.isna().sum()
```

This returned the following result:

| Column        | Missing Values |
|:--------------|---------------:|
| golddiffat10  |              0 |
| csdiffat10    |              0 |
| killsat10     |              0 |
| deathsat10    |              0 |
| firstdragon   |              0 |
| golddiffat15  |              0 |
| csdiffat15    |              0 |
| killsat15     |              0 |
| deathsat15    |              0 |
| golddiffat20  |              0 |
| csdiffat20    |              0 |
| killsat20     |              0 |
| deathsat20    |              0 |
| firstbaron    |              0 |
| result        |              0 |

Since there were no missing values in any of the columns, **no imputation was necessary**.




---

## Framing a Prediction Problem

### Objective:
**Predict the outcome of a League of Legends match (Win or Loss) based on early game metrics captured at 10, 15, 20, and post-20 minute intervals**. 

This prediction problem explores how early-game advantages — such as gold difference, CS (creep score) difference, kills, deaths, and key objectives (like securing the first dragon or baron) — contribute to a team’s likelihood of winning. This ties into the broader theme of **snowballing** in League of Legends, where small early leads can lead to significantly better outcomes later in the match.

### Problem Type:
This is a **binary classification** problem. The goal is to predict a categorical response variable with two possible outcomes:  
- **1 = Win**  
- **0 = Loss**

All predictive models used in this project will be **logistic regression classifiers**.

### Response Variable:
- `result`: a binary variable indicating whether the team **win (1)** or **loss (0)** the match.

### Features and Time of Prediction:
All features used in modeling are available at the **time of prediction**. For example, to predict match outcome based on 10-minute metrics, only features collected by the 10-minute mark are used (e.g., `golddiffat10`, `killsat10`, `firstdragon`). The result of the match is unknown to all of the models.

### Evaluation Metrics:
To evaluate model performance, the following metrics were used:
- **Accuracy**: Measures the proportion of correct predictions.
- **Precision**: Indicates how many predicted wins were actual wins.
- **Recall**: Measures how many actual wins were correctly predicted.

> Note **F1-score was not considered** because the class distribution was approximately balanced, making accuracy, precision, and recall sufficient for meaningful evaluation.

#### Class Distribution in Final Dataset:

| Result | Proportion |
|--------|------------|
| 0 (Loss) | 0.50 |
| 1 (Win)  | 0.50 |


---

## Baseline Model

### Model:
The baseline model uses a **logistic regression classifier**, trained on early-game metrics available at the **10-minute mark**. 

### Features:
The baseline model uses the following features:

- `golddiffat10`: Total team gold differential at the 10-minute mark (quantitative)  
- `csdiffat10`: Total team creep score (CS) differential at 10 minutes (quantitative)  
- `killsat10`: Total number of team kills by 10 minutes (quantitative)  

All three features are **quantitative**, so no encoding was necessary.

### Performance:

- **Accuracy**: *0.63967*
- **Precision**: *0.61364*
- **Recall**: *0.63905*

### Confusion Matrix:
To further assess performance, a confusion matrix is included below:

<iframe  
src="assets/baseline_model_confusion.html"  
width="820"  
height="630"  
frameborder="0"  
></iframe>

### Evaluation:
This baseline model provides a reasonable starting point for understanding how early-game performance influences match outcomes. While it is limited to three features, its simplicity offers a clear foundation for evaluating improvements in future models.

---


## Final Model

The goal of this project was to capture the snowballing effect that occurs in League of Legends matches, particularly how early leads influence the outcome of the game. To do this, a range of features were added at each significant time interval (10, 15, 20, and post-20 minutes). As the time intervals extend, the model’s performance should improve, reflecting the increasing impact of early advantages as the game progresses. The idea is that the longer the game goes on, the more pronounced the snowballing effect becomes, making the gold difference, kills, and other early metrics more predictive of the match outcome.

### Features Added

The features were grouped based on different time intervals (10, 15, and 20 minutes) and were selected to capture important game events as they occur during the match. Below are the features added and their rationale:

#### 10-Minute Features (Dataset: `data_10`)
- 'deathsat10'
- 'firstdragon'

These early game features can influence the snowballing effect, particularly in terms of early deaths (causes a team to fall behind) and securing the first dragon (first major objective).

#### 15-Minute Features (Dataset: `data_15`)
- 'golddiffat15'
- 'csdiffat15'
- 'killsat15'
- 'deathsat15'

At this point in the game, the influence of early game decisions and mechanics starts to compound. These features provide a more complete picture of a team's progress through the first 15 minutes, including gold and kill differences as well as continued deaths.

#### 20-Minute Features (Dataset: `data_20`)
- 'golddiffat20'
- 'csdiffat20'
- 'killsat20'
- 'deathsat20'

By 20 minutes, teams have generally established a stronger advantage or disadvantage. These features help capture that shift and assess how early advantages continue to affect the outcome.

#### First Baron Feature (Dataset: `data_first_baron`)
- 'firstbaron'

Securing the first Baron is one of the most significant advantages a team can gain during a match, as seen in the "Intersting Aggregates" section. This feature helps capture a pivotal moment in the game, which can drastically influence the match's flow and eventual outcome.

#### Overall...
These features are still highly relevant, as the mean game length in the dataset was approximately **32.45 minutes**. With this in mind, analyzing data at 10, 15 and even 20 minutes and including the first Baron (which spawns in at 20 minutes) are all relevant to consider for the overall match outcome, while not being entirely deterministic.

```python
    LOL24_team_major['gamelength'].mean()/60 = 32.4478
```

### Modeling Algorithm and Hyperparameter Tuning

For the modeling algorithm, Logistic Regression was used for all models. Hyperparameter tuning was performed using GridSearchCV to find the optimal combination of hyperparameters. The grid search considered the following parameters:

- **C**: Smaller values of C lead to stronger regularization, while larger values mean weaker regularization.
- **Penalty**: Both 'l1' and 'l2' penalties were tested to see which performed best.
- **Solver**: Different solvers work better with specific penalties. 'liblinear' is suitable for smaller datasets, while 'saga' works well for larger datasets.

The final best parameters for the `data_first_baron` dataset (representing the final model) were:

| Parameter | Value  |
|-----------|--------|
| C         | 10     |
| Penalty   | l1     |
| Solver    | liblinear |


### Final Model Performance vs. Baseline Model Performance

The performance of the final model (`data_first_baron`) is a significant improvement over the baseline model. The baseline model only included features at 10 minutes, which, while useful, do not account for the compounding effects of early leads. The final model, with features at 10, 15, and 20 minutes, as well as the first Baron, provides a more comprehensive view of the game’s progression, leading to improved predictive performance.

Below, we can see the performance of the final model, as well as several intermediary models that perfectly capture the snowballing nature of League of Legends.

1. **Model Performance (Recall, Precision, and Accuracy Across Models)**  
   <iframe src="assets/baron_dragon_perf.html" width="920" height="630" frameborder="0"></iframe>

   This graph shows a steady upward trend in model performance from baseline to 10, 15, and 20 minutes, highlighting how early-game advantages compound over time. The most dramatic jumps occur at the 20-minute mark and after the first Baron is captured. These results reflect the snowballing nature of League of Legends and by the time Baron is slain, the game is often already decided.

2. **Confusion Matrix for the Final Model**  
   <iframe src="assets/data_first_baron_confusion.html" width="820" height="630" frameborder="0"></iframe>

   This graph shows the confusion matrix corresponding to the model trained on the `data_first_baron` dataset. When commparing this to the baseline model confusion matrix, it is evident that the final model demonstrates significantly improved performance.


This analysis confirms the **snowballing effect** in professional League of Legends: teams that secure an early lead are more likely to maintain their advantage, rapidly compounding into game-deciding momentum.