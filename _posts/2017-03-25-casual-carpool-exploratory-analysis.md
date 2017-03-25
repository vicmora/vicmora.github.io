

```python
%matplotlib inline
import datetime
import calendar
import pandas as pd
import seaborn as sns
import plotly
import plotly.graph_objs as go
import matplotlib.pyplot as plt
from datetime import date
from matplotlib.pylab import rcParams
rcParams['figure.figsize'] = [15, 6]
plotly.offline.init_notebook_mode()
pd.set_option('display.float_format', lambda x: '%.2f' % x)
```

# Load data
Load the prepared data from the previous step.


```python
data = pd.read_csv('data/data.csv', encoding='utf-8', index_col=0, parse_dates=['timestamp_arrive', 'timestamp_depart'])
data.head()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>weekday</th>
      <th>line_count</th>
      <th>notes</th>
      <th>timestamp_arrive</th>
      <th>timestamp_depart</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Wednesday</td>
      <td>19</td>
      <td>NaN</td>
      <td>2016-06-08 07:50:00</td>
      <td>2016-06-08 08:05:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Thursday</td>
      <td>4</td>
      <td>NaN</td>
      <td>2016-06-09 07:26:00</td>
      <td>2016-06-09 07:29:00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Thursday</td>
      <td>19</td>
      <td>Bus arrived as I got in line</td>
      <td>2016-06-09 08:01:00</td>
      <td>2016-06-09 08:21:00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Friday</td>
      <td>9</td>
      <td>NaN</td>
      <td>2016-06-10 07:30:00</td>
      <td>2016-06-10 07:38:00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Friday</td>
      <td>21</td>
      <td>Took the bus after 10 minutes</td>
      <td>2016-06-10 07:50:00</td>
      <td>2016-06-10 08:00:00</td>
    </tr>
  </tbody>
</table>
</div>



# How does the time of day impact the wait time?
Create a column for the wait time. Also create columns for the weekday and month numbers in order to cut the data in different ways.


```python
data['wait_time'] = data['timestamp_depart'] - data['timestamp_arrive']
data['wait_min'] = data['wait_time'].apply(lambda x: x.total_seconds() / 60)
data['weekday_num'] = data['timestamp_arrive'].dt.weekday
data['month_num'] = data['timestamp_arrive'].dt.month
```


```python
data.head()
```


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>weekday</th>
      <th>line_count</th>
      <th>notes</th>
      <th>timestamp_arrive</th>
      <th>timestamp_depart</th>
      <th>wait_time</th>
      <th>wait_min</th>
      <th>weekday_num</th>
      <th>month_num</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Wednesday</td>
      <td>19</td>
      <td>NaN</td>
      <td>2016-06-08 07:50:00</td>
      <td>2016-06-08 08:05:00</td>
      <td>00:15:00</td>
      <td>15.00</td>
      <td>2</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Thursday</td>
      <td>4</td>
      <td>NaN</td>
      <td>2016-06-09 07:26:00</td>
      <td>2016-06-09 07:29:00</td>
      <td>00:03:00</td>
      <td>3.00</td>
      <td>3</td>
      <td>6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Thursday</td>
      <td>19</td>
      <td>Bus arrived as I got in line</td>
      <td>2016-06-09 08:01:00</td>
      <td>2016-06-09 08:21:00</td>
      <td>00:20:00</td>
      <td>20.00</td>
      <td>3</td>
      <td>6</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Friday</td>
      <td>9</td>
      <td>NaN</td>
      <td>2016-06-10 07:30:00</td>
      <td>2016-06-10 07:38:00</td>
      <td>00:08:00</td>
      <td>8.00</td>
      <td>4</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Friday</td>
      <td>21</td>
      <td>Took the bus after 10 minutes</td>
      <td>2016-06-10 07:50:00</td>
      <td>2016-06-10 08:00:00</td>
      <td>00:10:00</td>
      <td>10.00</td>
      <td>4</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>


```python
data.describe()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>line_count</th>
      <th>wait_time</th>
      <th>wait_min</th>
      <th>weekday_num</th>
      <th>month_num</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>269.00</td>
      <td>252</td>
      <td>252.00</td>
      <td>269.00</td>
      <td>269.00</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>9.35</td>
      <td>0 days 00:05:41.666666</td>
      <td>5.69</td>
      <td>2.03</td>
      <td>6.39</td>
    </tr>
    <tr>
      <th>std</th>
      <td>7.54</td>
      <td>0 days 00:04:36.890100</td>
      <td>4.61</td>
      <td>1.31</td>
      <td>3.67</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.00</td>
      <td>0 days 00:00:00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>1.00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>3.00</td>
      <td>0 days 00:02:00</td>
      <td>nan</td>
      <td>1.00</td>
      <td>2.00</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>8.00</td>
      <td>0 days 00:05:00</td>
      <td>nan</td>
      <td>2.00</td>
      <td>7.00</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>14.00</td>
      <td>0 days 00:09:00</td>
      <td>nan</td>
      <td>3.00</td>
      <td>9.00</td>
    </tr>
    <tr>
      <th>max</th>
      <td>31.00</td>
      <td>0 days 00:20:00</td>
      <td>20.00</td>
      <td>4.00</td>
      <td>12.00</td>
    </tr>
  </tbody>
