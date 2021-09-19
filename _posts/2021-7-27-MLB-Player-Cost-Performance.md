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


## 2. Data Cleaning



## 3. Dashboard




<img src="https://i.imgur.com/HnsNDRb.png" width=1000>
