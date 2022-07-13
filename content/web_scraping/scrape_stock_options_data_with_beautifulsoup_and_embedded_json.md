# Scrape Stock Options Data with BeautifulSoup and Embedded JSON
In this tutorial, I will demonstrate how to scrape stock options data for a particular stock from Yahoo! Finance, for the upcoming options expiration date. The options data is loaded dynamically via JavaScript. There are two possible ways to get the data:
1. Inspecting the page to determine the URL endpoint from XHR requests that are loading the data. Then we can copy the cURL command and convert it to python using https://curlconverter.com/. 
2. Using BeautifulSoup to parse the embedded JSON dictionary containing the options data, which is stored in a `<script>` tag inside the HTML. This will enable us to easily parse the JSON directly without executing any elaborate code to emulate XHR requests.
Method 1 is not always reliable and for this particular case, method 2 is a more robust option. I will use that in this tutorial. First, let's start by creating a basic utility function to get a BeautifulSoup object.


```python
import re
import json
import requests
import pandas as pd
from bs4 import BeautifulSoup
from fake_useragent import UserAgent

def get_soup(url, headers):
    # Get response
    response = requests.get(url, headers=headers)

    # Initialize soup object
    soup = BeautifulSoup(response.content, "html.parser")

    # Return soup object
    return soup
```

Now, let's take a look at the Yahoo! Finance options page for a particular stock. In this case, we'll choose ticker symbol TSLA (Tesla): 

https://finance.yahoo.com/quote/TSLA/options?p=TSLA

![scrape-options-1.png](attachment:scrape-options-1.png)

If we right-click and select "View Page Source," we can inspect the source code to see if the JSON data is stored within a `<script>` tag. After some inspection, we can see that the options data is embedded within the `<script>` tag highlighted below, on line 238, after `root.App.main =`. We'll combine the use of BeautifulSoup with regular expressions to parse the contents of this `<script>` element.

![scrape-options-2.png](attachment:scrape-options-2.png)

Now let's write a new function called `scrape_options_data()` to get the contents of the `<script>` tag and parse the JSON that it contains. The options data is located in a nested JSON dictionary within this `<script>` element.


```python
def scrape_options_data(symbol):
    print("Scraping options data for: {}".format(symbol))

    # Set Yahoo! Finance URL
    url = "https://finance.yahoo.com/quote/{}/options?p={}".format(symbol, symbol)

    # Get random user agent
    ua = UserAgent()
    user_agent = ua.random

    # Set headers with random user agent
    headers = {
        'User-Agent': user_agent,
        'Accept': '*/*',
        'Accept-Language': 'en-US,en;q=0.5',
        'Accept-Encoding': 'gzip, deflate, br',
        'Referer': 'https://finance.yahoo.com/quote/{}/options?p={}'.format(symbol, symbol),
        'Origin': 'https://finance.yahoo.com',
        'Connection': 'keep-alive',
        'Sec-Fetch-Dest': 'empty',
        'Sec-Fetch-Mode': 'cors',
        'Sec-Fetch-Site': 'same-site',
    }

    # Get BeautifulSoup object
    soup = get_soup(url, headers)
    
    # Find desired <script> tag containing "root.App.main" text
    script = soup.find("script", text=re.compile("root.App.main")).text
    
    # Search for JSON dictionary using regex and capture groups
    data = json.loads(re.search("root.App.main\\s+=\\s+(\\{.*\\})", script).group(1))
    
    # Get data in "context"->"dispatcher"->"stores"
    stores = data["context"]["dispatcher"]["stores"]
    
    # Get calls data in "OptionContractsStore"->"contracts"->"calls"
    calls_data = stores["OptionContractsStore"]["contracts"]["calls"]

    # Get puts data in "OptionContractsStore"->"contracts"->"puts"
    puts_data = stores["OptionContractsStore"]["contracts"]["puts"]
    
    # Normalize JSON and save to DataFrame
    calls = pd.json_normalize(calls_data, max_level=1)
    puts = pd.json_normalize(puts_data, max_level=1)
    
    # Write DataFrame to CSV file
    calls.to_csv("calls-data-{}.csv".format(symbol))
    puts.to_csv("puts-data-{}.csv".format(symbol))

    print("Scraping completed")
```

