# YouTube Data API Scraping Mini Project

![YouTube Logo](https://github.com/mhdkerol/Youtube-Data-API-Scraping-Project/blob/main/1727611626_logo-youtube-png.jpg)

## Overview

This project utilizes the YouTube Data API to scrape and extract channel data, storing it in a structured format using Pandas DataFrames. The goal is to enable efficient data retrieval and provide a foundation for further data analysis and manipulation. The project also includes simple visualizations to offer insights about the data retrieved.

## YouTube API Documentation

[YouTube Data API Documentation](https://developers.google.com/youtube/v3)

## Key Features

- **YouTube Data API Integration:** Establishes secure access and communication with the YouTube Data API to extract relevant channel data.
- **Data Extraction:** Retrieves key channel metrics, including subscriber count, total views, and video statistics for specified channels.
- **Data Structuring:** Organizes extracted data into Pandas DataFrames, offering a streamlined and flexible structure for further analysis or integration with other data workflows.

## Installation

Before running the project, make sure to install the necessary libraries. You can install them using `pip`:

```bash
pip install google-api-python-client pandas numpy seaborn
```

## Python Script

### 1. Install and Import Necessary Libraries & Define Variables

```python

!pip install google-api-python-client  # Install the YouTube API client

from googleapiclient.discovery import build  # Import YouTube API client
import pandas as pd  # Import Pandas for data handling
import numpy as np  # Import NumPy for numerical operations

api_key = "YOUR_YOUTUBE_API_KEY"  # Your YouTube API Key
channel_ids = [
    'UCAq9f7jFEA7Mtl3qOZy2h1A',  # Zach Wilson
    'UCChmJrVa8kDg05JfCmxpLRw',  # Darshil Parmar
    'UCkHdBeQ4DuvBXTahMYZVlMA',  # Kenji Explains
    'UC7cs8q-gJRlGwj4A8OmCmXg',  # Alex the Analyst
    'UCtoNXlIegvxkvf5Ji8S57Ag',  # Lore So What
    'UCLLw7jmFsvfIVaUFsLs8mlQ',  # Luke Barousse
    'UCD7FERT7OXNgLYkvEyy3qGQ',  # Zero Analyst
]

# Initialize the YouTube API client
youtube = build('youtube', 'v3', developerKey=api_key)

```

### 2. Define a Function to Extract Channel Data

```python

def get_channel_stats(youtube, channel_ids):
    # List to store all extracted channel data
    all_data = []
    
    # API request to get channel data
    request = youtube.channels().list(
        part='snippet,contentDetails,statistics',
        id= ','.join(channel_ids)
    )
    
    # Execute the request and store the response
    response = request.execute()
    
    # Loop through each channel's response and extract relevant data
    for i in range(len(response['items'])):
        data = {
            'channel_name': response['items'][i]['snippet']['title'],
            'subscribers': response['items'][i]['statistics']['subscriberCount'],
            'total_video': response['items'][i]['statistics']['videoCount'],
            'total_view': response['items'][i]['statistics']['viewCount']
        }
        all_data.append(data)
    
    return all_data
```
