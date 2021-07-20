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
  - psycopg2
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
	title="project-overview" height="250"/>  

 
# Procedure  
## Step 1: Gathering Data
The match and player data is available [here](https://overwatchleague.com/en-us/statslab) for free thanks to the Overwatch League (OWL). We will use the match data as well as player data from the 2020 and 2021 season. The data is loaded into a PostgreDB server using DDL (CREATE TABLE) to create the table and set data column data types then importing the csv. 

<img src="https://i.imgur.com/AIX4kk9.jpg" alt="tables"
	title="postgre-tables" width="150" height="75"/>  


## Step 2: Data Overview
The database schema looks like this but with multiple player_data tables for each year. The schema below is simplified as it only shows one player_data table.  


<img src="https://i.imgur.com/CU1oTgA.png" alt="schema"
	title="data-schema" height="500" />


The match data contains highlevel information about the match: Most importantly, which teams were playing and who won because that will be our model's target variable. 
The player data contains individual players' performance in each round of each match. The performance metrics are called stats in the data and are stored in a long format. 





## Step 3: Data Exploration/Manipulation
So, what exactly does our data need to look like in order for it to be usable by our predictive model? We the match id, the two teams in the match, which team won the match, and the players' performance stats in a pandas dataframe. Although we won't be using the match id or team names for training, it will be included to validate the data.  
One of the trickiest parts of this project is how to organize this data into our desired, wide-format data. The first step is to create a list that will serve as the column names for our future dataframe.


### Getting all column names for the dataframe
```python
cur.execute("""
SELECT DISTINCT stat_name
FROM public.player_data_2020
WHERE hero_name = 'All Heroes' """)
all_hero_stats = [stat[0] for stat in cur.fetchall()]
all_hero_stats= sorted(all_hero_stats)
cols = ['match_id', 'team_one', 'team_two', 'w_l']
cols.extend(all_hero_stats)
```


### Getting all match IDs in 2020 
```python
cur.execute("""
SELECT DISTINCT esports_match_id
FROM public.player_data_2020""")

all_match_ids = [x[0] for x in cur.fetchall()]
all_match_ids = sorted(all_match_ids)
```


### Getting team name's associated with a given match ID
```python
# Returns a tuple of the two teams that plyaed in the match. Order is random
def find_teams(match_id):
    cur.execute(f"""
    SELECT team_one_name, team_two_name
    FROM public.match_data
    WHERE match_id = {match_id}
    LIMIT 1
    """)
    return cur.fetchall()[0]
```


### Getting the winning team of a match
```python
# Returns the winning team of a match
def find_winner(match_id):
    cur.execute(f"""
    SELECT match_winner
    FROM public.match_data
    WHERE match_id = {match_id}
    LIMIT 1
    """)
    return cur.fetchall()[0][0]
```


Now we have the functions/queries necessary to fill three of the columns in the dataframe. Next, we will deal with the player stats data. The player_data table, as mentioned earlier, stores the player stats in long format. We will fill in each stats with the average of the team's players' stats. The following function takes in the match ID, team name, and year of the match and returns a list of the average of the 36 player stats during that match. 

```python
def find_avg_stats(match_id, team, yr):
    cur.execute(f"""
    SELECT DISTINCT stat_name, AVG(stat_amount)
    FROM public.player_data_{yr}
    WHERE   hero_name= 'All Heroes' AND  
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
```
Finally, we will take the difference between the average stats of both teams to get the differnece of each average stat. If the stat does not appear in the data, we will treat it as "null" in our dataframe. 

```python
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


Now we have the functions & queries necessary to completely fill in our dataframe which will become our training and testing data. 

## Step 3: Data Exploration











<img src="https://i.imgur.com/XjwCwAh.jpg" alt="split"
	title="data-allocation" height="350" />