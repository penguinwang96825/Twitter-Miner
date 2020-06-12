# Twitter_Crawler
Crawl tweets with tweepy.

## Import Packages
```python
import multiprocessing
import tweepy
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from tqdm.notebook import tqdm
from config import *
```

## Main Code
Create a function `crawl_tweet_dataframe()` to crawl tweets using Twitter API, and return the results as a dataframe.
```python
def crawl_tweet_dataframe(hashtag, count): 
    """
    Get KEY and ACCESS TOKEN from https://developer.twitter.com/en/apps
    
    Parameters:
        hashtag: the search query string of 500 characters maximum, including operators.
        count: the number of results to try and retrieve per page.
    """
    authentication = tweepy.OAuthHandler(CONSUMER_KEY, CONSUMER_SECRET)
    authentication.set_access_token(ACCESS_TOKEN, ACCESS_TOKEN_SECRET)
    api = tweepy.API(authentication, wait_on_rate_limit=True, wait_on_rate_limit_notify=True)
    maxId = -1
    tweetCount = 0

    tweet_list = []
    while tweetCount < maxTweets: 
        if(maxId <= 0):
            newTweets = api.search(
                q=hashtag, count=tweetsPerQry, result_type="recent", tweet_mode="extended")
        else:
            newTweets = api.search(
                q=hashtag, count=tweetsPerQry, max_id=str(maxId - 1), result_type="recent", tweet_mode="extended")

        if not newTweets:
            print("Tweet Habis")
            break

        for tweet in newTweets:
            user = tweet.user.screen_name.encode('utf-8')
            content = tweet.full_text.encode('utf-8')
            date = tweet.created_at
            likes = tweet.favorite_count
            location = tweet.coordinates["coordinates"] if tweet.coordinates is not None else None
            tweet_list.append([user, content, date, likes, location])

        tweetCount += len(newTweets)
        maxId = newTweets[-1].id

    tweet_df = pd.DataFrame(tweet_list)
    tweet_df.columns = ["user", "text", "date", "likes", "location"]
    tweet_df["user"] = tweet_df["user"].apply(lambda x: x.decode("utf-8"))
    tweet_df["text"] = tweet_df["text"].apply(lambda x: x.decode("utf-8"))

    return tweet_df
```

### Evaluate
```python
tweetsPerQry = 100
maxTweets = 100000
hashtag = "#sheffield"

tweet_df = crawl_tweet_dataframe(
    hashtag=hashtag, 
    count=tweetsPerQry)
```

### Visualisation
```python
plt.figure(figsize=(10, 4))
sns.heatmap(tweet_df.isnull(), cbar=True, cmap=sns.color_palette("GnBu_d"))
plt.title("Missing Values Heatmap")
plt.show()
```