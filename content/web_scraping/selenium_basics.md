# Selenium Basics
* Selenium is a tool for controlling web browsers and performing browser automation
* It is often used to automate testing of websites and simulate a real user's activity
* The Selenium Python bindings enables us to perform actions on a webpage using the Selenium WebDriver
    * We can use this to scrape dynamic (JavaScript) content
* To demonstrate the basics of Selenium, we will search for and download some images from Google


```python
import time
import pandas as pd
import requests
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

query = 'Mexico'

driver = webdriver.Chrome('./chromedriver')
url = 'https://www.google.com/imghp?hl=en'
driver.get(url)
print(driver.title)

# //input[@id=”search”]

search_bar = driver.find_element_by_xpath('//input[@aria-label="Search"]')

#search_bar = driver.find_element_by_class_name('gLFyf gsfi')

#search_bar = driver.find_element_by_xpath('/html/body/div[1]/div[3]/form/div[1]/div[1]/div[1]/div/div[2]/input')

# /html/body/div[1]/div[3]/form/div[1]/div[1]/div[1]/div/div[2]/input

search_bar.send_keys(query)

search_bar.send_keys(Keys.RETURN)

time.sleep(3)

#img_link = driver.find_element_by_xpath('//a[@jsname="sTFXNd"]')

#img_link = driver.find_element_by_class_name('wXeWr islib nfEiy')

#img_link = driver.find_element_by_xpath('/html/body/div[2]/c-wiz/div[3]/div[1]/div/div/div/div[1]/div[1]/span/div[1]/div[1]/div[1]/a[1]')

# /html/body/div[2]/c-wiz/div[3]/div[1]/div/div/div/div[1]/div[1]/span/div[1]/div[1]/div[1]/a[1]

#img_link.click()

img = driver.find_element_by_xpath('//a[@jsname="Q4LuWd"]')

img.screenshot(query + '-1.png')

time.sleep(3)

driver.quit()

#element = driver.find_element_by_xpath('//input[@id="passwd-id"]')

#cuisine_selection = driver.find_element_by_xpath('/html/body/div[1]/div[1]/div/main/div/div[1]/div[1]/nav/ul/li[13]/a')
```

    Google Images


## Instantiate the webdriver


```python
driver = webdriver.Firefox()
url = 'https://www.ubereats.com'
driver.get(url)
print(driver.title)
```


```python
# /html/body/div[1]/div[1]/div/main/div/div[1]/div[1]/nav/ul/li[13]/a/div[1]/img
# /html/body/div[1]/div[1]/div/main/div/div[1]/div[1]/nav/ul/li[13]/a
# .gLFyf
# /html/body/div[2]/c-wiz/div[3]/div[1]/div/div/div/div[1]/div[1]/span/div[1]/div[1]/div[1]/a[1]
```


```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time

WEBDRIVER_PATH = './'

# Initialize the web driver
driver = webdriver.Chrome(WEBDRIVER_PATH)
URL = 'https://www.youtube.com'
driver.get(URL)
print(driver.title)

search_box = driver.find_element_by_xpath('//input[@id="search"]')

search_box.send_keys('Selenium')

search_box.send_keys(Keys.ENTER)

time.sleep(5)

videos = driver.find_elements_by_xpath('//*[@id="dismissable"]')

print(len(videos))

for video in videos:
    title = video.find_element_by_xpath('.//*[@id="video-title"]')
    print(title.text)

driver.quit()
```

    YouTube
    0



```python
# XPath for YouTube search bar: //*[@id="search"]
```
