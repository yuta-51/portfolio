---
layout: post
title:  "Predicting the Outcome of E-sports Matches Based on Player Performance"
info: "A binary classification machine learning project."
tech: "Python, PostgreSQL, Sklearn"
type: Project
---

# Goal
The goal of this project is to create a model that can predict which team will win a match based on player performance and match data provided by the [Overwatch League](https://overwatchleague.com/en-us/statslab): 
the e-sports league for professional Overwatch players. 

# Tools
- PostgreDB/SQL
- Python
  - Pandas
  - Scikit Learn Models
    - Logistic Regression
    - Random Forest
    - Decision Tree
  - Seaborn
    - Confusion Matrix
    - Heatmaps  
 
 
 # Project Overview
 <img src="https://i.imgur.com/gVAVuL1.jpg" alt="overview"
	title="project-overview" height="300"/>  

 
# Procedure  
## Step 1: Gathering Data
The match and player data is available [here](https://overwatchleague.com/en-us/statslab) for free thanks to the Overwatch League (OWL). We will use the match data as well as player data from the 2020 and 2021 season. The data is loaded into a PostgreDB server using DDL (CREATE TABLE) to create the table and set data column data types then importing the csv. 

<img src="https://i.imgur.com/AIX4kk9.jpg" alt="tables"
	title="postgre-tables" width="150" height="75" align="left"/>  


## Step 2: Data Overview
The database schema looks like this but with multiple player_data tables for each year. The schema below is simplified as it only shows one player_data table.  


<img src="https://i.imgur.com/CU1oTgA.png" alt="schema"
	title="data-schema" height="500" />


The match data contains highlevel information about the match: Most importantly, which teams were playing and who won because that will be our model's target variable. 
The player data contains individual players' performance in each round of each match. The performance metrics are called stats in the data and are stored in a long format. 





## Step 3: Data Exploration/Manipulation
So, what exactly does our data need to look like in order for it to be usable by our predictive model? We the match id, the two teams in the match, which team won the match, and the players' performance stats in a pandas dataframe. Although we won't be using the match id or team names for training, it will be included to validate the data.  
One of the trickiest parts of this project is how to organize this data into our desired, wide-format data. The first step is to create a list that will serve as the column names for our future dataframe. 

```python
cur.execute("""
SELECT DISTINCT stat_name
FROM public.player_data_2020
WHERE hero_name = 'All Heroes' """)

all_hero_stats = [stat[0] for stat in cur.fetchall()]
all_hero_stats= sorted(all_hero_stats)
cols = ['match_id', 'team_one', 'team_two', 'w_l']
cols.extend(all_hero_stats)
print(len(cols))
```





## Step 3: Data Exploration











<img src="https://i.imgur.com/XjwCwAh.jpg" alt="split"
	title="data-allocation" height="350" />
