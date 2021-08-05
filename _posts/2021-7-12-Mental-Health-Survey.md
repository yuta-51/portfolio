---
layout: post
title:  "Mental Health Survey Analysis"
info: "An EDA project on raw survey data using Excel."
tech: "MS Excel"
type: Project
thumbnail: https://files.usef.org/assets/f4tsB1ysiAU/healthinsuranceiconmental-health_large.png
---

# Goal 
To clean and gather insights from raw survey data which pertains to employees' mental health in the tech industry. 
Mental health is a serious issue that has been looked over by many until recently. 
It is important to explore survey data to gain insights on what kind of people are more likely to not get help with their mental health issues as well as what companies can do to help their employees. 


# Tools
- MS Excel 


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
As the data is only ~ 1200 rows, using Excel to clean is practical. Each action taken towards cleaning the data will be logged here.

### Removing Outliers 
- Removed rows where the age was negative, < 11,  or > 300. Total of 8 rows removed. 


### Inconsistent Data Types
- The ```no_employees``` column had a mix of strings and dates for some reason. We only want the string so a formula, ```=IF(ISNUMBER(K1241), "Unknown", K1241)```, was used to 
replace the dates with "Unknown". 


### Inconsistent String Format
- The ```gender``` column was a free-response question so the formatting was very inconsistent. To simplify the process, I divided the responses into Male, Female, and Non-Binary. The formula I created is essentially a nested IF statement which checks for "female", "f", "male", and "m" to identify males and females. If the user responded with anything else, it would simply carry over their original response, as it is impossible to include checks for all genders. The response, "woman" and "man" was replaced with female/male to avoid having to include an extra check in the formula using find and replace. Also, it is important to trim the response to remove all extra spaces. 

```
=IF(OR(EXACT("female",TRIM(LOWER(D2))),EXACT("f",TRIM( LOWER(D2)))),"Female",IF(OR(EXACT("male",TRIM(LOWER(D2))),EXACT("m", TRIM(LOWER(D2)))),"Male",TRIM(D2)))
```


The formula used looks long and scary but it is just checking if a formatted version of the original string matches with a one of the search strings. 
Some manual changes were made as well for small typos. This is a sample of the original and cleaned version of the gender column. 


<img src="https://i.imgur.com/Afjkt9B.jpg" alt="gender-column" width="300"> 


## Step 4: Feature Engineering

### ```has_experience```
This column indicates whether the respondent has had any experience with mental health conditions. The value of this column is ```Yes``` if they have been treated for mental health problems or have had their work interfered by their mental health. Otherwise, it is a ```No```. A simple formula which reads values from the ```treatment``` and ```work_interfere``` columns and applies logic to it in an IF statement is made. 


```
=IF( OR(I2="Yes", NOT( OR(J2="NA", J2="Never"))), "Yes", "No")
```


### ```no_treatment_despite_interfere```
Another boolean column which is marked ```Yes``` is the respondent experiences interference at work from mental health conditions but have not sought to get treatment. 


```
=IF(AND(J2="No",NOT(OR(K2="NA",K2="Never"))),"Yes","No")
```


The rest of the columns (other than ```comments```) seem to not be free response, so we don't have to worry as much about cleaning them. 



## Step 4: Insight using Pivot Tables

Most important insights gained from this project: 


**% of Respondents Who Have Experienced Mental Health Issues:**
> 64.83 %



**% of Respondents Who Have Experienced Mental Health Issues and Have a Family History of Mental Health Issues:**
> 86%


Most respondents who have experienced mental health issues have a family history of mental health issues. 


**Does The Respondent's Company Provide Mental Health Benefits?:**
> Yes: 37.81 %  
> No: 29.66 %  
> Don't Know: 32.53 %  


So a third of companies offer mental health benefits. This is definitely an issue, but something else I see is that a third of the people don't even know if they have mental health benefits! This supports the view that not enough people even know about or care about peoples' mental health (including themselves).  


**% of Respondents Who Experience Mental Health Issues Based on How Likely They Are to Talk to Their Coworkers About Their Mental Health:**
> Likely: 9%             
> Somewhat Likely: 14%      
> Unlikely: 19%  


**% of Respondents Who Experience Mental Health Issues Based on How Likely They Are to Talk to Their Supervisors About Their Mental Health:**
> Likely: 11%            
> Somewhat Likely: 15%        
> Unlikely: 17%  


The employees' willingless to talk about their mental health with coworkers/supervisors strongly affects the chance of them experiencing mental heal issues (More than double the percentage when comapring likely to unlikely).


**% Of People Who Would Talk About their Mental Health Issues with an Employer in an Interview:**	
> 3%


Almost nobody would talk to their employer about their mental health most likley because they see it as a disadvantage and something that would be frowned upon.



**% Of People Who Don't Get their Mental Health Issues Checked Based on Whether or Not Company Respects their Anonymity when Offering Mental Help:**
> Respects their anonymity when assisting them:	9%    
> Does not respect their anonymity when assisting them:	20%    


More than double the percentage of people do not get help with their mental health when the company resources do not respect their employees' anonymity.


# Recommendations 

- **Create an environment where people can talk about their mental health openly**. More people need to know about it and respect it. We saw that employee being able to talk about their mental health with their coworkers/supervisors helps tremendously. When the entire company knows about and can understand the seriousness of mental health issues, it will create a more inclusive and comfortable environment, allowing more people to speak out on their mental health issues.
- **Fix the stigma around mental health through educating employees through a wellness program**. Only 3% of respondents said that they would talk about their mental health issues with an employer. There is clearly a stigma around mental health which makes it harder for people to talk about their issues openly.
- **Offer help while respecting employees' anonymity***. Just talking and knowing about mental health helps, but at some point people will need profressional help. Companies should offer this profressional help to all employees and also respect their anonymity while helping them. We saw that people are twice more likley to not get their mental health checked when their company does not respect their anonymity. 