</table>
</div>



## Create interactive scatter and box plots
First off, creating a scatter plot at this state helps get a good picture of what we're working with. I added an extra dimension to the plot, the weekday, in order to see if the wait time was impacted by the day of the week. In order to create this plot, the data needed to be re-arranged.


```python
df = pd.pivot_table(data, index='timestamp_arrive', columns='weekday_num', values='wait_min')
```


```python
df = df.reset_index()
```


```python
df['time_arrive'] = df['timestamp_arrive'].dt.time
df = df.sort_values('time_arrive')
df = df.set_index('time_arrive')
df['total'] = sum([df[0].fillna(0),
                   df[1].fillna(0),
                   df[2].fillna(0),
                   df[3].fillna(0),
                   df[4].fillna(0)])
df.head()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>weekday_num</th>
      <th>timestamp_arrive</th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>total</th>
    </tr>
    <tr>
      <th>time_arrive</th>
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
      <th>06:27:00</th>
      <td>2017-03-10 06:27:00</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>4.00</td>
      <td>4.00</td>
    </tr>
    <tr>
      <th>06:30:00</th>
      <td>2017-03-16 06:30:00</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>8.00</td>
      <td>nan</td>
      <td>8.00</td>
    </tr>
    <tr>
      <th>06:31:00</th>
      <td>2017-03-17 06:31:00</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>7.00</td>
      <td>7.00</td>
    </tr>
    <tr>
      <th>06:37:00</th>
      <td>2016-08-10 06:37:00</td>
      <td>nan</td>
      <td>nan</td>
      <td>2.00</td>
      <td>nan</td>
      <td>nan</td>
      <td>2.00</td>
    </tr>
    <tr>
      <th>06:52:00</th>
      <td>2017-03-22 06:52:00</td>
      <td>nan</td>
      <td>nan</td>
      <td>0.00</td>
      <td>nan</td>
      <td>nan</td>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
</div>



### The scatter plot
With this scatter plot, we can see how wait time is impacted by time and day of the week. It looks a bit random but for the most part, the earlier and later in the day look to have shorter wait times. Also, a later day of the week seems to produce a shorter wait time as well.


```python
d = {0: 'Monday',
     1: 'Tuesday',
     2: 'Wednesday',
     3: 'Thursday',
     4: 'Friday',
     'total': 'Total'}

chart_data = []

for key, value in d.items():
    trace = go.Scatter(x=df.index,
                    y=df[key],
                    mode='markers',
                    name=value
                    )
    chart_data.append(trace)
    