Some observations:
* First, we set our URL and header variables and get our BeautifulSoup object.
* Next, we use the BeautifulSoup `find()` method to locate the `<script>` tag containing the relevant data.
* We search for the `<script>` tag containing the text "root.App.main" by passing this into the `text` parameter of the `find()` method, and extract the text from the resulting BeautifulSoup object.
* Then, we use regex to specifically extract the JSON content from the text of the `<script>` object.
    * The `re.search()` function enables us to search for the the required content and extract it using capture groups.
    * Our data is located in the second capture group (capture group 1), so we select it using `group(1)` on the match object resulting from `re.search()`.
    * We feed this into the `json.loads()` function to extract the JSON dictionary and save it as the variable `data`.
* We extract the nested dictionary containing the relevant data, which is located under the key sequence "context"->"dispatcher"->"stores."
    * We still need to find our required field, the options data, which is a nested JSON dictionary located further down under the key sequence "OptionContractsStore"->"contracts"->"calls."
* Finally, we use the pandas `json_normalize()` function to normalize the JSON, since it contains multiple-key sequences for each variable of options data, and save it to a pandas DataFrame.
    * The resulting DataFrame is written to a CSV file.

Now, we can run our program to scrape the data.


```python
symbol = "TSLA"
scrape_options_data(symbol)
```

    Scraping options data for: TSLA
    Scraping completed


Let's take a look at the output of our CSV file to see if we got the correct data.


```python
calls = pd.read_csv("calls-data-TSLA.csv")
puts = pd.read_csv("puts-data-TSLA.csv")
calls = calls.iloc[:, 1:]
puts = puts.iloc[:, 1:]
```


```python
len(calls)
```




    210




```python
len(puts)
```




    202




```python
calls.dtypes
```




    contractSymbol            object
    currency                  object
    contractSize              object
    inTheMoney                  bool
    impliedVolatility.raw    float64
    impliedVolatility.fmt     object
    expiration.raw             int64
    expiration.fmt            object
    expiration.longFmt        object
    change.raw                 int64
    change.fmt               float64
    strike.raw               float64
    strike.fmt                object
    lastPrice.raw            float64
    lastPrice.fmt            float64
    openInterest.raw           int64
    openInterest.fmt           int64
    openInterest.longFmt       int64
    percentChange.raw          int64
    percentChange.fmt         object
    ask.raw                  float64
    ask.fmt                  float64
    volume.raw               float64
    volume.fmt                object
    volume.longFmt            object
    lastTradeDate.raw          int64
    lastTradeDate.fmt         object
    lastTradeDate.longFmt     object
    bid.raw                  float64
    bid.fmt                  float64
    dtype: object




```python
puts.dtypes
```




    contractSymbol            object
    currency                  object
    contractSize              object
    inTheMoney                  bool
    impliedVolatility.raw    float64
    impliedVolatility.fmt     object
    expiration.raw             int64
    expiration.fmt            object
    expiration.longFmt        object
    change.raw                 int64
    change.fmt               float64
    strike.raw               float64
    strike.fmt                object
    lastPrice.raw            float64
    lastPrice.fmt             object
    openInterest.raw           int64
    openInterest.fmt           int64
    openInterest.longFmt       int64
    percentChange.raw          int64
    percentChange.fmt         object
    ask.raw                  float64
    ask.fmt                   object
    volume.raw               float64
    volume.fmt                object
    volume.longFmt            object
    lastTradeDate.raw          int64
    lastTradeDate.fmt         object
    lastTradeDate.longFmt     object
    bid.raw                  float64
    bid.fmt                   object
    dtype: object




