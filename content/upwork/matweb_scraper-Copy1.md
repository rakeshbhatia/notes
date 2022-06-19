```python
import csv
import time
import string
import urllib
import requests
import lxml.html as lh
import json
import numpy
import pandas as pd
import bs4
from bs4 import BeautifulSoup
from requests_html import HTMLSession
import nest_asyncio
nest_asyncio.apply()
#if asyncio.get_event_loop().is_running(): # Only patch if needed (i.e. running in Notebook, Spyder, etc)
#    import nest_asyncio
#    nest_asyncio.apply()
```


```python
# Instantiate session
session = HTMLSession()

# Store the url in a variable
url = "http://www.matweb.com/search/PropertySearch.aspx"

# Get the site content using requests
r = session.get(url)

# Execute js in the page
r.html.render()

# Extract text from the content
content = r.text

# Convert html text content into a beautiful soup object
soup = BeautifulSoup(content, "lxml")
```


    ---------------------------------------------------------------------------

    RuntimeError                              Traceback (most recent call last)

    <ipython-input-4-7a7d62818827> in <module>
          9 
         10 # Execute js in the page
    ---> 11 r.html.render()
         12 
         13 # Extract text from the content


    ~/anaconda/lib/python3.6/site-packages/requests_html.py in render(self, retries, script, wait, scrolldown, sleep, reload, timeout, keep_page)
        584         """
        585 
    --> 586         self.browser = self.session.browser  # Automatically create a event loop and browser
        587         content = None
        588 


    ~/anaconda/lib/python3.6/site-packages/requests_html.py in browser(self)
        727             self.loop = asyncio.get_event_loop()
        728             if self.loop.is_running():
    --> 729                 raise RuntimeError("Cannot use HTMLSession within an existing event loop. Use AsyncHTMLSession instead.")
        730             self._browser = self.loop.run_until_complete(super().browser)
        731         return self._browser


    RuntimeError: Cannot use HTMLSession within an existing event loop. Use AsyncHTMLSession instead.



```python
table = soup.find("table", {"class": "_tableborder"})
```


```python
td = table.findAll("td")
```


```python
metal_table = td[0].findAll("table")[3]
```


```python
metal_table
```


```python
js_link = metal_table.select("a[href^='javascript:']")[0]
```


```python
js_link["href"]
```




    "javascript:TreeView_PopulateNode(ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeView_Data,3,document.getElementById('ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewn3'),document.getElementById('ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewt3'),null,'t','Metal (17052 matls)','9','f','','f')"




```python
r = session.get("http://" + js_link["href"])
```


    ---------------------------------------------------------------------------

    LocationParseError                        Traceback (most recent call last)

    ~/anaconda/lib/python3.6/site-packages/requests/models.py in prepare_url(self, url, params)
        378         try:
    --> 379             scheme, auth, host, port, path, query, fragment = parse_url(url)
        380         except LocationParseError as e:


    ~/anaconda/lib/python3.6/site-packages/urllib3/util/url.py in parse_url(url)
        400     except (ValueError, AttributeError):
    --> 401         return six.raise_from(LocationParseError(source_url), None)
        402 


    ~/anaconda/lib/python3.6/site-packages/urllib3/packages/six.py in raise_from(value, from_value)


    LocationParseError: Failed to parse: http://javascript:TreeView_PopulateNode(ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeView_Data,3,document.getElementById('ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewn3'),document.getElementById('ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewt3'),null,'t','Metal (17052 matls)','9','f','','f')

    
    During handling of the above exception, another exception occurred:


    InvalidURL                                Traceback (most recent call last)

    <ipython-input-90-f0d0e44cb9cc> in <module>
    ----> 1 r = session.get("http://" + js_link["href"])
    

    ~/anaconda/lib/python3.6/site-packages/requests/sessions.py in get(self, url, **kwargs)
        544 
        545         kwargs.setdefault('allow_redirects', True)
    --> 546         return self.request('GET', url, **kwargs)
        547 
        548     def options(self, url, **kwargs):


    ~/anaconda/lib/python3.6/site-packages/requests/sessions.py in request(self, method, url, params, data, headers, cookies, files, auth, timeout, allow_redirects, proxies, hooks, stream, verify, cert, json)
        517             hooks=hooks,
        518         )
    --> 519         prep = self.prepare_request(req)
        520 
        521         proxies = proxies or {}


    ~/anaconda/lib/python3.6/site-packages/requests/sessions.py in prepare_request(self, request)
        460             auth=merge_setting(auth, self.auth),
        461             cookies=merged_cookies,
    --> 462             hooks=merge_hooks(request.hooks, self.hooks),
        463         )
        464         return p


    ~/anaconda/lib/python3.6/site-packages/requests/models.py in prepare(self, method, url, headers, files, data, params, auth, cookies, hooks, json)
        311 
        312         self.prepare_method(method)
    --> 313         self.prepare_url(url, params)
        314         self.prepare_headers(headers)
        315         self.prepare_cookies(cookies)


    ~/anaconda/lib/python3.6/site-packages/requests/models.py in prepare_url(self, url, params)
        379             scheme, auth, host, port, path, query, fragment = parse_url(url)
        380         except LocationParseError as e:
    --> 381             raise InvalidURL(*e.args)
        382 
        383         if not scheme:


    InvalidURL: Failed to parse: http://javascript:TreeView_PopulateNode(ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeView_Data,3,document.getElementById('ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewn3'),document.getElementById('ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewt3'),null,'t','Metal (17052 matls)','9','f','','f')



```python
tr = material_tables[4].findAll("tr")[1]
```


```python
metals_expand = urllib.request.urlopen(js_link[0])
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    <ipython-input-89-87818f944678> in <module>
    ----> 1 metals_expand = urllib.request.urlopen(js_link[0])
    

    ~/anaconda/lib/python3.6/site-packages/bs4/element.py in __getitem__(self, key)
        969         """tag[key] returns the value of the 'key' attribute for the tag,
        970         and throws an exception if it's not there."""
    --> 971         return self.attrs[key]
        972 
        973     def __iter__(self):


    KeyError: 0



```python

```


```python

```


```python

```


```python

```


```python

```


```python
dropdown1 = soup.find("selected", {"name": "ctl00$ContentMain$ucPropertyDropdown1$drpPropertyList"}).findAll("option")
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-14-d8090156029c> in <module>
    ----> 1 dropdown1 = soup.find("selected", {"name": "ctl00$ContentMain$ucPropertyDropdown1$drpPropertyList"}).findAll("option")
    

    AttributeError: 'NoneType' object has no attribute 'findAll'



```python
print(dropdown1)
```
