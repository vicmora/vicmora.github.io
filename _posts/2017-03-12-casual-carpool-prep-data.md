
Living in the SF Bay Area offers very interesting experiences. My commute is one of them. I could use the traditional services, BART or AC Transit, to commute to San Francisco, however, neither are as enticing as [SF Casual Carpool](http://sfcasualcarpool.com/).

Commuting to San Francisco can be tough, especially during rush hour. The Bay Bridge is the only way to get to San Francisco from the East Bay and traffic is normally pretty heavy. However, using the carpool lane can significantly reduce a driver's commute. Also, the bridge toll during rush hour is \$6 while the carpool lane toll is \$2.50. The only caveat here is that the carpool lane requires at least 3 people in a vehicle.

This is where SF Casual Carpool comes into play! Drivers pick-up enough passengers, at designated pick-up locations, to fulfill the carpool requirement. Drivers also have the option to ask passengers to contribute $1 for the ride. In my experience, drivers rarely ask so passengers offer the dollar. Drivers rarely accept it though. At first glance, people are very skeptical of this service, but we've been using it for a little over a year and it's been great!

My wife and I use this service as passengers. Often times, we find ourselves waiting in line for an undetermined amount of time. Faced with the need to understand our wait times, I started tracking a few data points related to my daily experience, and convinced my wife to do so as well. In a Google Sheet, we log the date, weekday, number of people in line, line-up time, pick-up time, and any notes we find relevant. The other twist here is that the designated pick-up location is right next to an AC Transit bus stop which also gets us to work. In the case that the line is too long, we opt for the bus.

The goal of this analysis is to understand the wait times. When is the line the longest? When is it better to walk straight to the bus stop? Through experience, we've noticed that the weather and school schedule also have an impact on the line count and wait times. I'd also like to understand this a little better. Lastly, I'd like to create a forecast of the wait time on a particular day and time.

In this notebook, I import the data and clean it up a little. In future notebooks, I'll perform exploratory analysis and forecasts.


```python
%matplotlib inline
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from matplotlib.pylab import rcParams
rcParams['figure.figsize'] = [15, 6]
pd.set_option('display.float_format', lambda x: '%.2f' % x)
```

# Load the data
Load the Google Sheet data and read the first 5 rows just to get a feel for what the raw data looks like.


```python
data = pd.read_csv('data/raw.csv', encoding='utf-8')
data.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Day of Week</th>
      <th># in Line</th>
      <th>Line-up Time</th>
      <th>Pick-up Time</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6/8/2016</td>
      <td>Wednesday</td>
      <td>19</td>
      <td>7:50:00 AM</td>
      <td>8:05:00 AM</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6/9/2016</td>
      <td>Thursday</td>
      <td>4</td>
      <td>7:26:00 AM</td>
      <td>7:29:00 AM</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6/9/2016</td>
      <td>Thursday</td>
      <td>19</td>
      <td>8:01:00 AM</td>
      <td>8:21:00 AM</td>
      <td>Bus arrived as I got in line</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6/10/2016</td>
      <td>Friday</td>
      <td>9</td>
      <td>7:30:00 AM</td>
      <td>7:38:00 AM</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6/10/2016</td>
      <td>Friday</td>
      <td>21</td>
      <td>7:50:00 AM</td>
      <td>8:00:00 AM</td>
      <td>Took the bus after 10 minutes</td>
    </tr>
  </tbody>
</table>
</div>



# Create timestamps
The Date and Line-up Time and Pick-up Time columns can be combined into timestamp columns which are more useable.


```python
data['timestamp_arrive'] = pd.to_datetime(data['Date']+' '+data['Line-up Time'])
data['timestamp_depart'] = pd.to_datetime(data['Date']+' '+data['Pick-up Time'])
```

# Clean-up columns
Rename three columns so they are easier to reference in the future. Next, drop columns that were used to create timestamps as they are no longer required.


```python
data = data.rename(columns={'Day of Week':'weekday',
                            '# in Line':'line_count',
                            'Notes':'notes'})
```


```python
data = data.drop(['Date', 'Line-up Time', 'Pick-up Time'], axis=1)
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



# Describe numeric columns
Calculate descriptive statistics of the one numeric column.


```python
data.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>line_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>255.00</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>9.55</td>
    </tr>
    <tr>
      <th>std</th>
      <td>7.58</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>3.00</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>8.00</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>15.00</td>
    </tr>
    <tr>
      <th>max</th>
      <td>31.00</td>
    </tr>
  </tbody>
</table>
</div>



# Histogram of line count
Plot a histogram of the line count just to get an initial feel of it. One thing to keep in mind though is that the line count may be a function of the time of day. We'll explore this further in a different notebook.


```python
data['line_count'].hist(bins=20)
plt.title('Line Count Distribution')
plt.xlabel('line_count')
plt.ylabel('frequency')
```




    <matplotlib.text.Text at 0x118533588>




![png](/assets/images/cc_line_count_hist.png)


# Save prepared data
Save the prepared data for exploratory analysis and forecast creation.


```python
data.to_csv('data/data.csv')
```
