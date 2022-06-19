# Scrape Hisco Product Pages


```python
import csv
import time
import string
import requests
import json
import numpy
import pandas as pd
import scrapy
from scrapy.crawler import CrawlerProcess

# Create a new spider class
class RotoSpider(scrapy.Spider):
    # Name our spider
    name = "RS"
    
    # URL(s) to start with.
    start_urls = [
        "https://www.hisco.com/"
    ]
    
    def parse(self, response):
        for link in self.link_extractor.extract_links(response):
            yield Request("/Catalog/", callback=self.parse)

# Pass in settings
process = CrawlerProcess({
    "FEED_FORMAT": "json",                 # Save our data as json
    "FEED_URI": "hisco_category_links.json",  # Specify the json output file
    "DEPTH_LIMIT": 3,                      # Only traverse three links
    "DOWNLOAD_DELAY": 0.50,                # Set a delay of 0.5 seconds
    "LOG_ENABLED": False                   # For debugging, change this to true
})
```
