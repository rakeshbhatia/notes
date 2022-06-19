# Get Domain Information with WHOIS
In this tutorial, I will show you how to get information for any domain name or IP address using the [whois](https://github.com/richardpenman/whois) library in Python. WHOIS is a popular internet record listing that contains all contact information for the person, group, or company that registered a particular domain. The whois library simply queries the WHOIS database directly to retrieve domain information. You can install the library with the following command:

`pip install python-whois`

In a [previous tutorial](https://rakeshbhatia.github.io/notes/content/web_scraping/scrape_free_proxies_with_selenium), I demonstrated how to scrape a list of free proxies from the [spys.one](https://spys.one/en/anonymous-proxy-list/) website using Selenium. Let's use one of the proxies we scraped in this tutorial, which were saved in a CSV file. First, we'll write a function to check if a domain name is registered or not.


```python
import whois
import pandas as pd
import numpy as np

def is_registered(domain_name):
    # Return a boolean indicating whether a domain is registered
    try:
        w = whois.whois(domain_name)
    except Exception:
        return False
    else:
        return bool(w.domain_name)
```


```python
# Load proxies into DataFrame
df = pd.read_csv("spys-proxy-list-30.csv")

# Add new columns
df["Registered"] = ""
df["Registrar"] = ""
df["WHOIS server"] = ""
df["Creation date"] = ""
df["Expiration date"] = ""

# Iterate over rows
for i in range(len(df)):
    #domain_name = df.loc[i, "Proxy type"] + "://" + df.loc[i, "Proxy address"].split(":")[0]
    domain_name = df.loc[i, "Proxy address"].split(":")[0]
    # Check if proxy is registered and assign values to DataFrame accordingly
    if is_registered(domain_name):
        #print("Domain registered")
        whois_info = whois.whois(domain_name)
        df.loc[i, "Registered"] = "yes"
        df.loc[i, "Registrar"] = str(whois_info.registrar)
        #print("Domain registrar:", df.loc[i, "Registrar"])
        df.loc[i, "WHOIS server"] = str(whois_info.whois_server)
        #print("WHOIS server:", df.loc[i, "WHOIS server"])
        if type(whois_info.creation_date) == list:
            df.loc[i, "Creation date"] = whois_info.creation_date[-1].strftime("%Y-%m-%d %H:%M:%S")
        else:
            df.loc[i, "Creation date"] = str(whois_info.creation_date)
        #print("Creation date:", df.loc[i, "Creation date"])
        if type(whois_info.expiration_date) == list:
            df.loc[i, "Expiration date"] = whois_info.expiration_date[-1].strftime("%Y-%m-%d %H:%M:%S")
        else:
            df.loc[i, "Expiration date"] = str(whois_info.expiration_date)
        #print("Expiration date:", df.loc[i, "Expiration date"])
    # If domain is not registered, leave everything else blank
    else:
        #print("Domain not registered")
        df.loc[i, "Registered"] = "no"
        df.loc[i, "Registrar"] = None
        df.loc[i, "WHOIS server"] = None
        df.loc[i, "Creation date"] = None
        df.loc[i, "Expiration date"] = None

df.head()
```

    Error trying to connect to socket: closing socket





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
      <th>Proxy address</th>
      <th>Proxy type</th>
      <th>Registered</th>
      <th>Registrar</th>
      <th>WHOIS server</th>
      <th>Creation date</th>
      <th>Expiration date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>122.99.125.85:80</td>
      <td>http</td>
      <td>no</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>220.116.226.105:80</td>
      <td>http</td>
      <td>no</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103.219.194.13:80</td>
      <td>http</td>
      <td>no</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>110.170.126.13:3128</td>
      <td>https</td>
      <td>yes</td>
      <td>THNIC</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>91.150.189.122:30389</td>
      <td>http</td>
      <td>yes</td>
      <td>home.pl S.A.</td>
      <td>None</td>
      <td>2003-06-14 08:45:04</td>
      <td>2025-06-13 14:00:00</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.tail()
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
      <th>Proxy address</th>
      <th>Proxy type</th>
      <th>Registered</th>
      <th>Registrar</th>
      <th>WHOIS server</th>
      <th>Creation date</th>
      <th>Expiration date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>25</th>
      <td>47.241.245.186:80</td>
      <td>http</td>
      <td>no</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>26</th>
      <td>185.91.116.140:80</td>
      <td>http</td>
      <td>yes</td>
      <td>None</td>
      <td>None</td>
      <td>2020-06-16 00:00:00</td>
      <td>2022-11-23 00:00:00</td>
    </tr>
    <tr>
      <th>27</th>
      <td>185.72.27.98:8080</td>
      <td>http</td>
      <td>no</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>28</th>
      <td>45.77.233.110:80</td>
      <td>http</td>
      <td>yes</td>
      <td>ENOM, INC.</td>
      <td>WHOIS.ENOM.COM</td>
      <td>2022-03-10 16:58:11</td>
      <td>2027-03-10 16:58:11</td>
    </tr>
    <tr>
      <th>29</th>
      <td>80.179.140.189:80</td>
      <td>http</td>
      <td>yes</td>
      <td>Domain The Net Technologies Ltd</td>
      <td>None</td>
      <td>None</td>
      <td>2023-04-11 00:00:00</td>
    </tr>
  </tbody>
</table>
</div>



The safest proxies to use would be those that were registered a long time ago (at least 10 years is a good amount). We can filter out all the proxies that are less than 10 years old.


```python
df2 = df[df["Creation date"] < "2012"]
df2 = df2.reset_index(drop=True)
df2.head()
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
      <th>Proxy address</th>
      <th>Proxy type</th>
      <th>Registered</th>
      <th>Registrar</th>
      <th>WHOIS server</th>
      <th>Creation date</th>
      <th>Expiration date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>91.150.189.122:30389</td>
      <td>http</td>
      <td>yes</td>
      <td>home.pl S.A.</td>
      <td>None</td>
      <td>2003-06-14 08:45:04</td>
      <td>2025-06-13 14:00:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>50.233.42.98:51696</td>
      <td>http</td>
      <td>yes</td>
      <td>CSC CORPORATE DOMAINS, INC.</td>
      <td>whois.corporatedomains.com</td>
      <td>2000-07-27 17:53:12</td>
      <td>2025-07-27 17:53:12</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5.167.141.239:3128</td>
      <td>http</td>
      <td>yes</td>
      <td>RU-CENTER-RU</td>
      <td>None</td>
      <td>2001-03-13 21:00:00</td>
      <td>2023-03-14 21:00:00</td>
    </tr>
  </tbody>
</table>
</div>



We can see that only three of the proxies are more than 10 years old. Finally, let's output our filtered list to a CSV file.


```python
df2.to_csv("spys-proxy-list-30-filtered.csv")
```
