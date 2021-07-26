---
layout: post
title:  "Data Professionals Survey Analysis"
info: "An analysis project on a raw, public survey dataset."
tech: "Python"
type: Project
---


# Goal
To perform data cleaning and analysis on the data to answer this question: How/Where do data analysts, engineers, and scientists differ? This could be interms of income, responsibilities, educational background, etc. 


# Assumptions
- Respondents answered the survey questions honestly. 
- Survey questions were not biased in anyway.


# Tools
Python
- Pandas
- Seaborn


# Challenges
- Dealing with the imperfections of raw data (lots of cleaning)


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

### Removing Duplicate / Irrelevant Rows
There is a useful pandas dataframe method ```duplicated``` that gives us duplicate rows. In this dataset, there are a number of rows that are filled with garbage and duplicated. 

```python
data[data.duplicated()]
```

<img src="https://i.imgur.com/B8raY49.jpg" alt="duplicate-rows" width="500">

We will go ahead and remove these rows. 

```python
data.drop(data[data['SurveyYear']== 20.040615027560197].index, inplace=True, axis=0)
```


### Replacing or Removing Outliers
Outliers only appear in numerical data. Let's take a look at the data type for each column in the data by using the ```dtypes``` method.

<img src="https://i.imgur.com/XB7R8E5.jpg" alt="data-types" width="500">

The columns with numerical data types (float, integer, double) is what we are looking for. What I notice is that 'HoursWorkedPerWeek' and 'TelecommuteDaysPerWeek' are not numerical. This is because they both have 'Not Asked' values which was explained at the top of this post. Even if one response is non-numerical, the entire column cannot be of a numerical datatype like float or int. With more exploration, it was found that 'TelecommuteDaysPerWeek' was a multiple choice quesiton which had responses like 'None, or less than 1 day per week' and '5 or more', and the 'HoursWorkedPerWeek' column just had 'Not Asked' entries. 



Now let's use the ```describe``` method on our data to see some key numerical/statistical features of each numerical column such as mean, standard deviation, max, etc.

```python
data.describe()
```

<img src="https://i.imgur.com/jL9aNcZ.jpg" alt="outliers" width="500">


The mean and standard deviations look fine, but take a look at the max values. There is no possible way someone has been working in data for 2020 years! They must have mistaken it for the year that they became a data professional. Also, there is no way a company has had their databse for almost 54 thousand years. This means there is atleast one outlier value for both columns. Let's look for more potential outliers in the 'YearsWithThisDatabase' and 'YearsWithThisTypeOfJob' columns.


Let's use the ```quantile``` method to check the 99th percentile values of the user responses (i.e the largest responses which are greater than 99% of the responses).
```python
data[data['YearsWithThisDatabase'] > data['YearsWithThisDatabase'].quantile(.99)][['YearsWithThisDatabase']].sort_values('YearsWithThisDatabase')
```

<img src="https://i.imgur.com/2On8aRS.jpg" alt="years-w-db-outliers" width="200">

There are 8 noticable outliers. Anything of the responses above 40 years is impossible, so let's replace those with mean values, since we can still use the responses. The respondent could have misunderstood the question or mistyped.


```python
data[data['YearsWithThisDatabase'] > 40] = data['YearsWithThisDatabase'].mean()
```


Now that we have replaced the outliers with the mean of the responses, plotting a histogram of the responses will show us a more realistic distribution.


<img src="https://i.imgur.com/8KKhxTJ.jpg" alt="histogram-years-w-db" width="500">


Next, let's check the 'YearsWithThisTypeOfJob' column for outliers.

```python
data[data['YearsWithThisTypeOfJob'] > data['YearsWithThisTypeOfJob'].quantile(.999)][['YearsWithThisTypeOfJob']].sort_values('YearsWithThisTypeOfJob')
```

<img src="https://i.imgur.com/wERtnU7.jpg" alt="outlier-years-w-this-job" width="300">

One respondent clearly misunderstood the question. There is no way they've worked 2020 years at this job, but their answer for other questions seem legitamate, so let's replace their response (row 628) with a mean value.

```python
data.at[628,'YearsWithThisTypeOfJob'] = data['YearsWithThisTypeOfJob'].mean()
```

<img src="https://i.imgur.com/08ECYl0.jpg" alt="histogram-years-w-this-job" width="500">

Now the histogram looks reasonable. 


### Renaming Columns

In one column, 'Survey Year', the words are separated by a space. The rest of the column names are not separated with a space. Let's fix this by replacing the column name.

```python
data.rename(columns={"Survey Year": "SurveyYear"}, inplace=True)
```



## Step 3: EDA





## Step 4: Visualization


## Step 5: Dashboard


# Conclusion
