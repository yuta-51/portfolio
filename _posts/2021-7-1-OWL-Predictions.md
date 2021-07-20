---
layout: post
title:  "Predicting Outcome of E-sports Matches"
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
 
# Procedure  
## Step 1: Gathering Data
The match and player data is available [here](https://overwatchleague.com/en-us/statslab) for free thanks to the Overwatch League (OWL).  We will use the match data as well as player data from the 2020 and 2021 season. The data is loaded into a PostgreDB server using DDL (CREATE TABLE) to create the table and set data column data types then importing the csv. 

<img src="https://i.imgur.com/AIX4kk9.jpg" alt="tables"
	title="postgre-tables" width="150" height="75" align="left"/>

## Step 2: Data Exploration
The database schema looks like this but with multiple player_data tables for each year. The schema below is simplified as it only shows one player_data table.  


<img src="https://i.imgur.com/CU1oTgA.png" alt="schema"
	title="data-schema" height="500" />


The match data contains highlevel information about the match: Most importantly, which teams were playing and who won because that will be our model's target variable. 
The player data contains individual players' performance in each round of each match. The performance metrics are called stats in the data and are stored in a long format. 





## Step 3: Data Manipulation
So, what exactly does our data need to look like in order for it to be usable by our predictive model? We need our data to be in wide format, where we have the match id, both team names, which team won the match, and the players' performance stats. 

```python
def find_winner(match_id):
    cur.execute(f"""
    SELECT match_winner
    FROM public.match_data
    WHERE match_id = {match_id}
    LIMIT 1
    """)
    return cur.fetchall()[0][0]

# Returns a tuple of the two teams that plyaed in the match. Order is random
def find_teams(match_id):
    cur.execute(f"""
    SELECT team_one_name, team_two_name
    FROM public.match_data
    WHERE match_id = {match_id}
    LIMIT 1
    """)
    return cur.fetchall()[0]

# Returns a list of 36 average "all heroes" stats for a specific team during a specific match
def find_avg_stats(match_id, team, yr):
    cur.execute(f"""
    SELECT DISTINCT stat_name, AVG(stat_amount)
    FROM public.player_data_{yr}
    WHERE 	hero_name= 'All Heroes' AND  
            esports_match_id = {match_id} AND 
            team_name = '{team}'
    GROUP BY stat_name
    ORDER BY stat_name
    """)
    stats_dict = dict.fromkeys(all_hero_stats, None)
    for stat_name, stat_avg in cur.fetchall():
        if stat_name in stats_dict.keys():
            stats_dict[stat_name] = stat_avg
    
    return stats_dict.values()

# Finds the difference in the team's average stats and returns as a list. 
def find_avg_stats_diff(match_id, team_1, team_2, yr):
    zipped = zip(find_avg_stats(match_id, team_1, yr), find_avg_stats(match_id, team_2, yr))
    avg_stats_diff = []
    for stat_team_1, stat_team_2 in zipped:
        if stat_team_1 and stat_team_2:
            avg_stats_diff.append(round(stat_team_1 - stat_team_2, 2))
        else:
            avg_stats_diff.append('Null')
    return avg_stats_diff
```





## Step 3: Data Exploration











<img src="https://i.imgur.com/XjwCwAh.jpg" alt="split"
	title="data-allocation" height="350" />
