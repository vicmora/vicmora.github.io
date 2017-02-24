
The first step in this project was to gather Twitter data and quantify the sentiment of each tweet. The Twitter API only allowed me to retrieve history for a max of 7 days. This wasn't enough data so the data needed to be scraped. Luckily, while searching around I found a package called Get Old Tweets helped me do this. I also used another package called TextBlob to quantify the sentiment of each tweet. Find the code at my [GitHub repo](https://github.com/vicmora/twitter_sentiment) as well.

```python
import datetime
import pandas as pd
import requests
from blk import cfg
# https://github.com/Jefferson-Henrique/GetOldTweets-python
from get_old_tweets import got3 as got
from textblob import TextBlob
from textblob import Blobber
from textblob.sentiments import NaiveBayesAnalyzer
from requests_oauthlib import OAuth1
from pandas.io.json import json_normalize
```

### Create a function to quantify sentiment
Creating a custom sentiment classification algorithm would be huge investment of time so I decided to use a package that was ready out of the box. This package provides different sentiment metrics: polarity, classification, probabibilty of positive sentiment, and probability of negative sentiment. After some initial exploration, the probability of positive sentiment seemed to have the most accurate data


```python
def sentiment(df):
    tb = Blobber(analyzer=NaiveBayesAnalyzer())
    df['polarity'] = df['text'].apply(lambda x: TextBlob(x).sentiment.polarity)
    df['classification'] = df['text'].apply(lambda x: tb(x).sentiment.classification)
    df['p_pos'] = df['text'].apply(lambda x: tb(x).sentiment.p_pos)
    df['p_neg'] = df['text'].apply(lambda x: tb(x).sentiment.p_neg)
```

### Create a function that pulls historical twitter data and computes sentiment
Once the data is scraped, it is stored in a pandas dataframe and ran through the sentiment function.


```python
def query_hist(query, start_date, end_date):
    tweet_criteria = got.manager.TweetCriteria().setQuerySearch(query).setSince(start_date).setUntil(end_date)
    tweets = got.manager.TweetManager.getTweets(tweet_criteria)

    for i in range(len(tweets)):
        d = {'index':tweets[i].date, 'text':tweets[i].text, 'id':tweets[i].id, 'username':tweets[i].username,
             'retweets':tweets[i].retweets, 'favorites':tweets[i].favorites,  'mentions':tweets[i].mentions,
             'hashtags':tweets[i].hashtags, 'geo':tweets[i].geo, 'permalink':tweets[i].permalink}
        if i == 0:
            df = pd.DataFrame.from_dict(d,orient='index').T
            df.index = df['index']
            df = df.drop('index', axis=1)
        else:
            df2 = pd.DataFrame.from_dict(d,orient='index').T
            df2.index = df2['index']
            df2 = df2.drop('index', axis=1)
            df = df.append(df2)
    sentiment(df)
    
    return df
```

### Sample Twitter history data pull
Below is a sample query and it's output. This isn't the exact data I used as I used data for all of 2016.


```python
query = '$eem'
start_date = '2017-01-03'
end_date = '2017-01-05'
```


