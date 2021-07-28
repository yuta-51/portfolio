---
layout: post
title:  "Financial Data ETL"
info: "ETL Data Engineering Project."
tech: "Python"
type: Coursework
thumbnail: https://media.istockphoto.com/photos/financial-and-technical-data-analysis-graph-showing-stock-market-picture-id943292690?k=6&m=943292690&s=612x612&w=0&h=AqwqtxoCVyAmgi1sYfGwsYKHpb_6pT19AVHmzmGg-a4=
---



# Goal
The goal of this project is to extract financial data from multiple online sources via. web scraping/APIs, transforming the data as necessary, and loading the data into a database.
This project is a basic example of the extract, transform, load (ETL) process of data engineering. 


# Tools
Python
- Pandas
- Glob
- BS4
- Requests


# Procedure


## Step 1: Extract

### Web Scraping Data
The first extraction method we will use is web scraping. The goal here is to scrape table data from [a wikipedia page](https://en.wikipedia.org/wiki/List_of_largest_banks) featureing the market capitalization of banks. 
To get started, we will get the html response of the target wikipedia page using requets.

```python
url = 'https://en.wikipedia.org/wiki/List_of_largest_banks'
html_data = requests.get(url).text
```
Next, we will parse the html using BS4

```python
soup= BeautifulSoup(html_data, 'html.parser')
```

Now that we have the parsed html, we can search for table data using the ```find_all``` method. But before doing that, we need to instantiate a pandas dataframe to store the table data in.

```python
data = pd.DataFrame(columns=["Name", "Market Cap (US$ Billion)"])
```

Since there are 4 tables total on the website, we need to specify the ```tbody``` element we want. Our target table happens to be the 4th table on the website, so we look for index 3. Then, within that table, we want to look for each row. The firs row is the header row so it is skipped using list slicing ```[1:]```. Finally, for each row in the table, we want the column values with are ```td``` elements to be appended to the dataframe. 

```python
for row in soup.find_all('tbody')[3].find_all('tr')[1:]:
    col = row.find_all('td')
    data.loc[len(data)] = [col[1].text, col[2].text]
```

Now we have successfuly scraped the table off of Wikipedia. All that is left is to save this dataframe as a csv.

```python
data.to_csv('wikipedia_table_data.csv')
```


### API Data Extraction
Our next source of data extraction is an [exchange rate API](https://exchangeratesapi.io/). After signging up and obtaining an API key, we can use requests to get an API response. 


```python
url = "http://api.exchangeratesapi.io/v1/latest?base=EUR&access_key=**KEY**"
response = requests.get(url)
response.text
```


The ```reponse``` variable is the JSON response from the API and can be read as a string JSON by looking at its ```text``` attribute. We now want to store this JSON resposne into a dataframe.


``python
df = pd.read_json(response.text)
```

There are many irrelevant columns in the dataframe, as we only need the currency name, and its exchange rate. We will filter out the rest of the columns by only selecting one column, and turning it into a dataframe from a series.


```python
df = df['rates'].to_frame()
```

Now that we have our desired data in a dataframe, we can save it as a csv file.

```python
df.to_csv('exchange_rates_1.csv')
```

At this point, we have all the data that we need. Let's create a function to extract the csv files now and create a dataframe out of the scraped data.

```python
def extract_from_csv(file):
    return pd.read_json(file)
```

Extracting the exchange rate for British Pounds form the exchange rate data.

```python
exchange_rates = pd.read_csv('exchange_rates.csv', index_col=0)
exchange_rate = exchange_rates.loc['GBP']['Rates']
```



## Step 2: Transform
We will now transform the data that we extracted.

1. Market Cap. will be converted from USD to GBP
2. Market Cap. will be rounded to 3 decimal places
3. The column will be renamed from ```Market Cap (US$ Billion)``` to ```Market Cap (GBP$ Billion)```  


Let's define a function that will perform these this transformation on a dataframe.

```python
def transform(df):
    df['Market Cap (US$ Billion)'] = df['Market Cap (US$ Billion)'] * exchange_rate
    df['Market Cap (US$ Billion)'] = round(df['Market Cap (US$ Billion)'], 3)
    df.rename({"Market Cap (US$ Billion)": "Market Cap (GBP$ Billion)"}, axis=1, inplace=True)
    return df
```


## Step 3: Load
Now that we have the ability to extract and transform the data, we just need to load it as a csv or any format that can be stored in a databse.
A small function using the ```to_csv``` method will do the trick.


```python
def load(df):
    df.to_csv('bank_market_cap_gbp.csv', index=False)
```


## Step 4: Logging 
Finally, taking all the functions we've written, we can actually perform the entire ETL process while also logging the program's activity.
The ```log``` function will allow us to keep track of how much time the program spends on each step of the process in a text file. 

```python
def log(message):
    timestamp_format = '%Y-%h-%d-%H:%M:%S' # Year-Monthname-Day-Hour-Minute-Second
    now = datetime.now() # get current timestamp
    timestamp = now.strftime(timestamp_format)
    with open("logfile.txt","a") as f:
        f.write(timestamp + ',' + message + '\n')
```

Now let's put everything together!


```python
log("ETL Job Started")

# Extract Phase
log("Extract phase Started")
df =extract()
log("Extract phase Ended")

# Transform Phase
log("Transform phase Started")
df = transform(df)
log("Transform phase Ended")

#Load Phase
log("Load phase Started")
load(df)
log("Load phase Ended")
```

# Conclusion
In this project, I learned about how ETL pipeline process works. Although I was already familiar with extraction of data, transforming (based on certain requirements) and also logging the progress of the program was a new concept. I can see how ETL pipelines can become much more complex and large-scale. 
