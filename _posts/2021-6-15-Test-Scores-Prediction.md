---
layout: post
title:  "Test Score Prediction using Linear Regression"
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


Data set available [here](https://www.kaggle.com/kwadwoofosu/predict-test-scores-of-students).


# Procedure

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


<img src="https://i.imgur.com/iTnwthf.png" width=500 alt="kde-plot">


The students tend to do better on the post-test compared to their pre-test (~20 points). 


### Distribution of Scores in Each Classroom
```python
fig= plt.figure(figsize=(30,10))
sns.boxplot(x='classroom', y='posttest', data=df.sort_values('posttest'))
```

<img src="https://i.imgur.com/97kOdK8.png" width=500 alt="class-scores">

### Free Lunch Qualification and Gender
```python
fig , ax= plt.subplots(1, 2, figsize=(10, 5))
sns.boxplot(x='lunch', y='posttest', data=df, ax=ax[0])
sns.boxplot(x='gender', y='posttest', data=df, ax=ax[1])
```


<img src="https://i.imgur.com/ZjgiOTb.png" width=500 alt="lunch-gender">

Just as suspsected, the students who do not qualify for free lunch (most likely more affluent) do better on the post-test. 
Gender, on the other hand, does not affect the student's test score (as expected).


### Classroom Size 
```python
sns.regplot(data=df, x='n_student', y='posttest')
```


<img src="https://i.imgur.com/2VPb3Dd.png" width=500 alt="score-nstudents">


The more students the classroom has, the worse students tend to do (small negative correlation)


### Other Features
```python
f, axes = plt.subplots(1, 3,figsize=(10, 5))
sns.boxplot(data=df, x='teaching_method', y='posttest', ax=axes[0])
sns.boxplot(data=df, x='school_type', y='posttest', ax=axes[1])
sns.boxplot(data=df, x='school_setting', y='posttest', ax=axes[2])
plt.tight_layout()
```


<img src="https://i.imgur.com/xhejPlO.png" width=500 alt="triple-bar">
All three features seem to affect student test scores in some way. 


### EDA Conclusion
All features but the identification features (classroom, school) and gender seem to have a correlation with the post-test scores. In our training data, we will only include these relevant features. 


## Step 3: Train/Test Data
The data contains many categorical features (school_setting, school_type, teaching_method, and lunch). These need to be converted into numerical values before being used as training data. We can use dummy variables to solve this issue. 

```python
x = df[['pretest', 'n_student', 'school_setting', 'school_type', 'teaching_method', 'lunch']]
y = df[['posttest']]

# Encode categorical data
x = pd.get_dummies(x)
```

There is one small thought I have: We saw earlier how the kde plot of the pre-test and post-test were highly similar (just offset by ~20 points). What if we could guess students' test scores just based off of their pre-test score? Let's try it!

```python
x_simple = x[['pretest']]
```

We can create two sets of train/test splits. One has x_simple (only including pre-test) the other (x) has all relevant features.


```python
x_train_simple, x_test_simple, y_train, y_test = train_test_split(x_simple, y, test_size = 0.4, random_state = 0)
x_train, x_test, y_train, y_test = train_test_split(x, y, test_size = 0.4, random_state = 0)
```

## Step 4: Model Building 
Creating our linear regression model using SkLearn. 


```python
model_linear = LinearRegression()
```


## Step 5: Model Evaluation
Let's create a function that will allow us to easily evluate our model using mean absolute error, mean squared error, and R^2. 


```python
def eval(y, y_hat):
    MAE = mean_absolute_error(y, y_hat)
    MSE = mean_squared_error(y, y_hat)
    r2 = r2_score(y, y_hat)
    print(f'Mean Abs Error: {MAE:.2f}\nMean Square Error: {MSE:.2f}\nR^2: {r2:.2f}')
    return (MAE, MSE, r2)
```

### Simple Linear Regression 
```python
model_linear.fit(x_train_simple, y_train)
y_hat = model_linear.predict(x_test_simple)
eval(y_test, y_hat)
```

> Mean Abs Error: 3.52  
> Mean Square Error: 18.96  
> R^2: 0.90  


### Multiple Linear Regression 
```python
model_linear.fit(x_train, y_train)
y_hat = model_linear.predict(x_test)
eval(y_test, y_hat)
```

> Mean Abs Error: 2.64  
> Mean Square Error: 10.74  
> R^2: 0.95  



# Conclusion
Using simple linear regression with the pre-test scores gave us decent results. However, the multiple linear regression model worked much better as its error values were lower and R^2 value was closer to 1. 
