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
	title="postgre-tables" width="150" height="75" />

## Step 2: Data Exploration
We will use matches from 2020 as training/testing data and use 2021 matches for validation.  

<img src="https://i.imgur.com/CU1oTgA.png" alt="schema"
	title="data-schema" width="700" height="600" />


The match data contains highlevel information about the match: Most importantly, which teams were playing and who won because that will be out target variable. 
The player data contains individual players' performance in each round of each match. The performance metrics are called stats in the data and are stored in a long format. 