```python
calls.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>contractSymbol</th>
      <th>currency</th>
      <th>contractSize</th>
      <th>inTheMoney</th>
      <th>impliedVolatility.raw</th>
      <th>impliedVolatility.fmt</th>
      <th>expiration.raw</th>
      <th>expiration.fmt</th>
      <th>expiration.longFmt</th>
      <th>change.raw</th>
      <th>...</th>
      <th>ask.raw</th>
      <th>ask.fmt</th>
      <th>volume.raw</th>
      <th>volume.fmt</th>
      <th>volume.longFmt</th>
      <th>lastTradeDate.raw</th>
      <th>lastTradeDate.fmt</th>
      <th>lastTradeDate.longFmt</th>
      <th>bid.raw</th>
      <th>bid.fmt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>TSLA220715C00100000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>0.00001</td>
      <td>0.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5.0</td>
      <td>5</td>
      <td>5</td>
      <td>1657202474</td>
      <td>2022-07-07</td>
      <td>2022-07-07T14:01</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TSLA220715C00120000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>0.00001</td>
      <td>0.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1</td>
      <td>1</td>
      <td>1656338708</td>
      <td>2022-06-27</td>
      <td>2022-06-27T14:05</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>TSLA220715C00140000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>0.00001</td>
      <td>0.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1655906547</td>
      <td>2022-06-22</td>
      <td>2022-06-22T14:02</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>TSLA220715C00160000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>0.00001</td>
      <td>0.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1656356582</td>
      <td>2022-06-27</td>
      <td>2022-06-27T19:03</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>TSLA220715C00180000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>0.00001</td>
      <td>0.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1</td>
      <td>1</td>
      <td>1657290987</td>
      <td>2022-07-08</td>
      <td>2022-07-08T14:36</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 30 columns</p>
</div>




```python
puts.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>contractSymbol</th>
      <th>currency</th>
      <th>contractSize</th>
      <th>inTheMoney</th>
      <th>impliedVolatility.raw</th>
      <th>impliedVolatility.fmt</th>
      <th>expiration.raw</th>
      <th>expiration.fmt</th>
      <th>expiration.longFmt</th>
      <th>change.raw</th>
      <th>...</th>
      <th>ask.raw</th>
      <th>ask.fmt</th>
      <th>volume.raw</th>
      <th>volume.fmt</th>
      <th>volume.longFmt</th>
      <th>lastTradeDate.raw</th>
      <th>lastTradeDate.fmt</th>
      <th>lastTradeDate.longFmt</th>
      <th>bid.raw</th>
      <th>bid.fmt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>TSLA220715P00100000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>63.0</td>
      <td>63</td>
      <td>63</td>
      <td>1657555076</td>
      <td>2022-07-11</td>
      <td>2022-07-11T15:57</td>
      <td>0.0</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TSLA220715P00120000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>25.0</td>
      <td>25</td>
      <td>25</td>
      <td>1657569476</td>
      <td>2022-07-11</td>
      <td>2022-07-11T19:57</td>
      <td>0.0</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>TSLA220715P00140000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>3.0</td>
      <td>3</td>
      <td>3</td>
      <td>1657546826</td>
      <td>2022-07-11</td>
      <td>2022-07-11T13:40</td>
      <td>0.0</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>TSLA220715P00160000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>376.0</td>
      <td>376</td>
      <td>376</td>
      <td>1657131317</td>
      <td>2022-07-06</td>
      <td>2022-07-06T18:15</td>
      <td>0.0</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>TSLA220715P00180000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>223.0</td>
      <td>223</td>
      <td>223</td>
      <td>1657555115</td>
      <td>2022-07-11</td>
      <td>2022-07-11T15:58</td>
      <td>0.0</td>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 30 columns</p>
</div>




