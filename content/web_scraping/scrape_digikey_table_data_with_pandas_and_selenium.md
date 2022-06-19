# Scrape Digi-Key Table Data with pandas and Selenium
* In this project we will demonstrate the use of the pandas `read_html()` method to extract table data from the Digi-Key website, which is a massive database of electronic components
* We will also use Selenium to extract a specialized piece of data which cannot be rendered by `read_html()`
* We will extract table data from the following webpage: https://www.digikey.com/en/products/filter/accessories/159
* Scrolling down, we can see that the table appears as below:

![digikey-table.png](attachment:digikey-table.png)

## Import required libraries


```python
import time
import pandas as pd
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.options import Options
from selenium.common.exceptions import NoSuchElementException
```

## Read in all tables in the url
* The `read_html()` method will read in all tables from the website's html as a list


```python
url = 'https://www.digikey.com/en/products/filter/accessories/159'

table_dk = pd.read_html(url)

print(f'Total tables: {len(table_dk)}')
```


    ---------------------------------------------------------------------------

    HTTPError                                 Traceback (most recent call last)

    /var/folders/r1/hrhfwyd51yn0ngwzdwr7h7v00000gn/T/ipykernel_6023/811688384.py in <module>
          1 url = 'https://www.digikey.com/en/products/filter/accessories/159'
          2 
    ----> 3 table_dk = pd.read_html(url)
          4 
          5 print(f'Total tables: {len(table_dk)}')


    ~/opt/anaconda3/lib/python3.9/site-packages/pandas/util/_decorators.py in wrapper(*args, **kwargs)
        309                     stacklevel=stacklevel,
        310                 )
    --> 311             return func(*args, **kwargs)
        312 
        313         return wrapper


    ~/opt/anaconda3/lib/python3.9/site-packages/pandas/io/html.py in read_html(io, match, flavor, header, index_col, skiprows, attrs, parse_dates, thousands, encoding, decimal, converters, na_values, keep_default_na, displayed_only)
       1096     io = stringify_path(io)
       1097 
    -> 1098     return _parse(
       1099         flavor=flavor,
       1100         io=io,


    ~/opt/anaconda3/lib/python3.9/site-packages/pandas/io/html.py in _parse(flavor, io, match, attrs, encoding, displayed_only, **kwargs)
        904 
        905         try:
    --> 906             tables = p.parse_tables()
        907         except ValueError as caught:
        908             # if `io` is an io-like object, check if it's seekable


    ~/opt/anaconda3/lib/python3.9/site-packages/pandas/io/html.py in parse_tables(self)
        220         list of parsed (header, body, footer) tuples from tables.
        221         """
    --> 222         tables = self._parse_tables(self._build_doc(), self.match, self.attrs)
        223         return (self._parse_thead_tbody_tfoot(table) for table in tables)
        224 


    ~/opt/anaconda3/lib/python3.9/site-packages/pandas/io/html.py in _build_doc(self)
        743                     pass
        744             else:
    --> 745                 raise e
        746         else:
        747             if not hasattr(r, "text_content"):


    ~/opt/anaconda3/lib/python3.9/site-packages/pandas/io/html.py in _build_doc(self)
        724         try:
        725             if is_url(self.io):
    --> 726                 with urlopen(self.io) as f:
        727                     r = parse(f, parser=parser)
        728             else:


    ~/opt/anaconda3/lib/python3.9/site-packages/pandas/io/common.py in urlopen(*args, **kwargs)
        210     import urllib.request
        211 
    --> 212     return urllib.request.urlopen(*args, **kwargs)
        213 
        214 


    ~/opt/anaconda3/lib/python3.9/urllib/request.py in urlopen(url, data, timeout, cafile, capath, cadefault, context)
        212     else:
        213         opener = _opener
    --> 214     return opener.open(url, data, timeout)
        215 
        216 def install_opener(opener):


    ~/opt/anaconda3/lib/python3.9/urllib/request.py in open(self, fullurl, data, timeout)
        521         for processor in self.process_response.get(protocol, []):
        522             meth = getattr(processor, meth_name)
    --> 523             response = meth(req, response)
        524 
        525         return response


    ~/opt/anaconda3/lib/python3.9/urllib/request.py in http_response(self, request, response)
        630         # request was successfully received, understood, and accepted.
        631         if not (200 <= code < 300):
    --> 632             response = self.parent.error(
        633                 'http', request, response, code, msg, hdrs)
        634 


    ~/opt/anaconda3/lib/python3.9/urllib/request.py in error(self, proto, *args)
        559         if http_err:
        560             args = (dict, 'default', 'http_error_default') + orig_args
    --> 561             return self._call_chain(*args)
        562 
        563 # XXX probably also want an abstract factory that knows when it makes


    ~/opt/anaconda3/lib/python3.9/urllib/request.py in _call_chain(self, chain, kind, meth_name, *args)
        492         for handler in handlers:
        493             func = getattr(handler, meth_name)
    --> 494             result = func(*args)
        495             if result is not None:
        496                 return result


    ~/opt/anaconda3/lib/python3.9/urllib/request.py in http_error_default(self, req, fp, code, msg, hdrs)
        639 class HTTPDefaultErrorHandler(BaseHandler):
        640     def http_error_default(self, req, fp, code, msg, hdrs):
    --> 641         raise HTTPError(req.full_url, code, msg, hdrs, fp)
        642 
        643 class HTTPRedirectHandler(BaseHandler):


    HTTPError: HTTP Error 403: Forbidden


## Initialize new pandas DataFrame to store the table data
* The length of `table_dk` is one, so only one table was extracted from the html
* We will initialize a new pandas DataFrame and set it equal to the first element of the list


```python
df = table_dk[0]

df = df.iloc[1:]

df.head()
```

## Remove unneeded column
* We don't need the column 'Compare' since it doesn't hold any relevant data
    * The column can be removed using the `drop()` method
    * We specify `axis=1` to indicate that we want to remove a column and not a row


```python
df.drop('Compare', inplace=True, axis=1)

df.head()
```

## Extract specialized link data not captured by `read_html()`
* We want to extract the link to the part data, which is also stored in the table
    * `read_html()` only extracts the text content, so we have to extract the link using another method (Selenium)
* First we need the XPath for each row element of the table, as well as the XPath for the `<a>` tag containing the link data

## Instantiate WebDriver


```python
chrome_options = Options()  
chrome_options.add_argument('--headless')
chrome_options.add_argument('--window-size=1920x1080')
driver = webdriver.Chrome(executable_path='./chromedriver', options=chrome_options)
driver.get(url)

print(driver.title)
```

## Determine number of rows in table
* The total number of rows should be equal to 25 in this case


```python
time.sleep(3)

tr_xpath = '/html/body/div[2]/main/section/div/div[2]/div/div[2]/div/div[1]/table/tbody/tr'

rows = 1+len(driver.find_elements_by_xpath(tr_xpath))

print(rows)
```

## Loop through table and extract link from each row
* We will use the `find_element_by_xpath` method to get the `<a>` tag
* Then we will use the Selenium `get_attribute` method to extract the data stored in the `href` attribute
* We will store our extracted links in a list and add these to the original DataFrame


```python
links = []

for i in range(1, rows):
    try:
        elem = driver.find_element_by_xpath(tr_xpath+'['+str(i)+']/td[2]/div/div[3]/div[1]/a')
        link = elem.get_attribute('href')
        links.append(link)
    except NoSuchElementException:
        pass

df['Link'] = links

df.head()
```

## Close the WebDriver


```python
driver.quit()
```

## Complete scraper


```python
import time
import pandas as pd
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.options import Options
from selenium.common.exceptions import NoSuchElementException

url = 'https://www.digikey.com/en/products/filter/accessories/159'

table_dk = pd.read_html(url)

print(f'Total tables: {len(table_dk)}')

df = table_dk[0]

df = df.iloc[1:]

df.drop('Compare', inplace=True, axis=1)

chrome_options = Options()  
chrome_options.add_argument('--headless')
chrome_options.add_argument('--window-size=1920x1080')
driver = webdriver.Chrome(executable_path='./chromedriver', options=chrome_options)
driver.get(url)

print(driver.title)

time.sleep(3)

tr_xpath = '/html/body/div[2]/main/section/div/div[2]/div/div[2]/div/div[1]/table/tbody/tr'

rows = 1+len(driver.find_elements_by_xpath(tr_xpath))

print(rows)

links = []

for i in range(1, rows):
    try:
        elem = driver.find_element_by_xpath(tr_xpath+'['+str(i)+']/td[2]/div/div[3]/div[1]/a')
        link = elem.get_attribute('href')
        links.append(link)
    except NoSuchElementException:
        pass

df['Link'] = links

driver.quit()
```
