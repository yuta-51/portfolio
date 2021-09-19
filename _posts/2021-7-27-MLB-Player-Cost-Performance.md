---
layout: post
title:  "MLB Players Cost Performance Dashboard"
info: "Data extraction, and visualization of MLB hitting stats and salary data."
tech: "Python, Tableau"
type: Project
thumbnail: https://upload.wikimedia.org/wikipedia/en/thumb/a/a6/Major_League_Baseball_logo.svg/1200px-Major_League_Baseball_logo.svg.png
---

# Goal
Gather MLB player stats and salary data to create a dashboard showcasing salary-based player performance evaluation i.e cost-effectiveness of players and certain players who should be focused on in the current (2021) season. 

# Challenge
- Integrating data from different data sources. 
- Creating a dashboard that clearly shows KPIs and players of interest. 


# Procedure
## 1. Data Extraction (Web Scraping)
Our first step is to extract salary data from this [website](https://www.spotrac.com/mlb/rankings/salary/). 
<img src = "https://i.imgur.com/c8hnox9.jpg" width = 500>
Because the table loads dynamically, we will not be able to scrape the entire table using the requests/bs4 libraries. We will instead use Selenium to scrape the table. The HTML structure of the website is standard. The table is represented as a ```table``` element with ```tr``` elements within it. Once we have the Selenium driver options setup, we can use this code chunk to find and scrape the salary table.

```python
salary_table = driver.find_element_by_tag_name('table')
salary_data = []
for tr in salary_table.find_elements_by_tag_name('tr')[1:]:
    player_name = tr.find_elements_by_tag_name('a')[-1].text.replace(',', '')
    salary = tr.find_elements_by_tag_name('span')[-1].text.replace('$', '').replace(',', '')
    salary_data.append([player_name, salary])
```

The ```salary_data``` list in this case will hold the table data. Once the loop is complete and the entire table is contained in ```salary_data```, the data is written into a csv file using the code below.

```python
with open('mlb_salaries.csv', 'w') as file:
    file.write("player_name,salary_usd" + '\n')
    for row in salary_data:
        file.write((',').join(row) + '\n')
```


## 2. Data Gathering (CSV)
Next, we will get MLB hitting data from [baseball reference](https://www.baseball-reference.com/leagues/majors/2021-standard-batting.shtml). There is no need to scrape data from this site, as it conveniently offers a download-as-csv option.

<img src="https://i.imgur.com/lcZOtU0.jpg" width=300>


## 3. Data Cleaning
The goal in this step is to clean both CSV files and combine them into one file to import to Tableau. First, both CSV files are imported as dataframes.

```python
stats = pd.read_csv('./data/mlb_hitting_stats.csv', usecols=use_cols)
salaries = pd.read_csv('./data/mlb_salaries.csv')
```

To make the player names consistent, some changed need to be made. The ```stats``` dataframe's name column looks like this:
> 0             Cory Abbott\abbotco01  
> 1            Albert Abreu\abreual01  
> 2             Bryan Abreu\abreubr01  
> 3              José Abreu\abreujo02  
> 4        Ronald Acuna Jr.\acunaro01  

While the ```salaries``` dataframe's name column looks like this:

> 0             Mike Trout  
> 1           Jacob deGrom  
> 2            Gerrit Cole  
> 3          Nolan Arenado  
> 4      Stephen Strasburg  
 
There are some obvious formatting inconsistencies between the two datasets. On top of this, the hitting stats data puts special symbols next to players names.

> Name -- Player Name  
> Bold can mean player is active for this team  
> or player has appeared in the majors  
> \* means LHP or LHB,  
> \# means switch hitter,  
> \+ can mean HOFer.  

On top of this, the ```salaries``` dataset does not include accented characters like é. Based on these differences, a function was made to clean the name column and match the hitting stats dataset to the salaries dataset.

```python
def clean_name(input_str: str) -> str:
    """
    Reformats the player name to match the name format in salaries.
    """
    clean_str = input_str[:input_str.find('\\')].strip()
    for char in ('*', '#', '+'):
        clean_str = clean_str.replace(char, '')
    return unidecode.unidecode(clean_str)
```

The function above first strips just the full name of the player, slicing off the player ID, strips any extra spaces using the ```strip``` method, goes through each special character and removes it from the name, and finally remoes any accented characters using the ```unidecode``` function from the unidecode library. Now the function is applied to each row in the names column.

```python
stats['name'] = stats['name'].apply(lambda x: clean_name(x))
```

Since the name columns are now consistent in formatting, we can inner join the two dataframes using the ```merge``` method. 

```python
stats_salaries = stats.merge(salaries, on='name', how='inner')
```
Lastly, we want to make sure the dataset does not include MLB hitters that have biased stats. If a player had one at bat so far in the season and hit a homerun, they would have a batting average of 1. To eliminate these types of stats, we can create a column called ```pa_per_game``` which stands for plate appearances per game. We can say that we only want to look at players who get atleast 1 at bat per game (1 pa/g) using filtering. 

```python
stats_salaries['pa_per_game'] = stats_salaries['pa'] / stats_salaries['g']
stats_salaries = stats_salaries[stats_salaries['pa_per_game'] >= 1]
```

## 5. Feature engineering
Since we want to do a salary-based performance evaulation, slary-based features are added to each player. 

```python
calc_cols = ('h', 'hr', 'rbi', 'tb')
for col_name in calc_cols:
    stats_salaries = stats_salaries[stats_salaries[col_name] > 0]
    if col_name != 'tb':
        stats_salaries[f'usd_per_{col_name}'] = stats_salaries['salary_usd'] / stats_salaries[col_name]
    else:
        stats_salaries[f'usd_per_base'] = stats_salaries['salary_usd'] / stats_salaries[col_name]
```

For each column in the list ```calc_cols```, we want to calculate the cost of that statistic. For example, how much each homerun costs. We can divide the number of homeruns by their salary to attain this value. The same method is used to calculate the cost each hit, rbi, and base. At this point, the dataframe is finally ready to be used in Tableau. It is exported as a csv using the ```to_csv``` method. 

```python
stats_salaries.to_csv('mlb_hitter_salary.csv')
```


## 6. Dashboard  




<img src="https://i.imgur.com/HnsNDRb.png" width=1000>
