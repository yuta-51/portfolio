---
layout: post
title:  "Mental Health in Tech Survey Analysis"
info: "An EDA project on raw survey data using Excel."
tech: "MS Excel"
type: Project
thumbnail: https://files.usef.org/assets/f4tsB1ysiAU/healthinsuranceiconmental-health_large.png
---

# Goal 
To clean and gather insights from raw survey data which pertains to employees' mental health in the tech industry. 
Mental health is a serious issue that has been looked over by many until recently. 
It is important to explore survey data to gain insights on what kind of people are more likely to suffer from mental health issues as well as how we can help them. 


# Tools
- MS Excel 
- Tableau 


# Procedure

## Step 1: Import Data
The data can be downloaded [here](https://www.kaggle.com/osmi/mental-health-in-tech-survey). It is publically availbale on Kaggle. 


## Step 2: Understand the Data
The data can be hard to understand at first because each question on the survey has to be represented by a short column name. The details about each question are provided on Kaggle where we originally downloaded the data. 
Some questions that caught my attention are:
- **Country**: This is an international survey. The results could vary between countries, as each country has their own way on dealing with mental health. Some may be better than others. 
- **no_employees**: The number of employees at the person's company. Does the size of the company affect mental health? 
- **benefits**: Wether or not the company offers mental health benefits. Companies can provide benefits, but it is a problem if they don't actually work. We will see if people who work at companies with mental health benefits are actually doing better than those who work at companies that don't. 


Overall, there are many questions that ask about how mental health is viewed/dealt with at the person's company. How do their coworkers/boss view mental health? Do they take it seriously? Does the company try and help people with mental health issues?
Next we will clean the data so we can explore the data and answer our questions.

## Step 3: Cleaning
As the data is only ~ 1200 rows, using Excel to clean is practical. 