```python
df = query_hist(query, start_date, end_date)
df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>permalink</th>
      <th>mentions</th>
      <th>retweets</th>
      <th>geo</th>
      <th>hashtags</th>
      <th>text</th>
      <th>username</th>
      <th>favorites</th>
      <th>id</th>
      <th>polarity</th>
      <th>classification</th>
      <th>p_pos</th>
      <th>p_neg</th>
    </tr>
    <tr>
      <th>index</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-01-04 15:54:50</th>
      <td>https://twitter.com/ETFFunds/status/8167950507...</td>
      <td></td>
      <td>0</td>
      <td></td>
      <td>#ETF #ETFs #Stocks</td>
      <td>Stock Market Strategies: Why You Should Buy Em...</td>
      <td>ETFFunds</td>
      <td>1</td>
      <td>816795050784919552</td>
      <td>0.0</td>
      <td>pos</td>
      <td>0.619072</td>
      <td>0.380928</td>
    </tr>
    <tr>
      <th>2017-01-04 15:48:02</th>
      <td>https://twitter.com/1MinuteStock/status/816793...</td>
      <td></td>
      <td>0</td>
      <td></td>
      <td>#HOLD</td>
      <td>Recommendation #HOLD for $ EEM with UB rating ...</td>
      <td>1MinuteStock</td>
      <td>0</td>
      <td>816793339257376769</td>
      <td>0.0</td>
      <td>neg</td>
      <td>0.428831</td>
      <td>0.571169</td>
    </tr>
    <tr>
      <th>2017-01-04 15:39:42</th>
      <td>https://twitter.com/moniology/status/816791245...</td>
      <td></td>
      <td>0</td>
      <td></td>
      <td></td>
      <td>Kind've liking Mexico here. Possible multiple-...</td>
      <td>moniology</td>
      <td>0</td>
      <td>816791245112614916</td>
      <td>0.3</td>
      <td>pos</td>
      <td>0.616400</td>
      <td>0.383600</td>
    </tr>
    <tr>
      <th>2017-01-04 14:46:31</th>
      <td>https://twitter.com/DS_Investools/status/81677...</td>
      <td></td>
      <td>6</td>
      <td></td>
      <td>#Investools</td>
      <td>1/4/17 - View today's Market Forecast here: ht...</td>
      <td>DS_Investools</td>
      <td>17</td>
      <td>816777858655907841</td>
      <td>0.0</td>
      <td>pos</td>
      <td>0.993135</td>
      <td>0.006865</td>
    </tr>
    <tr>
      <th>2017-01-04 14:41:29</th>
      <td>https://twitter.com/MikeZaccardi/status/816776...</td>
      <td></td>
      <td>2</td>
      <td></td>
      <td></td>
      <td>Everything up in 17. Just like 2016. Through 2...</td>
      <td>MikeZaccardi</td>
      <td>3</td>
      <td>816776594547179523</td>
      <td>0.0</td>
      <td>neg</td>
      <td>0.451618</td>
      <td>0.548382</td>
    </tr>
  </tbody>
</table>
</div>



### Twitter API data
I was also interested in seeing if I add features not present in the scraped Twitter data. Using the Twitter API, we can pull features for each username, such as followers that can be used as a proxy for influence. I created a function that allowed me to conveniently query the Twitter API.


```python
def username_lookup(un_list):
    df = pd.DataFrame()

    for i in range(0,len(un_list),100):
        working_list = un_list[i:i+100]
        usernames = ''
        for name in working_list:
            if name == working_list[-1]:
                usernames = usernames + name
                break
            else:
                string = name + '%2C'
                usernames = usernames + string

        url = 'https://api.twitter.com/1.1/users/lookup.json?screen_name=%s' % usernames
        auth = OAuth1(cfg.API_KEY, cfg.API_SECRET, cfg.ACCESS_TOKEN, cfg.ACCESS_TOKEN_SECRET)
        r = requests.get(url, auth=auth)
        for i in range(len(working_list)):
            try:
                name_df = json_normalize(r.json()[i])
                df = df.append(name_df)
            except:
                pass
    return df
