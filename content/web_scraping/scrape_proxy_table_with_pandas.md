# Scrape Proxy Table with pandas
## Overview
* We will use the pandas library to easily scrape the list of proxies at the following website:
    * https://hidemy.name/en/proxy-list/
* The `read_html()` method will be used to extract all tables from the HTML
* The website looks like this:

![proxy-table-1.png](attachment:proxy-table-1.png)

## Import libraries


```python
import requests
import pandas as pd
```

## Set user agent header and get website response
* In order to avoid getting an HTTP Error 403 (Forbidden), we need to set a user agent header
* We can then feed this header as an argument to the `get()` method from the `requests` library


```python
url = 'https://hidemy.name/en/proxy-list/'

header = {
  "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.75 Safari/537.36",
  "X-Requested-With": "XMLHttpRequest"
}

r = requests.get(url, headers=header)
```

## Get all tables from the HTML
* Now we can use `r.text` (the text of the response) as our input to the `read_html()` method
* The `read_html()` method returns a list of all tables from the HTML stored in the response `r`


```python
tables = pd.read_html(r.text)

print(f'Total tables: {len(tables)}')
```

    Total tables: 1


## Store proxy list table in pandas DataFrame
* Since there is only one table, our list `tables` will only contain one element
* We can store the table in a pandas DataFrame
* Printing `df.head()` and comparing with the screenshot above confirms that the table has been extracted successfully


```python
df = tables[0]

df.head()
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
      <th>IP address</th>
      <th>Port</th>
      <th>Country, City</th>
      <th>Speed</th>
      <th>Type</th>
      <th>Anonymity</th>
      <th>Latest update</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>180.180.170.188</td>
      <td>8080</td>
      <td>Thailand</td>
      <td>4560 ms</td>
      <td>HTTP</td>
      <td>no</td>
      <td>41 seconds</td>
    </tr>
    <tr>
      <th>1</th>
      <td>223.27.194.66</td>
      <td>63141</td>
      <td>Thailand</td>
      <td>1900 ms</td>
      <td>HTTP</td>
      <td>no</td>
      <td>43 seconds</td>
    </tr>
    <tr>
      <th>2</th>
      <td>101.51.55.153</td>
      <td>8080</td>
      <td>Thailand Don Chedi</td>
      <td>3040 ms</td>
      <td>HTTP</td>
      <td>no</td>
      <td>43 seconds</td>
    </tr>
    <tr>
      <th>3</th>
      <td>122.154.35.190</td>
      <td>8080</td>
      <td>Thailand Panare</td>
      <td>1880 ms</td>
      <td>HTTP</td>
      <td>no</td>
      <td>43 seconds</td>
    </tr>
    <tr>
      <th>4</th>
      <td>203.23.106.190</td>
      <td>80</td>
      <td>Cyprus</td>
      <td>480 ms</td>
      <td>HTTP</td>
      <td>no</td>
      <td>1 minutes</td>
    </tr>
  </tbody>
</table>
</div>


