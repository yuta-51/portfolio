---
layout: post
title:  "Cyclistic Data Analysis & Dashboard"
info: "Python analysis and Tableau dashboard using bike ride data."
tech: "Python, Tableau"
type: Coursework
thumbnail: https://miro.medium.com/max/293/1*ddC1KfTAHBXjmGseS2drRw.png
---

# Goal
Complete a capstone project for the Google Data Analytics Coursera course. In this hypothetical situation, we are working for a bike-share company which wants to maximize the
number of annual members that use their service. To do this, we will perform data analysis on 1 year's worth of bike ride data in csv form. The insights gained
from this analysis will be used to drive new marketing strategies aiming to convert members to annual members. 


# Tools
- Python
- Tableau

# Challenges
- Thinking of metrics that can be used to comapre member vs. non-member behavior based on limited data. 
- Creating an intuitive, comparative dashboard without cluttering/overcomplexity. 



# Procedure

## 1. Import the data
The data provided is split up by month. Because we are working with a year's worth of data, we have 12 csv files in total. 
All csv's are moved into a subfolder and imported as dataframes into a list. Then, the ```pd.concat``` function is used to concatenate all the dataframes in the list together
into one dataframe. 


<img src='https://i.imgur.com/2AxXMRJ.jpg' width=200>


```python
csv_folder = './csv'
trip_data = []
for filename in os.listdir(csv_folder):
    trip_data.append(pd.read_csv(f'{csv_folder}/{filename}'))
trip_df = pd.concat(trip_data)
```


## 2. Clean/Transform the data
There are several steps that need to be taken to clean this dataset. 

1. The ```dtypes``` attribute shows us the data type of each column in the dataframe. 
<img src='https://i.imgur.com/EV5sPOq.jpg' width=200>
It can be observed that the ```started_at``` and ```ended_at``` columns are strings instead of datetime objects. We can fix this by using the ```to_datetime``` method and specifying the date format.

```python
date_format = '%Y-%m-%d %H:%M:%S'
df['started_at'] = pd.to_datetime(df['started_at'], format=date_format)
df['ended_at'] = pd.to_datetime(df['ended_at'], format=date_format)
```

2. The ```end_station_id``` and ```start_station_id``` are strings instead of integers. We can use the ```to_numeric``` function to fix this.
```python
  df['start_station_id'] = pd.to_numeric(df['start_station_id'], errors='coerce').astype('Int64')
  df['end_station_id'] = pd.to_numeric(df['end_station_id'], errors='coerce').astype('Int64')
```

3. The ```started_at``` value should be before the ```ended_at``` value in every row or it would not make sense. However, upon inspection, there are rows that do not pass this
simple error check. We can remove these by simply filtering out the dataframe. Since both columns are now datetime objects, we can compare them directly. 

```python
df = df[df['started_at'] <= df['ended_at']]
```

These three steps can be converted into a function called ```clean```, which will take in the combined dataframe and clean it. 

```python
def clean(df: pd.DataFrame) -> pd.DataFrame:
    df['started_at'] = pd.to_datetime(df['started_at'], format='%Y-%m-%d %H:%M:%S')
    df['ended_at'] = pd.to_datetime(df['ended_at'], format='%Y-%m-%d %H:%M:%S') 
    df['start_station_id'] = pd.to_numeric(df['start_station_id'], errors='coerce').astype('Int64')
    df['end_station_id'] = pd.to_numeric(df['end_station_id'], errors='coerce').astype('Int64')
    df = df[df['started_at'] <= df['ended_at']]
    return df
```


## 3. Add features
Although this step can be done in Tableau, we will do it in our Python environment. Some features that I would like to add in order to get more out of this dataset is
a ```duration``` and ```day_of_week``` feature. 

The ```duration``` feature can be derived through subtracting the end and start time. Again, since they are datetime objects, they can be subtracted to produce another datetime object.
```python
 df['duration'] = (df['ended_at'] - df['started_at']).dt.total_seconds().div(60).astype(float)
```

The ```day_of_week``` column will express each day of the week as an integer between 0 - 6, where 0 is Monday, 1 is Tuesday, and so on. The ```weeekday``` method will help us do this.

```python
df['day_of_week'] = df['started_at'].apply(lambda x: x.weekday())
```

To get rid of outliers in the ```day_of_week``` column, we'll use Scipy library to get the zscore of the column, and filter out the outliers.


```python
df = df[(np.abs(stats.zscore(df['duration'])) < 3)]
```

Final function is shown below.

```python
def add_features(df: pd.DataFrame) -> pd.DataFrame:
    df['duration'] = (df['ended_at'] - df['started_at']).dt.total_seconds().div(60).astype(float)
    df['day_of_week'] = df['started_at'].apply(lambda x: x.weekday())
    df = df[(np.abs(stats.zscore(df['duration'])) < 3)]
    return df
```


## 4. Export CSV

We will run the dataframe through both functions (```clean``` and ```add_features```) and save it as a csv file to import into Tableau.

```python
trip_df = add_features(clean(trip_df))
trip_df.to_csv('./output/trip_data.csv')
```


## 4. Think of comparative metrics
Let's refer back to our original goal: to find behavioral differneces between members and non-members. Here are some methods of comparison that I thought of:
1. Total rides at each hour of the day
2. Total rides on each day of the week
3. Top routes (start/end station) taken
4. Percentage of ride where start/end station are the same
5. Average ride duration


## 5. Dashboard
Implementing each comparison as a table, chart, or graph in Tableau, our final dashbaord is complete. This dashboard aims to compare the differences between how members and non-members use the bike share service and how these differences can turn into insights to drive new marketing strategies for the company. 
The final dashboard can be seen below as well as [here](https://public.tableau.com/views/BikeShare_16295803525410/Dashboard?:language=en-US&:display_count=n&:origin=viz_share_link) on my Tableau profile. 
 
<img src="https://i.imgur.com/O7L9LT5.jpg" width=1000>


## 6. Insights
1. Members tend to start riding earlier in the day compared to non-members most likely to commute to work or school. 
2. The number of non-member rides exceed the number of member rides only on the weekend.
3. Non-members tend to rent and return bikes on at the same station.
    - Notably, stations at "Streeter Dr & Grand Ave" and "Lake Shore Dr & Monroe Street" are the most popular among non-members.

Based on these data-driven insights, the broad trend seems to be that there are more member rides because members tend to use Cyclistic to commute to work or school. However, when it comes to sight seeing or traveling on the weekend for fun, there are more non-member riders. To convert the non-members to members, the company should insentivie using their bikes to commute to work/school and go to the popular non-member stations to advertise during the weekend to advertise membership deals/benefits. 

## 7. Strategy
Based on the 3 main insights, we can come up with a broad marketing strategy outline.
> Insentivize using Cyclistic bikes to commute to school or work (student discount? partnerships?). During the weekend, advertise the bike-share service as well as benefits in becoming a member at "Streeter Dr & Grand Ave" and "Lake Shore Dr & Monroe Street", which are both extremely popular among non-members. 