```

### Twitter API query
With the function above, we can pass a list of twitter usernames and obtain all of the username features provided. I then took all unique usernames from the data scrape above and passed them to the username_lookup function.


```python
un_list= df['username'].unique()
un_df = username_lookup(un_list)
un_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>contributors_enabled</th>
      <th>created_at</th>
      <th>default_profile</th>
      <th>default_profile_image</th>
      <th>description</th>
      <th>entities.description.urls</th>
      <th>entities.url.urls</th>
      <th>favourites_count</th>
      <th>follow_request_sent</th>
      <th>followers_count</th>
      <th>...</th>
      <th>status.retweeted_status.truncated</th>
      <th>status.source</th>
      <th>status.text</th>
      <th>status.truncated</th>
      <th>statuses_count</th>
      <th>time_zone</th>
      <th>translator_type</th>
      <th>url</th>
      <th>utc_offset</th>
      <th>verified</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>Thu Apr 24 08:33:51 +0000 2014</td>
      <td>False</td>
      <td>False</td>
      <td>New! Just Launched ! All About Exchange Traded...</td>
      <td>[]</td>
      <td>[{'expanded_url': 'http://TopETFFunds.com', 'i...</td>
      <td>3</td>
      <td>False</td>
      <td>356</td>
      <td>...</td>
      <td>NaN</td>
      <td>&lt;a href="http://12stocks.com/energy" rel="nofo...</td>
      <td>Resilient Global Growth Supports Equities.. ht...</td>
      <td>False</td>
      <td>75040</td>
      <td>Atlantic Time (Canada)</td>
      <td>none</td>
      <td>http://t.co/95gnxY4OKK</td>
      <td>-14400</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>Tue Jan 27 03:39:41 +0000 2015</td>
      <td>False</td>
      <td>False</td>
      <td>Flash, 1 minute stock insights about trending ...</td>
      <td>[]</td>
      <td>[{'expanded_url': 'http://unicornbay.com', 'in...</td>
      <td>21</td>
      <td>False</td>
      <td>1969</td>
      <td>...</td>
      <td>NaN</td>
      <td>&lt;a href="https://unicornbay.com" rel="nofollow...</td>
      <td>Do you know that #Book Value for $TIF is $23.3...</td>
      <td>False</td>
      <td>172121</td>
      <td>Pacific Time (US &amp; Canada)</td>
      <td>none</td>
      <td>https://t.co/kUdxipjwKL</td>
      <td>-28800</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>Wed Sep 15 05:20:19 +0000 2010</td>
      <td>False</td>
      <td>False</td>
      <td></td>
      <td>[]</td>
      <td>NaN</td>
      <td>306</td>
      <td>False</td>
      <td>522</td>
      <td>...</td>
      <td>NaN</td>
      <td>&lt;a href="http://stocktwits.com" rel="nofollow"...</td>
      <td>Watch what happens when 100 breaks to the upsi...</td>
      <td>False</td>
      <td>6557</td>
      <td>Pacific Time (US &amp; Canada)</td>
      <td>none</td>
      <td>None</td>
      <td>-28800</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>Sat Oct 30 18:07:33 +0000 2010</td>
      <td>False</td>
      <td>False</td>
      <td>I work at @Investools from TD Ameritrade Holdi...</td>
      <td>[{'expanded_url': 'http://bit.ly/1Ml2kKW', 'in...</td>
      <td>[{'expanded_url': 'https://www.youtube.com/use...</td>
      <td>8059</td>
      <td>False</td>
      <td>2967</td>
      <td>...</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>RT @benmillr: We're looking for somebody with ...</td>
      <td>False</td>
      <td>7166</td>
      <td>Mountain Time (US &amp; Canada)</td>
      <td>none</td>
      <td>https://t.co/qV1yLnks5T</td>
      <td>-25200</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>Thu Feb 26 22:35:08 +0000 2009</td>
      <td>False</td>
      <td>False</td>
      <td>MISO Power and Energy Trader. Chart Czar. Weat...</td>
      <td>[]</td>
      <td>[{'expanded_url': 'http://www.seeitmarket.com/...</td>
      <td>4434</td>
      <td>False</td>
      <td>2616</td>
      <td>...</td>
      <td>NaN</td>
      <td>&lt;a href="https://about.twitter.com/products/tw...</td>
      <td>@commoditywx whoa! The never-say-die blowtorch...</td>
      <td>False</td>
      <td>55318</td>
      <td>Eastern Time (US &amp; Canada)</td>
      <td>none</td>
      <td>http://t.co/RkGuTCu6Nl</td>
      <td>-18000</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 111 columns</p>
</div>



### List of username features provided by Twitter API


```python
for c in un_df.columns:
    print(c)
```

    contributors_enabled
    created_at
    default_profile
    default_profile_image
    description
    entities.description.urls
    entities.url.urls
    favourites_count
    follow_request_sent
    followers_count
    following
    friends_count
    geo_enabled
    has_extended_profile
    id
    id_str
    is_translation_enabled
    is_translator
    lang
    listed_count
    location
    name
    notifications
    profile_background_color
    profile_background_image_url
    profile_background_image_url_https
    profile_background_tile
    profile_banner_url
    profile_image_url
    profile_image_url_https
    profile_link_color
    profile_sidebar_border_color
    profile_sidebar_fill_color
    profile_text_color
    profile_use_background_image
    protected
    screen_name
    status.contributors
    status.coordinates
    status.created_at
    status.entities.hashtags
    status.entities.media
    status.entities.symbols
    status.entities.urls
    status.entities.user_mentions
    status.extended_entities.media
    status.favorite_count
    status.favorited
    status.geo
    status.id
    status.id_str
    status.in_reply_to_screen_name
    status.in_reply_to_status_id
    status.in_reply_to_status_id_str
    status.in_reply_to_user_id
    status.in_reply_to_user_id_str
    status.is_quote_status
    status.lang
    status.place
    status.possibly_sensitive
    status.quoted_status_id
    status.quoted_status_id_str
    status.retweet_count
    status.retweeted
    status.retweeted_status.contributors
    status.retweeted_status.coordinates
    status.retweeted_status.created_at
    status.retweeted_status.entities.hashtags
    status.retweeted_status.entities.symbols
    status.retweeted_status.entities.urls
    status.retweeted_status.entities.user_mentions
    status.retweeted_status.favorite_count
    status.retweeted_status.favorited
    status.retweeted_status.geo
    status.retweeted_status.id
    status.retweeted_status.id_str
    status.retweeted_status.in_reply_to_screen_name
    status.retweeted_status.in_reply_to_status_id
    status.retweeted_status.in_reply_to_status_id_str
    status.retweeted_status.in_reply_to_user_id
    status.retweeted_status.in_reply_to_user_id_str
    status.retweeted_status.is_quote_status
    status.retweeted_status.lang
    status.retweeted_status.place
    status.retweeted_status.place.bounding_box.coordinates
    status.retweeted_status.place.bounding_box.type
    status.retweeted_status.place.contained_within
    status.retweeted_status.place.country
    status.retweeted_status.place.country_code
    status.retweeted_status.place.full_name
    status.retweeted_status.place.id
    status.retweeted_status.place.name
    status.retweeted_status.place.place_type
    status.retweeted_status.place.url
    status.retweeted_status.possibly_sensitive
    status.retweeted_status.quoted_status_id
    status.retweeted_status.quoted_status_id_str
    status.retweeted_status.retweet_count
    status.retweeted_status.retweeted
    status.retweeted_status.source
    status.retweeted_status.text
    status.retweeted_status.truncated
    status.source
    status.text
    status.truncated
    statuses_count
    time_zone
    translator_type
    url
    utc_offset
    verified
    
