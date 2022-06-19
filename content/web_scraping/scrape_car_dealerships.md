```python
import time
import pandas as pd
import requests
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

url = 'https://www.carsforsale.com/car-dealers'

driver = webdriver.Chrome('./chromedriver')
driver.get(url)
print(driver.title)
driver.quit()
```

    []



```python

```
