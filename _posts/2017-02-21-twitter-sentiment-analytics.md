
The purpose of this analysis is to take an initial look at monthly Twitter sentiment. If you haven't yet, be sure to check-out my Twitter Sentiment - Gather Data post as it dives into how I gathered the data. There's so much to explore but in order to refine my scope, I focused on one sentiment metric: the probability of positive sentiment and deviations from the mean. Find the code at my [GitHub repo](https://github.com/vicmora/twitter_sentiment) as well.

```python
%matplotlib inline
%pylab inline
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import warnings
import plotly
import plotly.graph_objs as go
warnings.filterwarnings('ignore')
pylab.rcParams['figure.figsize'] = (12, 6)
plotly.offline.init_notebook_mode()
pd.set_option('display.float_format', lambda x: '%.4f' % x)
```

    Populating the interactive namespace from numpy and matplotlib
    

### Load data
Load the historical Twitter data scraped using the tweets_export.py script. Next, load the Twitter username data using the twitter_api.py script.


```python
tweets = pd.read_csv('data/eem_hyg_tweets.csv', encoding='utf-8',
                   usecols=['index','username', 'retweets', 'favorites', 'polarity', 'ticker', 'p_pos', 'p_neg'])
usernames = pd.read_csv('data/twitter_usernames.csv', encoding='utf-8',
                   usecols=['screen_name', 'followers_count'])
```


```python
tweets.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>username</th>
      <th>retweets</th>
      <th>favorites</th>
      <th>polarity</th>
      <th>ticker</th>
      <th>p_pos</th>
      <th>p_neg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2015-06-29 16:44:07</td>
      <td>rosnerstocks</td>
      <td>0</td>
      <td>0</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.6637</td>
      <td>0.3363</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2015-06-29 16:35:08</td>
      <td>Jake132013</td>
      <td>0</td>
      <td>0</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.6315</td>
      <td>0.3685</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015-06-29 16:30:44</td>
      <td>ETFtrends</td>
      <td>0</td>
      <td>0</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.7585</td>
      <td>0.2415</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015-06-29 16:18:49</td>
      <td>nicohof1</td>
      <td>0</td>
      <td>0</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.6768</td>
      <td>0.3232</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2015-06-29 15:43:13</td>
      <td>rosnerstocks</td>
      <td>0</td>
      <td>0</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.7626</td>
      <td>0.2374</td>
    </tr>
  </tbody>
</table>
</div>




```python
usernames.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>followers_count</th>
      <th>screen_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1477.0000</td>
      <td>rosnerstocks</td>
    </tr>
    <tr>
      <th>1</th>
      <td>941.0000</td>
      <td>Jake132013</td>
    </tr>
    <tr>
      <th>2</th>
      <td>22857.0000</td>
      <td>ETFtrends</td>
    </tr>
    <tr>
      <th>3</th>
      <td>825.0000</td>
      <td>nicohof1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1006.0000</td>
      <td>VRodrigoFP</td>
    </tr>
  </tbody>
</table>
</div>



### Combine both datasets
Combine the historical tweets and usernames data into one dataframe and drop unused columns.


```python
data = tweets.merge(usernames, how='outer', left_on='username', right_on='screen_name')
data['date'] = pd.to_datetime(data['index'])
data.drop(['screen_name', 'username', 'index'], axis=1, inplace=True)
data.fillna(0, inplace=True)
data.set_index('date', inplace=True)
data.index = data.index.normalize()
data.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>retweets</th>
      <th>favorites</th>
      <th>polarity</th>
      <th>ticker</th>
      <th>p_pos</th>
      <th>p_neg</th>
      <th>followers_count</th>
    </tr>
    <tr>
      <th>date</th>
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
      <th>2015-06-29</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.6637</td>
      <td>0.3363</td>
      <td>1477.0000</td>
    </tr>
    <tr>
      <th>2015-06-29</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.7626</td>
      <td>0.2374</td>
      <td>1477.0000</td>
    </tr>
    <tr>
      <th>2015-06-29</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.7013</td>
      <td>0.2987</td>
      <td>1477.0000</td>
    </tr>
    <tr>
      <th>2015-06-29</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.7451</td>
      <td>0.2549</td>
      <td>1477.0000</td>
    </tr>
    <tr>
      <th>2015-06-29</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>eem</td>
      <td>0.7558</td>
      <td>0.2442</td>
      <td>1477.0000</td>
    </tr>
  </tbody>
</table>
</div>



### Get the number of monthly tweets
The number of monthly tweets will be used in the next step in order to calculate the weight that each tweet has. First, I needed to group the data to calculate the number of daily tweets per ticker. Once that was done, the data was resampled back to days. The monthly tweet count was maintained for each day for ease of weight calculation.


```python
total = pd.DataFrame(data.groupby([pd.TimeGrouper('M', closed='right'), 'ticker'])['ticker'].count())
total.columns = ['monthly_tweet_count']
total_eem = total[total.index.get_level_values(1) == 'eem']
total_eem = total_eem.reset_index(level=1).resample('D', how='last', fill_method='bfill').set_index('ticker', append=True)
total_hyg = total[total.index.get_level_values(1) == 'hyg']
total_hyg = total_hyg.reset_index(level=1).resample('D', how='last', fill_method='bfill').set_index('ticker', append=True)
total = total_eem.append(total_hyg).sort_index()
total.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>monthly_tweet_count</th>
    </tr>
    <tr>
      <th>date</th>
      <th>ticker</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">2014-12-31</th>
      <th>eem</th>
      <td>2.0000</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>1.0000</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">2015-01-01</th>
      <th>eem</th>
      <td>1121.0000</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>874.0000</td>
    </tr>
    <tr>
      <th>2015-01-02</th>
      <th>eem</th>
      <td>1121.0000</td>
    </tr>
  </tbody>
</table>
</div>



### Create new features
Next, I calculated the weight of each tweet based on the percentage of total monthly tweets. The weight was applied to the probability that a tweet is positive (p_pos_w).


```python
df = data.set_index('ticker', append=True).join(total)
df['tweet_count'] = 1
df['weight'] = df.tweet_count / df.monthly_tweet_count
df['p_pos_w'] = df.weight*df.p_pos
df['p_neg_w'] = df.weight*df.p_neg
df['polarity_w'] = df.weight*df.polarity
df = df.drop(['p_pos', 'p_neg', 'polarity'], axis=1)
df.loc['2015-01-01']
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>retweets</th>
      <th>favorites</th>
      <th>followers_count</th>
      <th>monthly_tweet_count</th>
      <th>tweet_count</th>
      <th>weight</th>
      <th>p_pos_w</th>
      <th>p_neg_w</th>
      <th>polarity_w</th>
    </tr>
    <tr>
      <th>date</th>
      <th>ticker</th>
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
      <th rowspan="21" valign="top">2015-01-01</th>
      <th>eem</th>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>1477.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0008</td>
      <td>0.0001</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>297.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0005</td>
      <td>0.0003</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>297.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0008</td>
      <td>0.0001</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>2109.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0007</td>
      <td>0.0002</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>2388.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0003</td>
      <td>0.0006</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>2.0000</td>
      <td>13311.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0004</td>
      <td>0.0004</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>429.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0008</td>
      <td>0.0001</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>4145.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0007</td>
      <td>0.0002</td>
      <td>0.0009</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1446.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0001</td>
      <td>0.0008</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>693.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0004</td>
      <td>0.0005</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>808.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0007</td>
      <td>0.0002</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>808.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0007</td>
      <td>0.0002</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>5563.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0005</td>
      <td>0.0004</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>646.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0009</td>
      <td>0.0000</td>
      <td>-0.0007</td>
    </tr>
    <tr>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>549.0000</td>
      <td>1121.0000</td>
      <td>1</td>
      <td>0.0009</td>
      <td>0.0006</td>
      <td>0.0003</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1477.0000</td>
      <td>874.0000</td>
      <td>1</td>
      <td>0.0011</td>
      <td>0.0011</td>
      <td>0.0000</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>10561.0000</td>
      <td>874.0000</td>
      <td>1</td>
      <td>0.0011</td>
      <td>0.0007</td>
      <td>0.0004</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>1.0000</td>
      <td>1.0000</td>
      <td>10561.0000</td>
      <td>874.0000</td>
      <td>1</td>
      <td>0.0011</td>
      <td>0.0007</td>
      <td>0.0005</td>
      <td>-0.0003</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>429.0000</td>
      <td>874.0000</td>
      <td>1</td>
      <td>0.0011</td>
      <td>0.0011</td>
      <td>0.0001</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>200.0000</td>
      <td>874.0000</td>
      <td>1</td>
      <td>0.0011</td>
      <td>0.0007</td>
      <td>0.0005</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>2200.0000</td>
      <td>874.0000</td>
      <td>1</td>
      <td>0.0011</td>
      <td>0.0006</td>
      <td>0.0006</td>
      <td>0.0000</td>
    </tr>
  </tbody>
</table>
</div>



### Group into monthly data
Finally, the data was grouped into monthly frequency and aggregated. Unnecessary columns were also dropped to produce the final dataset.


```python
f = {'tweet_count':['sum'], 'retweets':['sum'], 'favorites':['sum'], 'polarity_w':['sum'],
     'p_pos_w':['sum'], 'p_neg_w':['sum'], 'followers_count':['sum'], 'monthly_tweet_count':['sum'],
     'weight':['sum']}
 
groups = df.reset_index(level=1).groupby([pd.TimeGrouper('M'),'ticker'])
df = groups.agg(f)
df.drop(['1970-01-31'], inplace=True)
df.drop(['weight', 'monthly_tweet_count'], axis=1, inplace=True)
df.columns = df.columns.get_level_values(0)
df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>retweets</th>
      <th>p_neg_w</th>
      <th>favorites</th>
      <th>tweet_count</th>
      <th>p_pos_w</th>
      <th>followers_count</th>
      <th>polarity_w</th>
    </tr>
    <tr>
      <th>date</th>
      <th>ticker</th>
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
      <th rowspan="2" valign="top">2014-12-31</th>
      <th>eem</th>
      <td>0.0000</td>
      <td>0.1843</td>
      <td>1.0000</td>
      <td>2</td>
      <td>0.8157</td>
      <td>13100.0000</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>0.0000</td>
      <td>0.4533</td>
      <td>0.0000</td>
      <td>1</td>
      <td>0.5467</td>
      <td>321.0000</td>
      <td>-0.7000</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">2015-01-31</th>
      <th>eem</th>
      <td>328.0000</td>
      <td>0.2376</td>
      <td>435.0000</td>
      <td>1121</td>
      <td>0.7624</td>
      <td>5478214.0000</td>
      <td>0.0402</td>
    </tr>
    <tr>
      <th>hyg</th>
      <td>240.0000</td>
      <td>0.2296</td>
      <td>362.0000</td>
      <td>874</td>
      <td>0.7704</td>
      <td>4343461.0000</td>
      <td>0.0311</td>
    </tr>
    <tr>
      <th>2015-02-28</th>
      <th>eem</th>
      <td>170.0000</td>
      <td>0.2992</td>
      <td>244.0000</td>
      <td>943</td>
      <td>0.7008</td>
      <td>3545623.0000</td>
      <td>0.0336</td>
    </tr>
  </tbody>
</table>
</div>



### Separate the dataset into EEM and HYG
For ease of plotting, the dataset was split into two dataframes, one for each ticker.


```python
eem_df = df.reset_index(level=1)[df.index.get_level_values(1) == 'eem']
hyg_df = df.reset_index(level=1)[df.index.get_level_values(1) == 'hyg']
```


```python
eem_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ticker</th>
      <th>retweets</th>
      <th>p_neg_w</th>
      <th>favorites</th>
      <th>tweet_count</th>
      <th>p_pos_w</th>
      <th>followers_count</th>
      <th>polarity_w</th>
    </tr>
    <tr>
      <th>date</th>
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
      <th>2014-12-31</th>
      <td>eem</td>
      <td>0.0000</td>
      <td>0.1843</td>
      <td>1.0000</td>
      <td>2</td>
      <td>0.8157</td>
      <td>13100.0000</td>
      <td>0.0000</td>
    </tr>
    <tr>
      <th>2015-01-31</th>
      <td>eem</td>
      <td>328.0000</td>
      <td>0.2376</td>
      <td>435.0000</td>
      <td>1121</td>
      <td>0.7624</td>
      <td>5478214.0000</td>
      <td>0.0402</td>
    </tr>
    <tr>
      <th>2015-02-28</th>
      <td>eem</td>
      <td>170.0000</td>
      <td>0.2992</td>
      <td>244.0000</td>
      <td>943</td>
      <td>0.7008</td>
      <td>3545623.0000</td>
      <td>0.0336</td>
    </tr>
    <tr>
      <th>2015-03-31</th>
      <td>eem</td>
      <td>273.0000</td>
      <td>0.3187</td>
      <td>341.0000</td>
      <td>1005</td>
      <td>0.6813</td>
      <td>7380253.0000</td>
      <td>0.0309</td>
    </tr>
    <tr>
      <th>2015-04-30</th>
      <td>eem</td>
      <td>472.0000</td>
      <td>0.2815</td>
      <td>619.0000</td>
      <td>1182</td>
      <td>0.7185</td>
      <td>10090891.0000</td>
      <td>0.0507</td>
    </tr>
  </tbody>
</table>
</div>




```python
hyg_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ticker</th>
      <th>retweets</th>
      <th>p_neg_w</th>
      <th>favorites</th>
      <th>tweet_count</th>
      <th>p_pos_w</th>
      <th>followers_count</th>
      <th>polarity_w</th>
    </tr>
    <tr>
      <th>date</th>
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
      <th>2014-12-31</th>
      <td>hyg</td>
      <td>0.0000</td>
      <td>0.4533</td>
      <td>0.0000</td>
      <td>1</td>
      <td>0.5467</td>
      <td>321.0000</td>
      <td>-0.7000</td>
    </tr>
    <tr>
      <th>2015-01-31</th>
      <td>hyg</td>
      <td>240.0000</td>
      <td>0.2296</td>
      <td>362.0000</td>
      <td>874</td>
      <td>0.7704</td>
      <td>4343461.0000</td>
      <td>0.0311</td>
    </tr>
    <tr>
      <th>2015-02-28</th>
      <td>hyg</td>
      <td>190.0000</td>
      <td>0.2847</td>
      <td>237.0000</td>
      <td>901</td>
      <td>0.7153</td>
      <td>3561552.0000</td>
      <td>0.0571</td>
    </tr>
    <tr>
      <th>2015-03-31</th>
      <td>hyg</td>
      <td>208.0000</td>
      <td>0.2810</td>
      <td>269.0000</td>
      <td>860</td>
      <td>0.7190</td>
      <td>3926348.0000</td>
      <td>0.0439</td>
    </tr>
    <tr>
      <th>2015-04-30</th>
      <td>hyg</td>
      <td>196.0000</td>
      <td>0.2435</td>
      <td>173.0000</td>
      <td>682</td>
      <td>0.7565</td>
      <td>2314834.0000</td>
      <td>0.0554</td>
    </tr>
  </tbody>
</table>
</div>



### Plot histograms of the sentiment distribution for each ticker
From the histrograms, it looks like HYG has more of a positive sentiment bias than EEM.


```python
plt.title('EEM: Probability of Positive Sentiment Distribution')
plt.xlabel('Prob. of Positive')
plt.ylabel('Frequency')
eem_df['p_pos_w'].hist()
```




    <matplotlib.axes._subplots.AxesSubplot at 0xcb73e48>




![png](/assets/images/twitter_sentiment_1.png)



```python
plt.title('HYG: Probability of Positive Sentiment Distribution')
plt.xlabel('Prob. of Positive')
plt.ylabel('Frequency')
hyg_df['p_pos_w'].hist()
```




    <matplotlib.axes._subplots.AxesSubplot at 0xd1605c0>




![png](/assets/images/twitter_sentiment_2.png)


### Plot a bubble chart
Bubble charts are like a traditional scatter plot. The main difference is the added dimension which is the size of each marker. In this case, the x-axis is time, y-axis is sentiment value, and s-axis is the volume of tweets that month. The assumption here is that months with more tweets signal more conviction. I've also added lines for mean sentiment values for comparison.


```python
eem_mean = np.full((1, len(eem_df.index)), eem_df['p_pos_w'].mean())
hyg_mean = np.full((1, len(hyg_df.index)), hyg_df['p_pos_w'].mean())
```


```python
trace0 = go.Scatter(x=eem_df.index,
                    y=eem_df['p_pos_w'].values,
                    mode='markers',
                    name='EEM Sentiment',
                    marker=dict(size=eem_df['tweet_count'].values,
                                sizemode='area',
                                color='blue')
                   )

trace1 = go.Scatter(x=eem_df.index,
                    y=eem_mean[0],
                    name='EEM Mean Sentiment',
                    line=dict(color='black',
                                dash='dash')
                   )

trace2 = go.Scatter(x=hyg_df.index,
                    y=hyg_df['p_pos_w'].values,
                    mode='markers',
                    name='HYG Sentiment',
                    marker=dict(size=hyg_df['tweet_count'].values,
                                sizemode='area')
                   )

trace3 = go.Scatter(x=hyg_df.index,
                    y=hyg_mean[0],
                    name='HYG Mean Sentiment',
                    line=dict(color='grey',
                               dash='dash')
                   )

layout = go.Layout(title='Twitter Sentiment of EEM & HYG',
                   xaxis=dict(title='Time'),
                   yaxis=dict(title='Sentiment'))

# plot in notebook
plotly.offline.iplot({
    "data": [trace0, trace1, trace2, trace3],
    "layout": layout
}, show_link=False)
```

Click for interactive plot:

<a href="/assets/html_files/twitter_bubble_plot.html" target="_blank"><img src="/assets/images/twitter_bubble_plot.png"></a>

### Next Steps
- Explore other features such as the followers_count field as a proxy for the "reach" that each tweet has.
- Scrub Twitter data thoroughly to get rid of nonsensical tweets.
- Create and / or refine sentiment classification algorithm.
- Explore other features of the Twitter dataset such as volume of tweets, follower count, retweets, and favorites.
- Create a forecast model using live Twitter feeds.