layout = go.Layout(title='Time of Day vs Wait Time',
                   xaxis=dict(title='Time'),
                   yaxis=dict(title='Wait Time (min)'))

# plot in notebook
plotly.offline.iplot({
    "data": chart_data,
    "layout": layout
}, show_link=False)
```
Click for interative plot:

<a href="/assets/html_files/cc_scatter_plot.html" target="_blank"><img src="/assets/images/cc_scatter_plot.png"></a>


### The box plot
With this plot, we can see that at any given time, there could be a wide spread of wait times. This is telling me that we need to find another feature that also impacts the wait time.


```python
trace0 = go.Box(x=df.index,
                    y=df['total'].values,
                    boxpoints=False,
                    name='Time of Day vs Wait Time'
                   )

layout = go.Layout(title='Time of Day vs Wait Time',
                   xaxis=dict(title='Time'),
                   yaxis=dict(title='Wait Time (min)'))

# plot in notebook
plotly.offline.iplot({
    "data": [trace0],
    "layout": layout
}, show_link=False)
```

Click for interative plot:

<a href="/assets/html_files/cc_box_plot.html" target="_blank"><img src="/assets/images/cc_box_plot.png"></a>


## Do wait times vary across months?
I discussed in my initial post that the school year or even weather seems to have an impact on the wait times. The information that the school year and weather provide can also be embedded in the months. Rainy weather usually occurs at certain times (months) of the year. Also, the school year spans several months of each year.

Let's create a new dataframe to explore the month impact on wait times.


```python
month_df = data[['wait_min', 'timestamp_arrive', 'month_num']]
month_df['timestamp_arrive'] = month_df['timestamp_arrive'].apply(lambda x: x.replace(month=1, day=1, year=2017))
month_df = month_df.set_index('timestamp_arrive')
month_df = month_df.sort_index()
month_df.head()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>wait_min</th>
      <th>month_num</th>
    </tr>
    <tr>
      <th>timestamp_arrive</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-01-01 06:27:00</th>
      <td>4.00</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2017-01-01 06:30:00</th>
      <td>8.00</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2017-01-01 06:31:00</th>
      <td>7.00</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2017-01-01 06:37:00</th>
      <td>2.00</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2017-01-01 06:52:00</th>
      <td>0.00</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



Since I'd like to see the impact of the month on the time of day, I needed to get creative with my data manipulation. First, all timestamps needed to be converted to the same day. I couldn't just take the time in this case because I knew I wanted to group the times by 10 minute buckets. For each 10 minute bucket, I calculated the average wait time. From there, we can compare each bucketed time across various months.


```python
grp = month_df.groupby([pd.TimeGrouper(freq='10min'), 'month_num'])
```


```python
aggdf = grp.agg(['mean'])
```


```python
aggdf.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th>wait_min</th>
    </tr>
    <tr>
      <th></th>
      <th></th>
      <th>mean</th>
    </tr>
    <tr>
      <th>timestamp_arrive</th>
      <th>month_num</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-01-01 06:20:00</th>
      <th>3</th>
      <td>4.00</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">2017-01-01 06:30:00</th>
      <th>3</th>
      <td>7.50</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2.00</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">2017-01-01 06:50:00</th>
      <th>2</th>
      <td>0.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
</div>



### Setup the plot
Just like the previous plots, I wanted to maintain the time index static so I had to pivot the dataframe in order to get the correct plotting points.


```python
m = {1: 'January',
     2: 'February',
     3: 'March',
     6: 'June',
     7: 'July',
     8: 'August',
     9: 'September',
     10: 'October',
     11: 'November',
     12: 'December'}
