```python
from selenium import webdriver
import time
 
options = webdriver.ChromeOptions()
options.add_argument('--ignore-certificate-errors')
options.add_argument("--test-type")
options.binary_location = "/usr/bin/chromium"
driver = webdriver.Chrome(chrome_options=options)
 
driver.get('https://www.w3.org/')
time.sleep(3)
driver.quit()
```

    /Users/rakeshbhatia/anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:8: DeprecationWarning: use options instead of chrome_options
      



```python

```
