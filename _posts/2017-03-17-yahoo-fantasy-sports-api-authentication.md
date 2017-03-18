
I've been playing Fantasy Football on Yahoo for a few years and have been itching to enhance my experience with data-driven decisions. This is the year. The first thing I am doing is plugging into the Yahoo Fantasy Sports API an seeing what I can do.

The first step in this process is to create an app with Yahoo. Creating an app provides a unique consumer key and a consumer secret. With the unique consumer key and consumer secret, Yahoo is able to provide access to protected resources. Apps can be created at the following link: [Yahoo - My Apps](https://developer.yahoo.com/apps/)

In this post, I'll walk through how I connected to the Yahoo Fantasy Sports API using Python. I'll also start putting together a module for this project. It turns out that the initial setup proved to be more difficult than expected. However, I came across across the following two repos that helped:

[Python-YahooAPI](https://github.com/dkempiners/python-yahooapi/)

[Yahoo OAuth](https://github.com/josuebrunel/yahoo-oauth)

Both repos provided their solutions but they did not fit what I wanted. I decided to create my own way of authenticating and querying the api, leveraging some of the work done in the repos.


```python
import json
import time
import webbrowser
import pandas as pd
from pandas.io.json import json_normalize
from rauth import OAuth1Service
from rauth.utils import parse_utf8_qsl
```

### Load Credentials
Load a json file which has my consumer key and consumer secret.


```python
credentials_file = open('oauth.json')
credentials = json.load(credentials_file)
credentials_file.close()
```

### Create OAuth Object
OAuth object is used for the three-legged authentication required by the Yahoo API.


```python
oauth = OAuth1Service(consumer_key = credentials['consumer_key'],
                      consumer_secret = credentials['consumer_secret'],
                      name = "yahoo",
                      request_token_url = "https://api.login.yahoo.com/oauth/v2/get_request_token",
                      access_token_url = "https://api.login.yahoo.com/oauth/v2/get_token",
                      authorize_url = "https://api.login.yahoo.com/oauth/v2/request_auth",
                      base_url = "http://fantasysports.yahooapis.com/")
```

### Leg 1
Obtain a request token which identifies you as the consumer.


```python
request_token, request_token_secret = oauth.get_request_token(params={"oauth_callback": "oob"})
```

### Leg 2
Obtain authorization to access protected resources. We do this by generating the auth url, opening it in our default web browser, and entering the verification code displayed.


```python
authorize_url = oauth.get_authorize_url(request_token)
webbrowser.open(authorize_url)
verify = input('Enter code: ')
```

    Enter code: k2pm8y


### Leg 3
Obtain access tokens. Tokens expire after 3600 seconds (60 minutes) so we'd like to save the time these tokens were generated. This time can be used later to check against and refresh the tokens if necessary. Next, we'll also save the tokens in our credentials dictionary and create a tuple that is used in the creation of a session.


```python
raw_access = oauth.get_raw_access_token(request_token,
                                        request_token_secret,
                                        params={"oauth_verifier": verify})

parsed_access_token = parse_utf8_qsl(raw_access.content)
access_token = (parsed_access_token['oauth_token'], parsed_access_token['oauth_token_secret'])

start_time = time.time()
end_time = start_time + 3600

credentials['access_token'] = parsed_access_token['oauth_token']
credentials['access_token_secret'] = parsed_access_token['oauth_token_secret']
tokens = (credentials['access_token'], credentials['access_token_secret'])
```

### Start a session


```python
s = oauth.get_session(tokens)
```

### Send a query
Query the api and receive output in json format. The [Yahoo Fantasy Sports API guide](https://developer.yahoo.com/fantasysports/guide/) explains how queries are created. 


```python
url = 'http://fantasysports.yahooapis.com/fantasy/v2/leagues;league_keys=nfl.l.427049'
r = s.get(url, params={'format': 'json'})
r.status_code
```




    200



### JSON output


```python
r.json()
```




    {'fantasy_content': {'copyright': 'Data provided by Yahoo! and STATS, LLC',
      'leagues': {'0': {'league': [{'allow_add_to_dl_extra_pos': 0,
          'current_week': '16',
          'draft_status': 'postdraft',
          'edit_key': '17',
          'end_date': '2016-12-26',
          'end_week': '16',
          'game_code': 'nfl',
          'is_cash_league': '0',
          'is_finished': 1,
          'is_pro_league': '0',
          'league_id': '427049',
          'league_key': '359.l.427049',
          'league_type': 'private',
          'league_update_timestamp': '1483604946',
          'name': '#TFTI 2.0',
          'num_teams': 12,
          'renew': '348_473481',
          'renewed': '',
          'scoring_type': 'head',
          'season': '2016',
          'short_invitation_url': 'https://yho.com/nfl?l=427049&ikey=40f621140eb68802',
          'start_date': '2016-09-08',
          'start_week': '1',
          'url': 'https://football.fantasysports.yahoo.com/f1/427049',
          'weekly_deadline': ''}]},
       'count': 1},
      'refresh_rate': '60',
      'time': '90.903997421265ms',
      'xml:lang': 'en-US',
      'yahoo:uri': '/fantasy/v2/leagues;league_keys=nfl.l.427049'}}



### Pandas DataFrame
Convert the json output to a pandas dataframe.


```python
data = json_normalize(r.json(), [['fantasy_content', 'leagues', '0', 'league']])
data
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>allow_add_to_dl_extra_pos</th>
      <th>current_week</th>
      <th>draft_status</th>
      <th>edit_key</th>
      <th>end_date</th>
      <th>end_week</th>
      <th>game_code</th>
      <th>is_cash_league</th>
      <th>is_finished</th>
      <th>is_pro_league</th>
      <th>...</th>
      <th>num_teams</th>
      <th>renew</th>
      <th>renewed</th>
      <th>scoring_type</th>
      <th>season</th>
      <th>short_invitation_url</th>
      <th>start_date</th>
      <th>start_week</th>
      <th>url</th>
      <th>weekly_deadline</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>16</td>
      <td>postdraft</td>
      <td>17</td>
      <td>2016-12-26</td>
      <td>16</td>
      <td>nfl</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>12</td>
      <td>348_473481</td>
      <td></td>
      <td>head</td>
      <td>2016</td>
      <td>https://yho.com/nfl?l=427049&amp;ikey=40f621140eb6...</td>
      <td>2016-09-08</td>
      <td>1</td>
      <td>https://football.fantasysports.yahoo.com/f1/42...</td>
      <td></td>
    </tr>
  </tbody>
</table>
<p>1 rows × 25 columns</p>
</div>




```python
data.T
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>allow_add_to_dl_extra_pos</th>
      <td>0</td>
    </tr>
    <tr>
      <th>current_week</th>
      <td>16</td>
    </tr>
    <tr>
      <th>draft_status</th>
      <td>postdraft</td>
    </tr>
    <tr>
      <th>edit_key</th>
      <td>17</td>
    </tr>
    <tr>
      <th>end_date</th>
      <td>2016-12-26</td>
    </tr>
    <tr>
      <th>end_week</th>
      <td>16</td>
    </tr>
    <tr>
      <th>game_code</th>
      <td>nfl</td>
    </tr>
    <tr>
      <th>is_cash_league</th>
      <td>0</td>
    </tr>
    <tr>
      <th>is_finished</th>
      <td>1</td>
    </tr>
    <tr>
      <th>is_pro_league</th>
      <td>0</td>
    </tr>
    <tr>
      <th>league_id</th>
      <td>427049</td>
    </tr>
    <tr>
      <th>league_key</th>
      <td>359.l.427049</td>
    </tr>
    <tr>
      <th>league_type</th>
      <td>private</td>
    </tr>
    <tr>
      <th>league_update_timestamp</th>
      <td>1483604946</td>
    </tr>
    <tr>
      <th>name</th>
      <td>#TFTI 2.0</td>
    </tr>
    <tr>
      <th>num_teams</th>
      <td>12</td>
    </tr>
    <tr>
      <th>renew</th>
      <td>348_473481</td>
    </tr>
    <tr>
      <th>renewed</th>
      <td></td>
    </tr>
    <tr>
      <th>scoring_type</th>
      <td>head</td>
    </tr>
    <tr>
      <th>season</th>
      <td>2016</td>
    </tr>
    <tr>
      <th>short_invitation_url</th>
      <td>https://yho.com/nfl?l=427049&amp;ikey=40f621140eb6...</td>
    </tr>
    <tr>
      <th>start_date</th>
      <td>2016-09-08</td>
    </tr>
    <tr>
      <th>start_week</th>
      <td>1</td>
    </tr>
    <tr>
      <th>url</th>
      <td>https://football.fantasysports.yahoo.com/f1/42...</td>
    </tr>
    <tr>
      <th>weekly_deadline</th>
      <td></td>
    </tr>
  </tbody>
</table>
</div>



### Refresh access tokens
Access tokens expire after 3600 seconds. In order to send more queries after expiration, tokens must be refreshed. Luckily, we don't need to go through the full three-legged auth again.


```python
# Access tokens expire after 3,600 seconds (60 minutes). Refresh tokens and get a new session.
tokens = oauth.get_access_token(parsed_access_token['oauth_token'],
                                parsed_access_token['oauth_token_secret'],
                                params={'oauth_session_handle':parsed_access_token['oauth_session_handle']}
                               )
credentials['access_token'] = tokens[0]
credentials['access_token_secret'] = tokens[1]

start_time = time.time()
end_time = start_time + 3600

s = oauth.get_session(tokens)
```


```python
r = s.get(url, params={'format': 'json'})
r.status_code
```




    200



### Create a reusable class


```python
class YahooFantasySports:
    def __init__(self, credentials_file):
        # load credentials
        self.credentials_file = open(credentials_file)
        self.credentials = json.load(self.credentials_file)   
        self.credentials_file.close()
    
        # create oauth object
        self.oauth = OAuth1Service(consumer_key = self.credentials['consumer_key'],
                                   consumer_secret = self.credentials['consumer_secret'],
                                   name = "yahoo",
                                   request_token_url = "https://api.login.yahoo.com/oauth/v2/get_request_token",
                                   access_token_url = "https://api.login.yahoo.com/oauth/v2/get_token",
                                   authorize_url = "https://api.login.yahoo.com/oauth/v2/request_auth",
                                   base_url = "http://fantasysports.yahooapis.com/")
        # leg 1
        request_token, request_token_secret = self.oauth.get_request_token(params={"oauth_callback": "oob"})
        
        # leg 2
        authorize_url = self.oauth.get_authorize_url(request_token)
        webbrowser.open(authorize_url)
        verify = input('Enter code: ')

        # leg 3
        raw_access = self.oauth.get_raw_access_token(request_token,
                                            request_token_secret,
                                            params={"oauth_verifier": verify})

        parsed_access_token = parse_utf8_qsl(raw_access.content)
        access_token = (parsed_access_token['oauth_token'], parsed_access_token['oauth_token_secret'])

        # log time
        self.start_time = time.time()
        self.end_time = self.start_time + 3600
        
        # store tokens
        self.credentials['access_token'] = parsed_access_token['oauth_token']
        self.credentials['access_token_secret'] = parsed_access_token['oauth_token_secret']
        self.tokens = (self.credentials['access_token'], self.credentials['access_token_secret'])
        
        # start session
        self.session = self.oauth.get_session(self.tokens)
    
    def refresh_tokens(self):
        # refresh a session
        self.tokens = self.oauth.get_access_token(parsed_access_token['oauth_token'],
                                                  parsed_access_token['oauth_token_secret'],
                                                  params={'oauth_session_handle':parsed_access_token['oauth_session_handle']}
                                                 )
        
        # update stored tokens
        self.credentials['access_token'] = self.tokens[0]
        self.credentials['access_token_secret'] = self.tokens[1]

        # update log time
        self.start_time = time.time()
        self.end_time = self.start_time + 3600
        
        # start a session with updated tokens
        self.session = self.oauth.get_session(self.tokens)
```

### Test the class
Test the newly created class within the notebook.


```python
credentials_file = 'oauth.json'
yfs = YahooFantasySports(credentials_file)
```

    Enter code: stryph



```python
r = yfs.session.get(url, params={'format': 'json'})
r.status_code
```




    200




```python
r.json()
```




    {'fantasy_content': {'copyright': 'Data provided by Yahoo! and STATS, LLC',
      'leagues': {'0': {'league': [{'allow_add_to_dl_extra_pos': 0,
          'current_week': '16',
          'draft_status': 'postdraft',
          'edit_key': '17',
          'end_date': '2016-12-26',
          'end_week': '16',
          'game_code': 'nfl',
          'is_cash_league': '0',
          'is_finished': 1,
          'is_pro_league': '0',
          'league_id': '427049',
          'league_key': '359.l.427049',
          'league_type': 'private',
          'league_update_timestamp': '1483604946',
          'name': '#TFTI 2.0',
          'num_teams': 12,
          'renew': '348_473481',
          'renewed': '',
          'scoring_type': 'head',
          'season': '2016',
          'short_invitation_url': 'https://yho.com/nfl?l=427049&ikey=40f621140eb68802',
          'start_date': '2016-09-08',
          'start_week': '1',
          'url': 'https://football.fantasysports.yahoo.com/f1/427049',
          'weekly_deadline': ''}]},
       'count': 1},
      'refresh_rate': '60',
      'time': '79.923152923584ms',
      'xml:lang': 'en-US',
      'yahoo:uri': '/fantasy/v2/leagues;league_keys=nfl.l.427049'}}



### Create a module
Create a module with the created class. We'll add api queries to this module in the next post.


```python
from yahoo_fantasy_sports import YahooFantasySports as yfs_test
```


```python
credentials_file = 'oauth.json'
yfs2 = yfs_test(credentials_file)
```

    Enter code: cgarzv



```python
r2 = yfs2.session.get(url, params={'format': 'json'})
r2.status_code
```




    200




```python
r2.json()
```




    {'fantasy_content': {'copyright': 'Data provided by Yahoo! and STATS, LLC',
      'leagues': {'0': {'league': [{'allow_add_to_dl_extra_pos': 0,
          'current_week': '16',
          'draft_status': 'postdraft',
          'edit_key': '17',
          'end_date': '2016-12-26',
          'end_week': '16',
          'game_code': 'nfl',
          'is_cash_league': '0',
          'is_finished': 1,
          'is_pro_league': '0',
          'league_id': '427049',
          'league_key': '359.l.427049',
          'league_type': 'private',
          'league_update_timestamp': '1483604946',
          'name': '#TFTI 2.0',
          'num_teams': 12,
          'renew': '348_473481',
          'renewed': '',
          'scoring_type': 'head',
          'season': '2016',
          'short_invitation_url': 'https://yho.com/nfl?l=427049&ikey=40f621140eb68802',
          'start_date': '2016-09-08',
          'start_week': '1',
          'url': 'https://football.fantasysports.yahoo.com/f1/427049',
          'weekly_deadline': ''}]},
       'count': 1},
      'refresh_rate': '60',
      'time': '105.51381111145ms',
      'xml:lang': 'en-US',
      'yahoo:uri': '/fantasy/v2/leagues;league_keys=nfl.l.427049'}}