```python
calls.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>contractSymbol</th>
      <th>currency</th>
      <th>contractSize</th>
      <th>inTheMoney</th>
      <th>impliedVolatility.raw</th>
      <th>impliedVolatility.fmt</th>
      <th>expiration.raw</th>
      <th>expiration.fmt</th>
      <th>expiration.longFmt</th>
      <th>change.raw</th>
      <th>...</th>
      <th>ask.raw</th>
      <th>ask.fmt</th>
      <th>volume.raw</th>
      <th>volume.fmt</th>
      <th>volume.longFmt</th>
      <th>lastTradeDate.raw</th>
      <th>lastTradeDate.fmt</th>
      <th>lastTradeDate.longFmt</th>
      <th>bid.raw</th>
      <th>bid.fmt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>205</th>
      <td>TSLA220715C02200000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>500.0</td>
      <td>500</td>
      <td>500</td>
      <td>1657040981</td>
      <td>2022-07-05</td>
      <td>2022-07-05T17:09</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>206</th>
      <td>TSLA220715C02250000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>2</td>
      <td>2</td>
      <td>1656340712</td>
      <td>2022-06-27</td>
      <td>2022-06-27T14:38</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>207</th>
      <td>TSLA220715C02300000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>3</td>
      <td>3</td>
      <td>1657308420</td>
      <td>2022-07-08</td>
      <td>2022-07-08T19:27</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>208</th>
      <td>TSLA220715C02350000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1</td>
      <td>1</td>
      <td>1657546205</td>
      <td>2022-07-11</td>
      <td>2022-07-11T13:30</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>209</th>
      <td>TSLA220715C02400000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>False</td>
      <td>0.500005</td>
      <td>50.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>2</td>
      <td>2</td>
      <td>1657546201</td>
      <td>2022-07-11</td>
      <td>2022-07-11T13:30</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 30 columns</p>
</div>




```python
puts.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>contractSymbol</th>
      <th>currency</th>
      <th>contractSize</th>
      <th>inTheMoney</th>
      <th>impliedVolatility.raw</th>
      <th>impliedVolatility.fmt</th>
      <th>expiration.raw</th>
      <th>expiration.fmt</th>
      <th>expiration.longFmt</th>
      <th>change.raw</th>
      <th>...</th>
      <th>ask.raw</th>
      <th>ask.fmt</th>
      <th>volume.raw</th>
      <th>volume.fmt</th>
      <th>volume.longFmt</th>
      <th>lastTradeDate.raw</th>
      <th>lastTradeDate.fmt</th>
      <th>lastTradeDate.longFmt</th>
      <th>bid.raw</th>
      <th>bid.fmt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>197</th>
      <td>TSLA220715P02125000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>0.000010</td>
      <td>0.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>1080.30</td>
      <td>1,080.30</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1639371626</td>
      <td>2021-12-13</td>
      <td>2021-12-13T05:00</td>
      <td>1070.80</td>
      <td>1,070.80</td>
    </tr>
    <tr>
      <th>198</th>
      <td>TSLA220715P02150000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>0.000010</td>
      <td>0.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>3.0</td>
      <td>3</td>
      <td>3</td>
      <td>1656599630</td>
      <td>2022-06-30</td>
      <td>2022-06-30T14:33</td>
      <td>0.00</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>199</th>
      <td>TSLA220715P02300000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>0.000010</td>
      <td>0.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>1541.65</td>
      <td>1,541.65</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1643121167</td>
      <td>2022-01-25</td>
      <td>2022-01-25T14:32</td>
      <td>1531.45</td>
      <td>1,531.45</td>
    </tr>
    <tr>
      <th>200</th>
      <td>TSLA220715P02350000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>0.000010</td>
      <td>0.00%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>1450.15</td>
      <td>1,450.15</td>
      <td>40.0</td>
      <td>40</td>
      <td>40</td>
      <td>1648474473</td>
      <td>2022-03-28</td>
      <td>2022-03-28T13:34</td>
      <td>1444.90</td>
      <td>1,444.90</td>
    </tr>
    <tr>
      <th>201</th>
      <td>TSLA220715P02400000</td>
      <td>USD</td>
      <td>REGULAR</td>
      <td>True</td>
      <td>9.079838</td>
      <td>907.98%</td>
      <td>1657843200</td>
      <td>2022-07-15</td>
      <td>2022-07-15T00:00</td>
      <td>0</td>
      <td>...</td>
      <td>1751.10</td>
      <td>1,751.10</td>
      <td>4.0</td>
      <td>4</td>
      <td>4</td>
      <td>1652976682</td>
      <td>2022-05-19</td>
      <td>2022-05-19T16:11</td>
      <td>1748.70</td>
      <td>1,748.70</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 30 columns</p>
</div>



We can see that the options data has been correctly scraped and saved to the CSV files. The `json_normalize()` function enabled us to flatten the JSON dictionary and create additional columns with the appropriate key appended to the variable name as a suffix following a period.