```


```python
monthdf = aggdf.unstack()
```


```python
monthdf.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="10" halign="left">wait_min</th>
    </tr>
    <tr>
      <th></th>
      <th colspan="10" halign="left">mean</th>
    </tr>
    <tr>
      <th>month_num</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>11</th>
      <th>12</th>
    </tr>
    <tr>
      <th>timestamp_arrive</th>
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
      <th>2017-01-01 06:20:00</th>
      <td>nan</td>
      <td>nan</td>
      <td>4.00</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
    </tr>
    <tr>
      <th>2017-01-01 06:30:00</th>
      <td>nan</td>
      <td>nan</td>
      <td>7.50</td>
      <td>nan</td>
      <td>nan</td>
      <td>2.00</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
    </tr>
    <tr>
      <th>2017-01-01 06:50:00</th>
      <td>nan</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>0.00</td>
      <td>nan</td>
      <td>nan</td>
    </tr>
    <tr>
      <th>2017-01-01 07:00:00</th>
      <td>nan</td>
      <td>nan</td>
      <td>1.00</td>
      <td>1.00</td>
      <td>0.00</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>0.00</td>
      <td>nan</td>
    </tr>
    <tr>
      <th>2017-01-01 07:10:00</th>
      <td>0.80</td>
      <td>3.50</td>
      <td>nan</td>
      <td>nan</td>
      <td>2.50</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
      <td>nan</td>
    </tr>
  </tbody>
</table>
</div>




```python
chart_data = []

for key, value in m.items():
    trace = go.Scatter(x=monthdf.index.time,
                       y=monthdf['wait_min', 'mean', key],
                       name=value
                    )
    chart_data.append(trace)
    
layout = go.Layout(title='Time vs Line Count',
                   xaxis=dict(title='Time of Day'),
                   yaxis=dict(title='Wait Time'))

# plot in notebook
plotly.offline.iplot({
    "data": chart_data,
    "layout": layout
}, show_link=False)
```

Click for interative plot:

<a href="/assets/html_files/cc_line_plot.html" target="_blank"><img src="/assets/images/cc_line_plot.png"></a>


I used the previous viz to compare one month to another. The only thing that really stood out to me is that July seems to have longer wait times on average. This may be explained by less people driving because of no school and vacations over the summer months. We can see that a little clearer when we explore particular points in time across months.

7:20am to 7:30am


```python
monthdf.loc[datetime.datetime(2017, 1, 1, 7, 20, 0)]
```




                    month_num
    wait_min  mean  1           0.80
                    2           1.50
                    3           0.50
                    6           3.00
                    7           7.33
                    8            nan
                    9            nan
                    10          5.00
                    11           nan
                    12           nan
    Name: 2017-01-01 07:20:00, dtype: float64



7:40am to 7:50am


```python
monthdf.loc[datetime.datetime(2017, 1, 1, 7, 40, 0)]
```




                    month_num
    wait_min  mean  1            5.33
                    2            5.00
                    3           10.00
                    6           13.50
                    7           10.20
                    8            8.00
                    9            4.80
                    10           7.67
                    11           7.00
                    12           4.00
    Name: 2017-01-01 07:40:00, dtype: float64



8am to 8:10am


```python
monthdf.loc[datetime.datetime(2017, 1, 1, 8, 0, 0)]
```




                    month_num
    wait_min  mean  1            4.80
                    2            3.50
                    3            3.50
                    6           14.50
                    7            5.25
                    8            5.12
                    9            6.82
                    10           6.00
                    11           4.50
                    12           7.33
    Name: 2017-01-01 08:00:00, dtype: float64



# Line Count
The obvious feature here might be the line count. Let's take a look at a scatter plot of the line count against the wait time.


```python
data.plot(kind='scatter', x='line_count', y='wait_min', title='Line Count vs Wait Time')
```

![png](/assets/images/cc_scatter_plot_2.png)


From the scatter plot above, it is confirmed that the length of the line is correlated with the wait time.

# Next Steps
In the next post, I'll create a predictive model to predict the wait minutes at a particular time of day using the features that we explored above: time of day, line_count, month, and day of the week. 
