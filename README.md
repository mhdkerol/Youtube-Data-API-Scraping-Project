# YouTube Data API Scraping Mini Project

![YouTube Logo](https://github.com/mhdkerol/Youtube-Data-API-Scraping-Project/blob/main/1727611626_logo-youtube-png.jpg)

## Overview

This project utilizes the YouTube Data API to scrape and extract channel data, storing it in a structured format using Pandas DataFrames. The goal is to enable efficient data retrieval and provide a foundation for further data analysis and manipulation. The project also includes simple visualizations to offer insights about the data retrieved.

## Pre-Requisites

This project requires a YouTube API key as a credential to access YouTube data.

### How to Generate a YouTube API Key

1. Go to the Google Cloud Console.
2. Create a new project or select an existing project.
3. Navigate to the "Library" section from the left-hand menu.
4. In the search bar, type "YouTube Data API v3" and click on it.
5. Click "Enable" to enable the API for your project.
6. Go to the "Credentials" tab on the left-hand menu.
7. Click on "Create Credentials" and select "API Key" from the dropdown menu.
8. Your new API key will be generated, and you can copy it for use in your project.

For more details, refer to the [YouTube Data API Documentation](https://developers.google.com/youtube/v3)

**NOTE: In this project, the YouTube API key is hidden and represented as "YOUR_YOUTUBE_API_KEY". Please replace it with your actual generated API key.**

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
            'total_view': response['items'][i]['statistics']['viewCount'],
            'playlist_id': response['items'][i]['contentDetails']['relatedPlaylists']['uploads']
        }
        all_data.append(data)
    
    return all_data

channel_stats = get_channel_stats(youtube, channel_ids)
channel_stats
```

### 3. Store Extracted Data in a DataFrame & Convert Columns to Numeric

```python

# Store the retrieved data into a Pandas DataFrame
channel_data = pd.DataFrame(channel_stats)

# Convert specific columns to numeric data types for analysis
channel_data['subscribers'] = pd.to_numeric(channel_data['subscribers'])
channel_data['total_video'] = pd.to_numeric(channel_data['total_video'])
channel_data['total_view'] = pd.to_numeric(channel_data['total_view'])
channel_data
```

### 4. Simple Visualization of Total Views and Subscribers

```python

import seaborn as sns  # Import Seaborn for visualization

# Set the figure size for better readability
sns.set(rc={'figure.figsize':(10, 10)})

# Create a bar plot for subscriber count per channel
ax1 = sns.barplot(x='channel_name', y='subscribers', data=channel_data)

# Create a bar plot for total views per channel
ax2 = sns.barplot(x='channel_name', y='total_view', data=channel_data)

```

### 5. Select a Specific Channel to Retrieve Video Stats

```python

# Retrieve playlist ID for a specific channel ('Alex The Analyst' in this case)
playlist_id = channel_data.loc[channel_data['channel_name'] == 'Alex The Analyst', 'playlist_id'].iloc[0]
playlist_id

```

### 6. Define a Function to Retrieve Video IDs from a Playlist

```python

def get_video_ids(youtube, playlist_id):
    # Initial request to get the first page of playlist items
    request = youtube.playlistItems().list(
        part='contentDetails',
        playlistId=playlist_id,
        maxResults=50
    )
    response = request.execute()

    video_ids = []

    # Loop through the first set of videos and append their video IDs
    for item in response['items']:
        video_ids.append(item['contentDetails']['videoId'])

    # Check if there are more pages of results
    next_page_token = response.get('nextPageToken')
    
    while next_page_token:
        # Request the next set of results if there's a next page
        request = youtube.playlistItems().list(
            part='contentDetails',
            playlistId=playlist_id,
            maxResults=50,
            pageToken=next_page_token
        )
        response = request.execute()

        # Append video IDs from the next page
        for item in response['items']:
            video_ids.append(item['contentDetails']['videoId'])

        # Update next page token for pagination
        next_page_token = response.get('nextPageToken')

    return video_ids

video_ids = get_video_ids(youtube, playlist_id)
video_ids

```

### 7. Define a Function to Retrieve Video Statistics

```python

def get_video_details(youtube, video_ids):
    all_video_stats = []
    
    # Loop over video IDs in chunks of 50
    for i in range(0, len(video_ids), 50):
        # Request video data for a batch of 50 videos at a time
        request = youtube.videos().list(
            part='snippet,statistics',  # Include video snippet and statistics
            id=','.join(video_ids[i:i+50])  # Join video IDs into a single string
        )
        response = request.execute()

        # Process each video in the response
        for video in response['items']:
            # Extract video statistics and create a dictionary
            video_stats = {
                'Title': video['snippet']['title'],
                'Published_date': video['snippet']['publishedAt'],
                'Views': video['statistics']['viewCount'],
                'Likes': video['statistics']['likeCount'],
                'Comments': video['statistics']['commentCount']
            }
            
            # Append the video stats to the list
            all_video_stats.append(video_stats)
    
    return all_video_stats

video_details = get_video_details(youtube, video_ids)
video_details

```

### 8. Store Video Data in a DataFrame and Clean Up the Data

```python

# Store the retrieved video data in a Pandas DataFrame
video_data = pd.DataFrame(video_details)

# Convert the 'Published_date' column to datetime and extract the date
video_data['Published_date'] = pd.to_datetime(video_data['Published_date']).dt.date

# Convert numeric columns to appropriate types
video_data['Views'] = pd.to_numeric(video_data['Views'])
video_data['Likes'] = pd.to_numeric(video_data['Likes'])
video_data['Comments'] = pd.to_numeric(video_data['Comments'])

# Display the video data
video_data

```

### 9. Sort the Data by Views and Display Top 10 Videos

```python

# Sort the data by views in descending order and display the top 10 most viewed videos
top10_videos = video_data.sort_values(by='Views', ascending=False).head(10)

# Create a bar plot showing the top 10 videos by views
ay1 = sns.barplot(x='Views', y='Title', data=top10_videos)

```

### 10. Analyze Videos per Month

```python

# Extract the month from the 'Published_date' column
video_data['Month'] = pd.to_datetime(video_data['Published_date']).dt.strftime('%b')

# Group the data by month and count the number of videos per month
videos_per_month = video_data.groupby('Month', as_index=False).size()

# Sort the months in calendar order
sort_order = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
videos_per_month.index = pd.CategoricalIndex(videos_per_month['Month'], categories=sort_order, ordered=True)
videos_per_month = videos_per_month.sort_index()

# Create a bar plot showing the number of videos per month
ay2 = sns.barplot(x='Month', y='size', data=videos_per_month)

```

