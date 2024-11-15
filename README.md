# Youtube Data API Scraping Mini Project

![](https://github.com/mhdkerol/Youtube-Data-API-Scraping-Project/blob/main/1727611626_logo-youtube-png.jpg)

## Overview

This project utilizes the YouTube Data API to scrape and extract channel data, storing it in a structured format using Pandas DataFrames. The goal is to enable efficient data retrieval and provide a foundation for further data analysis and manipulation.

## Key Features

- **YouTube Data API Integration:** Establishes secure access and communication with the YouTube Data API for extracting relevant channel data.
- **Data Extraction:** Retrieves key channel metrics, including subscriber count, total views, and video statistics for specified channels.
- **Data Structuring:** Organizes extracted data into a Pandas DataFrame, offering a streamlined and flexible structure for further analysis or integration with other data workflows.

## Python Script

### Install and import necessary libraries

```python
!pip install google-api-python-client
from googleapiclient.discovery import build
import pandas as pd
import numpy as np
```

### Define function to extract the data

```python

api_key = "AIzaSyALXOgP81UrZyKiZShyB27VcFNX1TC2agw"
channel_ids = ['UCAq9f7jFEA7Mtl3qOZy2h1A', #Zach Wilson
               'UCChmJrVa8kDg05JfCmxpLRw', #Darshil Parmar
               'UCkHdBeQ4DuvBXTahMYZVlMA', #Kenji Explains
               'UC7cs8q-gJRlGwj4A8OmCmXg', #Alex the Analyst
               'UCtoNXlIegvxkvf5Ji8S57Ag', #Lore so what
               'UCLLw7jmFsvfIVaUFsLs8mlQ', #Luke Barousse
               'UCD7FERT7OXNgLYkvEyy3qGQ',] #Zero Analyst

youtube = build('youtube', 'v3', developerKey = api_key)
youtube


def get_channel_stats(youtube, channel_ids):
    
    all_data=[]
    
    request = youtube.channels().list(
               part='snippet,contentDetails,statistics',
               id= ','.join(channel_ids))
    
    response = request.execute()
    
    
    for i in range(len(response['items'])):
        data = dict(channel_name = response['items'][i]['snippet']['title'],
                subscribers = response['items'][i]['statistics']['subscriberCount'],
                total_video = response['items'][i]['statistics']['videoCount'],
                total_view = response['items'][i]['statistics']['viewCount'])
    
        all_data.append(data)
    
    return all_data
```

