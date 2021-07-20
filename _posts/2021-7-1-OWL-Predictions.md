---
layout: post
title:  "Binary Classification: Predicting Outcome of E-sports Matches"
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
 The match and player data is available [here](https://overwatchleague.com/en-us/statslab) for free thanks to the Overwatch League (OWL).  
 The data is structured like this:  
 
| match_data                                    |   player_data                     |                                
| :---                                          |   :---                            |   
| round_start: timestamp                        |   start_time: timestamp           |           
| round_end: timestamp                          |   esports_match_id: bigint        |    
| stage: text                                   |   tournament_title: text          | 
| match_id: bigint                              |   map_type: text                  |        
| game_number: integer                          |   player_name: text               |   
| match_winner: text                            |   team_name: text                 | 
| map_winner: text                              |   stat_name: text                 | 
| map_loser: text                               |   hero_name: text                 | 
| map_name: text                                |   stat_amount: double precision   |   
| map_round: integer                            |                                   |          
| winning_team_final_map_score: integer         |                                   | 
| losing_team_final_map_score: integer          |                                   | 
| control_round_name: text                      |                                   |    
| attacker: text                                |                                   |          
| defender: text                                |                                   |           
| team_one_name: text                           |                                   |    
| team_two_name: text                           |                                   |           
| attacker_payload_distance: double precision   |                                   |  
| defender_payload_distance: double precision   |                                   |          
| attacker_time_banked: double precision        |                                   |          
| defender_time_banked: double precision        |                                   |    
| attacker_control_percent: integer             |                                   |           
| defender_control_percent: integer             |                                   |           
| attacker_round_end_score: integer             |                                   |           
| defender_round_end_score: integer             |                                   |    


The match data contains highlevel information about the match: Most importantly, which teams were playing and who won because that will be out target variable. 
The player data contains individual players' performance in each round of each match. The performance metrics are called stats in the data and are stored in a long format. 
We will use matches from 2020 as training/testing data and use 2021 matches for validation.  
The data is loaded into a PostgreSQL server using DDL statement CREATE TABLE and will be queried out for further use using Python and SQL. 


## Step 2: Data Exploration




