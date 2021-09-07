---
layout: post
title:  "Twitch Viewership Data Project"
info: "Data extraction, analysis, and visualization of livestream viewership data on Twitch.tv."
tech: "Python, GCP Cloud SQL, Tableau"
type: Project
thumbnail: https://blog.twitch.tv/assets/uploads/sports-blog3-header-1306x700.jpg
---

# Background
Twitch.tv is a livestream platform focused mainly on video games. People livestream their gameplay and interact with their viewers through a chat system. In April 2021, Twitch had approximately 9.36 million active streamers, making it one the most popular livestremaing platform for gamers. It is extremely beneficial and common for video game creaters to partner with streamers and have them stream their game because it brings more peopel into their game, resulting in more revenue. 

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

Our goal with this API is to get data from the top 10 current livestreams streaming Apex Legends using the [```Get Streams``` end point](https://dev.twitch.tv/docs/api/reference#get-streams). The 


