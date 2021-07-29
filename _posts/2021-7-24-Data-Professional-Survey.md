---
layout: post
title:  "Data Professionals Survey Analysis"
info: "An analysis project on a raw, public survey dataset."
tech: "Python"
type: Project
thumbnail: https://blog.timescale.com/content/images/2019/02/_IoT---Relational-Database.png
---


# Goal
To perform data cleaning and analysis on the data to answer this question: How/Where do data analysts, engineers, and scientists differ? This could be interms of income, responsibilities, educational background, etc. 


## Assumptions
- Respondents answered the survey questions honestly. 
- Survey questions were not biased in anyway.


# Tools
Python
- Pandas
- Seaborn
- Matplotlib


# Data Overview
The dataset is available [here](https://data.world/finance/data-professional-salary-survey).
Some notes that the dataset uploader left for us:
- The spreadsheet includes the results for all 3 years. We’ve gradually asked more questions over time, so if a question wasn’t asked in a year, the answers are populated with Not Asked.
- The postal code field was totally optional, and may be wildly unreliable. Folks asked to be able to put in small portions of their zip code, like the leading numbers.
  

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

The columns with numerical data types (float, integer, double) is what we are looking for. What I notice is that 'HoursWorkedPerWeek' and 'TelecommuteDaysPerWeek' are not numerical. This is because they both have 'Not Asked' values which was explained at the top of this post. Even if one response is non-numerical, the entire column cannot be of a numerical datatype like float or int. 

Upon further exploration, it was found that 'TelecommuteDaysPerWeek' was a multiple choice quesiton which had responses like 'None, or less than 1 day per week' and '5 or more', so it could be classified as categorical data which has no outliers. On the other hand, 'HoursWorkedPerWeek' was not of a numerical data type because it had 'Not Asked' entires. Therefore, we can disregard the 'Not Asked' entries and find outliers. 

Let's see what the distribution of the 'HoursWorkedPerWeek' column looks like. This line of code excludes the 'Not Asked' entries in the column and converts it to an integer type so it is graphable. 

```python
sns.histplot(data=data[data['HoursWorkedPerWeek']!= 'Not Asked']['HoursWorkedPerWeek'].astype('int'))
```

<img src="https://i.imgur.com/ZX0FsZE.jpg" alt="histogram-hrs-worked" width="500">


There are only 168 hours in a week, meaning there are invalid responses as well as some outliers. Let's look at the exact values. These two lines of code will 
1.  Create a temporary dataframe of just the 'HoursWorkedPerWeek' as an integer column
2.  Display the 99 th percentile values (greater than 99% of the responses)

```python
hrs_int = data[data['HoursWorkedPerWeek']!= 'Not Asked']['HoursWorkedPerWeek'].astype('int').to_frame()
hrs_int[hrs_int['HoursWorkedPerWeek'] > hrs_int['HoursWorkedPerWeek'].quantile(.999)][['HoursWorkedPerWeek']].sort_values('HoursWorkedPerWeek')
```

<img src="https://i.imgur.com/0rN9cig.jpg" alt="highest-hrs-worked" width="350">

All values displayed here are certainly outliers, but there is only one invalid response of 200 hours which we will replace. The rest are unlikely, but possible so we will leave them.

```python
data.loc[7371, 'HoursWorkedPerWeek'] = data['HoursWorkedPerWeek'].mean()
```


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
Now that our data is cleaned, it is ready to be analyzed. Let's go back to our main goal: to look for differences between data analysts, sciensists, and engineers in terms of various metrics. We will go through each metric and create a visualization to better understand it!

### Overview

Let's first look at how many people of each job type actually took this survey. Since the survey data showed Data Analysts as 'Analyst' and Data Engineer as 'Engineer', we quickly substitute those with their full job title. We can also create a separate dataframe which is the original dataframe but filtered so that only the jobs we are looking at are shown. We will be using this filtered version of the dataframe repeatedly throughout the EDA. 

```python
jobs = ['Data Analyst', 'Data Engineer', 'Data Scientist']

data['JobTitle'] = data['JobTitle'].str.replace('Analyst','Data Analyst').replace('Engineer', 'Data Engineer')

filtered_data = data[(data['JobTitle'] =='Data Analyst') | (data['JobTitle'] =='Data Engineer')| (data['JobTitle'] =='Data Scientist')]['JobTitle']

job_counts = filtered_data.value_counts().to_frame().reset_index()
job_counts.plot(kind='bar', x='index', xlabel='', ylabel='Respondents', legend=None)
```

<img src="https://i.imgur.com/H79zSaL.png" alt="respondents-per-job" width="400">

Definitely not enough to make any broad conclusions, especially for data scientists, as the data certainly is not able to represent the entire data community. However, we will still see what insight we can gain from this data. 


### Salary

Let's look at the average salary for each job.

```python
avg_salaries = filtered_data.groupby('JobTitle').mean()['SalaryUSD'].to_frame().reset_index()
sns.barplot(data = avg_salaries, x='JobTitle', y='SalaryUSD').set(xlabel='Job Title', ylabel='Average Salary (USD)')
```

<img src="https://i.imgur.com/n7mAtLK.png" alt="avg-salary-per-job" width="400">

We know the data is not representative because there were less than 100 respondents who were data scientists. However, based on salaries that I've seen in the past, the data does not seem too far off from the true averages. 


### Educational Background


Next, let's check the educational background of people working in each job. To do this, I decided to create 3 separate dataframes and keep them in a dictionary. Then, create three separate pie charts shown in 3 matplotlib axes. To keep the coloring consistent, I also defined a dictionary of colors for each educaitonal background type. 

```python
education = dict.fromkeys(jobs)

for job in jobs:
    education[job] = data[data['JobTitle'] == job]['Education'].value_counts().to_frame().drop('Not Asked')

pie, ax = plt.subplots(ncols=3, figsize=[15,6])
plt.subplots_adjust(hspace=0)

colors = {
    'Bachelors (4 years)': 'tab:cyan', 
    'Masters': 'tab:orange',
    'None (no degree completed)': 'dimgrey',
    'Associates (2 years)': 'tab:green',
    'Doctorate/PhD': 'tab:red'
}

for i, job in enumerate(jobs):
    education[job].plot(kind='pie', 
                        y='Education', 
                        autopct="%.1f%%", 
                        ax=ax[i], 
                        legend=False, 
                        title=job, 
                        ylabel='', 
                        colors=[colors[ed] for ed in education[job].index],
                        fontsize=15,
                        labels=None)
ax[0].legend(bbox_to_anchor=(0, 1.75), loc='upper left', ncol=1, labels=education['Data Analyst'].index, prop={'size': 15})
```

<img src="https://i.imgur.com/EJbpjvk.png" alt="educational-bg" width="400">

The one thing that stands out to me is that a majority of data scientists have a masters degree, while data analysts and data engineers primarily have a bachelors degree. 
Now, what percentage of these educational backgrounds are computer related? 


<img src="https://i.imgur.com/eVwCH5Z.png" alt="educational-bg" width="400">


Seems that a majority of data engineers and scientists have an educational background related to computers,while a majority of data analysts dont. Again, since there is a small smaple size in this survey, there may be some bias. 

### Duties
There is a column in the original dataset which lists a number of the tasks performed (duties) of each respondent. The format is in comma separated string format, where each task is separated by commas. 


<img src="https://i.imgur.com/OCQs5cL.jpg" alt="tasks-entries" width="400">

Due to the consistency of each entry, it was determined that the survey question was a checklist question where rspondents chose a number of options from a checklist, and the survey turned their response into a comma-separated string. 
To analyze the distribution tasks that each job title has, we will split the column entries into lists. Let's define a function that will help us count up the amount of respondents with each role called ```get_task_counts```. Each respondent can have multiple since the entry is a list of strings.

```python
def get_task_counts(job_title):
    tasks_total = {}
    for row in data[(data['JobTitle']==job_title) & (data['KindsOfTasksPerformed'] != np.nan)]['KindsOfTasksPerformed']:
        if isinstance(row, str):
            tasks = row.split(', ')
        for task in tasks:
            if task not in tasks_total.keys():
                tasks_total[task] = 1
            else:
                tasks_total[task] += 1
    
    return pd.DataFrame.from_dict(tasks_total, orient='index', columns=['Count']).drop(['Not Asked'], axis=0).sort_values('Count', ascending=False)
```

Now, we can use this function to, again, create a separate dataframe for each job type and plot them in a matplotlib subplot.


```python
pie, ax = plt.subplots(ncols=3, figsize=[20,6])
for i, job in enumerate(jobs):
    get_task_counts(job).plot(kind='pie', 
                              y='Count', 
                              autopct="%.1f%%", 
                              ax=ax[i], 
                              legend=False, 
                              title=job, 
                              ylabel='')
```


<img src="https://i.imgur.com/CGZduWO.png" alt="tasks-pie-chart" width="400">


# Conclusion
Some insights that we found through EDA (w/ assumptions mentioned in the "goal" section):
- Data Analysts are, on average, paid the least out of the three.
- A majority of data analysts and data engineers have a bachelor's degrees.
- A majority of data scientists have a master's degrees.
- Most data analysts have non-computer related backgrounds.
- Task distribution for each job type through visualization. 

