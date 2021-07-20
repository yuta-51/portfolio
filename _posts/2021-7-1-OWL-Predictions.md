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
 The match and player data is available [here](https://overwatchleague.com/en-us/statslab) for free thanks to the Overwatch League.  
 The data is structure like this:  
 
| match_data                       |                                                       
| -----------                      |                                 
| round_start: timestamp           |             
| round_end: timestamp             |              
| stage: text                      |  
| match_id: bigint                 |      
| game_number: integer             |          
| match_winner: text               |          
| map_winner: text                 |      
| map_loser: text                  |      
| map_name: text                   |      
| map_round: integer               |          
| winning_team_final_map_score: integer | 
| losing_team_final_map_score: integer  | 
| control_round_name: text         |              
| attacker: text                   |      
| defender: text                   |      
| team_one_name: text              |          
| team_two_name: text              |          
| attacker_payload_distance: double precision |                        
| defender_payload_distance: double precision |                    
| attacker_time_banked: double precision      |                   
| defender_time_banked: double precision      |                      
| attacker_control_percent: integer|                      
| defender_control_percent: integer|                      
| attacker_round_end_score: integer|                      
| defender_round_end_score: integer| 



