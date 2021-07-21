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
	title="project-overview" width="800"/>  

 
# Procedure  
## Step 1: Gathering Data
The match and player data is available [here](https://overwatchleague.com/en-us/statslab) for free thanks to the Overwatch League (OWL). We will use the match data as well as player data from the 2020 and 2021 season. The data is loaded into a PostgreDB server using DDL (CREATE TABLE) to create the table and set data column data types then importing the csv. 

<img src="https://i.imgur.com/AIX4kk9.jpg" alt="tables"
	title="postgre-tables" width="150"/>  


## Step 2: Data Overview
The database schema looks like this but with multiple player_data tables for each year. The schema below is simplified as it only shows one player_data table.  


<img src="https://i.imgur.com/CU1oTgA.png" alt="schema"
	title="data-schema" width="700" />


The match data contains highlevel information about the match: Most importantly, which teams were playing and who won because that will be our model's target variable. 
The player data contains individual players' performance in each round of each match. The performance metrics are called stats in the data and are stored in a long format. 





## Step 3: Data Preparation
So, what exactly does our data need to look like in order for it to be usable by our predictive model? We the match id, the two teams in the match, which team won the match, and the players' performance stats in a pandas dataframe. Although we won't be using the match id or team names for training, it will be included to validate the data.  
One of the trickiest parts of this project is how to organize this data into our desired, wide-format data. The first step is to create a list that will serve as the column names for our future dataframe.


### Getting all column names
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

the w_l feature is our target variable. When w_l is 1, team_one won the match. If w_l is 0, team_two won the match. 

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

### Getting a team's average stats
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

### Getting the differnece between average stats of two teams in a match
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
At this point, we have our dataframe ready with our desired data: The difference between the teams' average stats and who won the match. However, the dataframe has 40 columns, and we can try and reduce this amount using data exploration and finding the features that will be most useful in our model and focusing on those. Here, we use a heatmap to show which features are most correlated with the outcome of the match (w/l) based on pearson correlation values. We only show correlations for features that have a pearson correlation value of 0.7 or higher. 


```python
corr = df.corr()
best_corr = corr[abs(corr)>=.7]
plt.figure(figsize=(12,8))
sns.heatmap(best_corr, cmap="Blues")
```


<img src="https://i.imgur.com/qJ2HLUy.jpg" alt="heatmap"
	title="corr-heatmap" width="500" />


We can see that there are only 7 features that have high correlation with the outcome of the match. Therefore, we can drop the other features. The less features the model relies on for predictions, the better. 


## Step 4: Train/Test split

The dataframe that we created in the previous step is what we will derive both training and testing data from. We first need to remove any features that are irrelevant for predicitons (i.e match_id, team_one, team_two). The model only needs to see the difference in average stats and the results. 
The current dataframe containing matches from 2020 includes 256 matches total. We will use 70% of the data in the dataframe for training and 30% for testing. After we create our model, we will import match data from 2021 for validation. The train_test_split function from Sklearn makes this very easy to do. We just need to make sure that x data does not include the target variable and the y data only includes the target variable. 

 
### Getting Training and Testing Data
```python
train_test = df[['w_l', 'time_alive', 'final_blows', 'eliminations', 'defensive_assists', 'deaths', 'average_time_alive', 'assists']]
x_train, x_test, y_train, y_test = train_test_split(train_test.drop('w_l', axis=1), train['w_l'], train_size = 0.7, random_state=1)
```

OWL Data is allocated as such


<img src="https://i.imgur.com/Amoq8sp.jpg" alt="split"
	title="data-allocation" width="500" />


## Step 5: Logistic Regression Model


```python
log_reg = LogisticRegression(solver='lbfgs')
classifier = log_reg.fit(x_train, y_train)
log_reg_test_score = log_reg.score(x_test, y_test)
```






## Step 6: Performance Evaluation

