---
layout: post
title:  "Apex Legends Viewership on Twitch"
info: "Data extraction, analysis, and visualization of livestream viewership data on Twitch.tv."
tech: "Python, GCP Cloud SQL, Tableau"
type: Project
thumbnail: https://heavy.com/wp-content/uploads/2019/02/apex-legends-twitch-prime-pack.jpg?quality=65&strip=all
---

# Background
Twitch.tv is a livestream platform focused mainly on video games. People livestream their gameplay and interact with their viewers through a chat system. In April 2021, Twitch had approximately 9.36 million active streamers, making it one the most popular livestremaing platform for gamers. It is extremely beneficial and common for video game creaters to partner with streamers and have them stream their game because it brings more people into their game and also gives streamers the chance to grow their channel. 

In this project, we will specifically be focusing on Apex Legends streams. Apex Legends is a popular battle-royale game on both PC and console. This game was chosen due to its rise in popularity as well as my personal liking for it. 

# Goal
Gaining insights and finding trends in viewership/livestream data through extracting data from [Twitch](https://www.twitch.tv/) using web scraping and API requests, loading the data into a Google Cloud Platform Cloud SQL database instance, analyzing the data and finally visualizing it using Tableau. 


# Challenges
- Difficult to web-scrape elements from Twitch due to how its website loads html elements.
- First time using GCP and creating a connection using Python to a cloud service. 


# Tools
Python
- pyodbc
- selenium
- requests
- json    


GCP Cloud SQL Database  
SQL  
Tableau  


# Procedure

## 1. Data Extraction (Web Scraping)
Our sources of data in this project is Twitch's website and API. The Twitch API allows us to access lots of data but will not allow us to get a certain game's current viewership. It does, however, allow us to get data about the current top N streamers streaming a specific game. This is the funciton that we will be working with in the API. 

Because we still want to be able to get the current total viewership for Apex Legends, I resorted to web scraping. Twitch's website shows how many total viewers are watching a certain game as shown below:

<img src="https://i.imgur.com/szxAwyF.jpg" width=600>

However, this is an estimated value. To get the exact value, we need to actually look at the HTML of the website. Interstingly enough, the actual value of the viewership is on the website as a value for an html element. I am not a web developer so I do not know if this is a common design pattern for websites, but this, to me, was very odd and made scraping the website much harder. 


```html
<p title="136,873 Viewers" class="CoreText-sc-cpl358-0 fXUBwA">
  <strong>137K</strong> Viewers
</p>
```

On top of this, the Twitch website seems to load in data dynamically. This makes it impossible for a library like requests to directly access all elements of the page. A bypass for this was to use another web module, Selenium. Selenium has a handy class called ```WebDriverWait``` with a method ```until```, which allows the web driver to wait for a certain amount of time until a certain element on the html page is available. In our case, because the ```p``` element is difficult to scrape do to its dynamic title and common class, we will scrape the ```strong``` element. 

```python
approx_viewership = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.TAG_NAME, 'strong'))
            )
```

This line of code will load the Twitch website using the chrome driver, wait up to 10 seconds until the strong tag is scrapable, then scrape it. It will give us an approximation of the total viewership. To get the exact value, again, we need to look at the ```strong``` element's parent; the ```p``` element.

```python
actual_viewership = approx_viewership.find_element_by_xpath('..').get_attribute('title')
```

This line of code navigates to the parent element of the ```strong``` element and gets its ```title``` value, which is the actual viewership value. 

Now we are able to scrape the current total viewership of Apex Legends through web-scraping. Next, we will see how Twitch's API is used for more data extraction in this project.  

## 2. Data Extraction (API)
The first step in using Twitch's API is getting your credentals and authenticate yourself using OAUTH to get an access token. Once we have that, we can start making API calls to get livestream data. First, a dictionary is created with my credentials. It will look something like this:

```python
    auth_params = { 
        'client_id': CLIENT_ID,
        'client_secret': SECRET,
        'grant_type': 'client_credentials'
    }
```


Then, a function is created called to use this dictionary, authorize it, and return an access token. 


```python
def get_access_token(auth_params:dict) -> str:
    """
    Returns access token by authenticating client information. 
    """
    auth_url = 'https://id.twitch.tv/oauth2/token'
    auth = requests.post(url=auth_url, params=auth_params) 
    return json.loads(auth.text)['access_token']
```


Finally, a function is created which sets up a header which we will need to request data from Twitch API. 

```python
def get_auth_header(auth_params: dict) -> dict:
    """
    Retruns authentication header required for API calls.
    """
    access_token = get_access_token(auth_params)
    return {
        'Client-ID': CLIENT_ID,
        'Authorization' : 'Bearer '+ access_token,
    }
```

Our goal with this API is to get data from the top 10 current livestreams streaming Apex Legends using the [```Get Streams``` end point](https://dev.twitch.tv/docs/api/reference#get-streams). This end point takes in optional parameters like ```game_id``` which we will be specifying as Apex Legend's game id. To get its game id, we can use the ```Games``` end point. 

```python
def get_game_id(game_name:str, header:dict) -> str:
    """
    Returns game id of a game based on its name.
    """
    url = 'https://api.twitch.tv/helix/games'
    param = {
        'name': game_name
    }
    r = requests.get(url, headers=header, params=param)
    return json.loads(r.text)['data'][0]['id']
```

Now that we have our authentication header as well as our game id, we can create a function to get livestream data from the ```Get Streams``` end point.


```python
def get_top_n(game_id:str, header:dict, limit:int = TOP_N_STREAMERS) -> dict:
    """
    Returns the top streamers of a certain game based on the game's id. 
    """
    url = f'https://api.twitch.tv/helix/streams?game_id={game_id}&first={limit}'
    r = requests.get(url, headers=header)
    return json.loads(r.text)['data']
```

By default, the limit is 10 meaning the function will return JSON data from the top 10 Apex Legends streamers at that time. 


## 3. Automating Data Extraction Process
At this point, we are able to use both web-scraping and API calls to get our data from Twitch but we need some way to automate this process so we don't have to run the script every x minutes manually. There are a couple of options to automate this:

1. CRON
2. Use a workflow platform (Airflow, Luigi, Dataflow, etc.)
3. Python Script

Because this is such a simple process to automate, I chose to go with using Python. It is as easy as using the ```time``` library and using the ```sleep``` function in a while loop. After some testing, it was found that 10 minutes (600 s) is a good time in between data extraction intervals. 


## 4. Loading Data into Cloud SQL
Now we must implement a loading stage into this automated data extraction process. Cloud SQL is used so that the data is not have to be stored locally. Google Cloud free tier is used to create a Cloud SQL MySQL database instance free of charge. This database will store our Twitch data.

<img src="https://i.imgur.com/5MiJRc6.jpg" width=500>

To load data into a remote database we need to establish a connection with it. For a Windows machine, the pyodbc library can be used along with a MySQL database connector. However, for a Linux machine, it is much easier to use a library called ```mysql-connector``` as it does not require a mysql connector to be installed seperately. This function creates a connection with our remote datbase using pyodbc. 


```python
def get_connection() -> pyodbc.Connection:
    return pyodbc.connect("""
        DRIVER={MySQL ODBC 8.0 Unicode Driver}; 
        SERVER=SERVER_IP; 
        PORT=SERVER_PORT;
        DATABASE=twitch; 
        UID=UN; 
        PASSWORD=PW;""", 
        autocommit=True) 
```
 
Now that we have the data and the connection to the database, we need to set up SQL statements to insert our data into the database. For our viewership data, as it only extracts one observation per interval (every 10 minutes), a simple ```INSERT INTO``` statement works. 

```python
def viewership_insertion_query(interval: int, viewers: int) -> str:
    return f"""
        INSERT INTO viewership (
        interval_id, 
        date_time, 
        viewer_count
    )
    VALUES ({interval}, '{datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S')}', {viewership});
    """
```


However, for the livestream data, there are 10 observations per interval (the top 10 streamers). Therefore, we will need a function that dynamically extends an ```INSERT INTO``` statement so that all 10 observations can be inserted in one query. 


```python
def generate_livestream_insertion_query(interval: int, top_streamers: list[dict]) -> str:
    query = """
        INSERT INTO livestream(
            interval_id,
            stream_id,
            user_id,
            user_login,
            user_name,
            game_id,
            game_name,
            stream_type,
            viewer_count,
            started_at,
            lang,
            thumbnail,
            is_mature
        ) VALUES 
        """
    for i, streamer in enumerate(top_streamers):
        query += f"""
        (   
            {interval}, 
            {streamer['id']}, 
            {streamer['user_id']}, 
            '{streamer['user_login']}', 
            '{streamer['user_name']}',
            {streamer['game_id']}, 
            '{streamer['game_name']}', 
            '{streamer['type']}', 
            {streamer['viewer_count']}, 
            '{dateutil.parser.parse(streamer['started_at']).strftime('%Y-%m-%d %H:%M:%S')}', 
            '{streamer['language']}', 
            '{streamer['thumbnail_url']}',
            '{streamer['is_mature']}'
        )
        """
        if i != (len(top_streamers) - 1):
            query += ',\n'
        else:
            query += ';'

    return query
```
As the SQL query is a string, it can be appended to using a for loop. In every loop, an observation is addded to the insertion query. 
We have all the parts for the data extraction and loading part at this point. Let's see how this script looks when we combine everything into a main function. 


```python
if __name__ == '__main__':
    conn = get_connection()
    cursor = conn.cursor()
    running = True
    interval = get_latest_interval(cursor)
    conn.close()
    print(f"starting main loop @ interval {interval}")
    time.sleep(5)
    while running:
        try:
            conn = get_connection()
            cursor = conn.cursor()                
            top_streamers = get_top_streamers()
            viewership = get_current_viewership()

            cursor.execute(viewership_insertion_query(interval, viewership))
            cursor.execute(generate_livestream_insertion_query(interval, top_streamers))

            print(f"interval: {interval} [success]")
            interval += 1
            print("sleeping...")
            time.sleep(INTERVAL_S)   
        except Exception as E: 
            running = False
            print(f"interval: {interval} [failed]\nerror inserting data: {str(E)}")
        finally:
            conn.close()
```


Our main loop follows the following steps:
1. Connect to the Cloud SQL Database
2. Get the top 10 streamers and viewership from Twitch
3. Generate and execute the insertion queries
4. Increment the row id (interval)
5. Close connection and repeat

This script is ran for a week to gather all the data we need for analysis. 

 
## 5. Analyzing Data using Tableau

