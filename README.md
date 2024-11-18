# YouTube Data API Scraping Project

![YouTube Logo](https://github.com/mhdkerol/Youtube-Data-API-Scraping-Project/blob/main/1727611626_logo-youtube-png.jpg)

## Overview

This project utilizes the YouTube Data API to scrape and extract channel data, storing it in a structured format using Pandas DataFrames. The goal is to enable efficient data retrieval and provide a foundation for further data analysis and manipulation. The project also includes simple visualizations to offer insights about the data retrieved.

## Pre-Requisites

This project requires a YouTube API key as a credential to access YouTube data.

### How to Generate a YouTube API Key

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
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
!pip install google-api-python-client
```

## Python Script

### 1. Install and Import Necessary Libraries & Define Variables

This code sets up a YouTube Data API client using Python for a personal project. It imports essential libraries for API interaction and data handling. With an API key for authentication and a list of YouTube channel IDs, it initializes the API client to extract data for analysis and visualization.

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

This code defines a function, get_channel_stats, to retrieve and process statistics for a list of YouTube channels using the YouTube Data API. The function takes the API client instance (youtube) and a list of channel IDs as parameters. It makes an API request to fetch details such as channel metadata (snippet) and statistics (statistics) for the provided channel IDs. The relevant data for each channel—such as channel name, subscriber count, total number of videos, total views, and the upload playlist ID—are extracted and stored in a dictionary, which is then appended to a list. This list, all_data, is returned, containing the gathered statistics for all specified channels. The variable channel_stats holds the results after calling the function with the given channel_ids.

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

This code snippet stores the data retrieved from the get_channel_stats function into a Pandas DataFrame named channel_data. This allows for easier manipulation and analysis of the data. It then converts the subscribers, total_video, and total_view columns to numeric data types to facilitate quantitative analysis. The DataFrame channel_data will display a structured table containing the channel statistics, with numeric values ready for further data processing and analysis.

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

This code uses Seaborn to create visualizations of channel statistics for a set of YouTube channels. First, it sets the figure size to enhance readability. The first bar plot (ax1) shows the number of subscribers for each channel, while the second bar plot (ax2) displays the total number of views per channel. The visualizations offer a clear comparison of channel performance based on subscribers and views, providing useful insights into their reach and popularity.

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

This function, get_video_ids, retrieves the video IDs from a specified YouTube playlist using the YouTube Data API. It starts by making an API request to fetch the first page of results from the playlist. The video IDs from this page are appended to the video_ids list. If the response contains a nextPageToken, indicating that more pages of results are available, the function continues to make subsequent requests to retrieve all video IDs from the playlist using pagination. This ensures that the function collects all video IDs from the specified playlist. The resulting video_ids list contains all extracted video IDs for further analysis or data retrieval.

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

This function, get_video_details, retrieves detailed information for a list of YouTube video IDs using the YouTube Data API. It processes the video IDs in chunks of 50 (due to API limitations) to make efficient requests. For each batch, it requests video details including snippet (metadata like title and publish date) and statistics (such as views, likes, and comments). The extracted data is stored in a dictionary for each video, containing the video title, published date, view count, like count, and comment count. All these dictionaries are collected into a list called all_video_stats, which is returned as the output. This output, video_details, can be used for further data analysis and visualization.

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

This code stores the detailed video data retrieved into a Pandas DataFrame named video_data. It then converts the Published_date column to a datetime object and extracts only the date for easier manipulation and analysis. Additionally, it converts the Views, Likes, and Comments columns to numeric types, enabling numerical operations such as sorting and aggregation. Finally, it displays the structured video_data DataFrame, providing a tabular view of the videos' titles, published dates, views, likes, and comments for further analysis and visualization.

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

This code sorts the video_data DataFrame by the Views column in descending order to identify the top 10 most viewed videos. It then creates a bar plot using Seaborn to visualize the top 10 videos by their view count. The x-axis represents the number of views, while the y-axis displays the titles of the videos. This visualization provides a clear overview of the most popular videos in terms of viewership.

```python

# Sort the data by views in descending order and display the top 10 most viewed videos
top10_videos = video_data.sort_values(by='Views', ascending=False).head(10)

# Create a bar plot showing the top 10 videos by views
ay1 = sns.barplot(x='Views', y='Title', data=top10_videos)

```

### 10. Analyze Videos per Month

This code first extracts the month from the Published_date column and creates a new Month column in video_data. It then groups the data by month and counts the number of videos published per month, storing the results in videos_per_month. To ensure the months are displayed in calendar order, the code uses a predefined sorting order. Finally, it creates a bar plot using Seaborn to visualize the number of videos published each month, giving a clear view of the publication trends across months.

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

## Conclusion
