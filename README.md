# League of Legends Snowballing Analysis

## Introduction

**League of Legends (LoL)** is one of the most popular competitive games in the world, boasting millions of players and a massive professional esports scene. In a standard match, two teams of five players each compete to destroy the opposing team’s base. Over the course of a game, teams earn gold and experience by defeating enemies and securing map objectives—advantages that allow them to purchase powerful items and gain momentum.

A key strategic concept in League is the **snowballing effect**—the idea that an early lead can rapidly spiral into a larger, often insurmountable advantage as the game progresses. Once a team gets ahead in gold, experience, or objectives, they can more easily secure future advantages, often leading to victory.

This project investigates the following central question:

> **To what extent do early-game leads predict a team’s advantage later in the match—and ultimately, their chance of winning?**

Understanding this relationship is important not just for players and coaches, but also for game designers and analysts seeking to maintain competitive balance and gain deeper insights into professional-level play.

### About the Dataset

The dataset used in this project consists of **117,600 rows** and **161 columns**, with each row representing one team or player’s performance in a professional match. To explore how advantages accumulate over time, the project focuses on a subset of features that capture key metrics at different game intervals.

### Relevant Features

#### At 10 Minutes
- **deathsat10**: Number of deaths by the 10-minute mark  
- **firstdragon**: Binary indicator (1 or 0) for whether the team secured the first dragon  
- **golddiffat10**: Gold differential at the 10-minute mark  
- **csdiffat10**: Creep score differential at the 10-minute mark  
- **killsat10**: Number of kills by the 10-minute mark  

#### At 15 Minutes
- **golddiffat15**: Gold differential at the 15-minute mark  
- **csdiffat15**: Creep score differential at the 15-minute mark  
- **killsat15**: Number of kills by the 15-minute mark  
- **deathsat15**: Number of deaths by the 15-minute mark  

#### At 20 Minutes
- **golddiffat20**: Gold differential at the 20-minute mark  
- **csdiffat20**: Creep score differential at the 20-minute mark  
- **killsat20**: Number of kills by the 20-minute mark  
- **deathsat20**: Number of deaths by the 20-minute mark  

#### Post-20 Minutes
- **firstbaron**: Binary indicator (1 or 0) for whether the team secured the first Baron Nashor  

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
   Since each match can contain multiple rows corresponding to individual players, we needed to focus on one row per team per game. This was done by filtering out rows where `playername` was not missing (i.e., retaining rows with missing player names that correspond to teams rather than individual players).

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

**This plot illustrates how winning teams tend to hold a small gold advantage at the 10-minute mark, which progressively increases by 15 and 20 minutes. This trend highlights the snowballing effect: teams that gain an early lead often maintain and build upon it as the game progresses.**

---

##### Win Rate by First Blood

<iframe  
src="assets/first_blood_plot.html"  
width="550"  
height="530"  
frameborder="0"  
></iframe>  
**This plot shows that teams securing first blood exhibit a significantly higher win rate. Gaining an early kill can grant tempo and map control, often translating into a gold lead and reinforcing the snowballing effect observed in professional League of Legends games.**

#### Bivariate Analysis

##### KDE: Gold Diff at 10 vs. Kills at 20

<iframe  
src="assets/kill20_vs_gold10_plot.html"  
width="820"  
height="630"  
frameborder="0"  
></iframe>  

**This graph highlights how a gold lead at 10 minutes leads to more kills at 20 minutes. As the game progresses, the impact of an early lead becomes even more significant, with the gap in kills widening. This further reinforces the idea that early advantages often snowball into greater success later in the game.**

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




---

## Framing a Prediction Problem

<!-- Content coming soon -->

---

## Baseline Model

<!-- Content coming soon -->

---

## Final Model

<!-- Content coming soon -->