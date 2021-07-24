---
layout: post
title:  "Test Scores Prediction using Linear Regression"
info: "Predicting Students' Test Performance using regression."
tech: "Python"
type: Kaggle Kernel
---

# Goal
Based on a dataset consisting of 10 features + the target variable (test score), create a model that can predict test score based on the student's attributes. 
We will be using linear regression as our model for this task.


# Tools
* Python
  * Pandas
  * Matplotlib
  * Seaborn
  * Sklearn


# Dataset Overview
## Features
- **school**: The school the student attends
- **school_setting**: The location of the school
- **school_type**: Either public or non-public
- **classroom**: classroom identifier
- **teaching_method**: Either experimental or Standard
- **n_student**: Number of students in the class
- **gender**: Student's gender
- **lunch**: Whether or not student qualifies for free/subsidized lunch.
- **pretest**: Pre-test score	
- **posttest**: Post-test score (target variable)


# Procedures

## Step 1: Import Data
We begin by importing the csv data and dropping the student id feature as it is irrelevant to our project.

```python
df = pd.read_csv('/kaggle/input/predict-test-scores-of-students/test_scores.csv')
df.drop('student_id', axis=1, inplace=True)
```

## Step 2: EDA
We need to get to know the features better. Here are some questions/thoughts I have at the moment.
* Are students getting better or worse grades on the post test scores? 
* Does each classroom perform differnetly in terms of average test scores?
* What Does "experimental" and "standard" teaching style mean? Do they affect students' performance?
* Generally, wealthier schools (private, urban?) will have better standards for education/more access to higher quality tools for students. Is this shown in the data? 
* The number of students in the classroom most likley negativel impact performance because the instructor's attention will inevitable be more distributed. 

### Relationship Between Pre-test and Post-test Scores
```python
plt.figure(figsize=(10,5))
sns.kdeplot(data=df['pretest'], shade=True, label='Pre-test')
sns.kdeplot(data=df['posttest'], shade=True, label='Post-test')
plt.title('Distribution of Pre Test and Post Test')
plt.legend()
plt.show()
```


<img src="https://bit.ly/2UCYevg" width=500 alt="kde-plot">


The students tend to do better on the post-test compared to their pre-test (~20 points). 


### Distribution of scores in each classroom
```python
fig= plt.figure(figsize=(30,10))
sns.boxplot(x='classroom', y='posttest', data=df.sort_values('posttest'))
```

<img src="https://bit.ly/3ruRqMd" width=500 alt="class-scores">







