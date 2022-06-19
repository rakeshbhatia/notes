# Scrape Historical OHLC (Open-High-Low-Close) Stock Prices

## Overview
* Create a basic web scraper to extract historical OHLC stock price data for a given ticker symbol
* Use the Alpha Vantage API to obtain the stock price data
* Alpha Vantage API provides last two years' worth of data
* Save the data to a pandas dataframe

### Obtain a unique API key here: https://www.alphavantage.co/support/#api-key
### Import libraries


```python
import io
import os
import sys
import math
import csv
import time
import itertools
import bisect
import string
import requests
import random
import requests
import json
import numpy
import pandas as pd
import alpha_vantage
import bs4
from bs4 import BeautifulSoup
```

### Define stock scraper method
* Use requests.get() with the Alpha Vantage API url in order to obtain response data
* Add appropriate time interval to base url
    * DAILY
    * WEEKLY
    * MONTHLY
* Add desired stock symbol to base url
* Add unique API key to base url


```python
def ohlc_stock_price_scraper(symbol, interval, api_key):
    # Use GET request to access data through API, using custom API key and desired symbol/interval options
    data = requests.get('https://www.alphavantage.co/query?function=TIME_SERIES_' + interval + '&symbol=' + symbol + '&apikey=' + api_key + '=csv')
    
    return data
```

### Test scraper method
* Test symbol: AAPL
* Test interval: DAILY


```python
data = ohlc_stock_price_scraper('AAPL', 'DAILY', 'WTX6IKTWWR57LOIQ&datatype')
```

### Store raw data in new variable
* Store data in pandas dataframe with read_csv() method
* Decode data content using 'utf-8' decoding
* Pass decoded content into io.StringIO() method before passing into read_csv


```python
raw_data = pd.read_csv(io.StringIO(data.content.decode('utf-8')))
```

### View the first 10 rows of data


```python
raw_data.head(10)
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
      <th>timestamp</th>
      <th>open</th>
      <th>high</th>
      <th>low</th>
      <th>close</th>
      <th>volume</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019-04-05</td>
      <td>196.450</td>
      <td>197.100</td>
      <td>195.93</td>
      <td>197.00</td>
      <td>18472107</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-04-04</td>
      <td>194.790</td>
      <td>196.370</td>
      <td>193.14</td>
      <td>195.69</td>
      <td>19114275</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-04-03</td>
      <td>193.250</td>
      <td>196.500</td>
      <td>193.15</td>
      <td>195.35</td>
      <td>23271830</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019-04-02</td>
      <td>191.090</td>
      <td>194.460</td>
      <td>191.05</td>
      <td>194.02</td>
      <td>22765732</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-04-01</td>
      <td>191.640</td>
      <td>191.680</td>
      <td>188.38</td>
      <td>191.24</td>
      <td>27861964</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2019-03-29</td>
      <td>189.830</td>
      <td>190.080</td>
      <td>188.54</td>
      <td>189.95</td>
      <td>23563961</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2019-03-28</td>
      <td>188.950</td>
      <td>189.559</td>
      <td>187.53</td>
      <td>188.72</td>
      <td>20780363</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2019-03-27</td>
      <td>188.750</td>
      <td>189.760</td>
      <td>186.55</td>
      <td>188.47</td>
      <td>29848427</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2019-03-26</td>
      <td>191.664</td>
      <td>192.880</td>
      <td>184.58</td>
      <td>186.79</td>
      <td>49800538</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2019-03-25</td>
      <td>191.510</td>
      <td>191.980</td>
      <td>186.60</td>
      <td>188.74</td>
      <td>43845293</td>
    </tr>
  </tbody>
</table>
</div>


