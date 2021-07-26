---
layout: post
title:  "Data Professionals Survey Analysis"
info: "An analysis project on a raw, public survey dataset."
tech: "Python"
type: Project
---


# Goal
To perform data cleaning and analysis on the data to answer this question: How/Where do data analysts, engineers, and scientists differ? This could be interms of income, responsibilities, educational background, etc. 


# Tools
Python
- Pandas
- Seaborn


# Challenges



# Data Overview
The dataset is available [here](https://data.world/finance/data-professional-salary-survey).
Some notes that the dataset uploader left for us:
- The spreadsheet includes the results for all 3 years. We’ve gradually asked more questions over time, so if a question wasn’t asked in a year, the answers are populated with Not Asked.
- The postal code field was totally optional, and may be wildly unreliable. Folks asked to be able to put in small portions of their zip code, like the leading numbers.

**survey_year**: Year the survey results were released (reflecting on data from the previous year)  
**timestamp**: Date and time of survey completion  
**salaryusd**: User submitted salary in USD  
**country**: Country of residence  
**postalcode**: User submitted postal code  
**primarydatabase**: Primary database used  
**yearswiththisdatabase**: Years using this database  
**otherdatabases**: Other databases used  
**employmentstatus**: Employment status  
**jobtitle**: Job title  
**managestaff**: Whether or not they manage staff  
**yearswiththistypeofjob**: Years working in this type of job  
**howmanycompanies**: How many companies  
**otherpeopleonyourteam**: How many other people are on your team  
**companyemployeesoverall**: How many employees your company has  
**databaseservers**: Database servers  
**education**: Highest level of education  
**educationiscomputerrelated**: Whether education was computer related  
**certifications**: Certifications  
**hoursworkedperweek**: Hours worked per week  
**telecommutedaysperweek**: Telecommute days per week  
**newestversioninproduction**: Newest version in production  
**oldestversioninproduction**: Oldest version in production  
**populationoflargestcitywithin20miles**: Population of largest city within 20 miles  
**employmentsector**: Employment sector  
**lookingforanotherjob**: Looking for another job?  
**careerplansthisyear**: Career plans this year  
**gender**: Gender  
**otherjobduties**: Any other job duties  
**kindsoftasksperformed**: Kinds of tasks performed  
  


## Potential Insights
- Average Salary of each type of data professional.
- Which countries pay data professionals best.
- How much experience affects salary.
- Most popular primary database.
- Average team size for each job type.
- How much educational background affects salary

# Procedure

## Step 1: Importing Data
As the data is directly downloaded as an Excel file, we will use read_excel to read it into a dataframe. The first 3 rows were irrelevant so they were skipped.  

```python
data = pd.read_excel('survey_data.xlsx', skiprows=3)
```

## Step 2: Data Cleaning

### Dropping Irrelevant Columns
Just by reading the data description, we can see that there are certian columns we will not be using. Let's first drop those irrelevant columns!
```python
data.drop(['PostalCode', 'Timestamp', 'Gender', 'Counter'], axis=1, inplace=True)
```

# Removing Duplicate / Irrelevant Rows
There is a useful pandas dataframe method ```duplicated()``` that gives us duplicate rows. In this dataset, there are a number of rows that are just filled with garbage. 

```python
data[data.duplicated()]
```



### Replacing Outliers


## Step 3: EDA



## Step 4: Visualization


## Step 5: Dashboard


# Conclusion
