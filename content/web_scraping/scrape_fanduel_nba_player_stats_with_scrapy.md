# Scrape Fanduel NBA Player Stats with Scrapy

## Fanduel player stats archive: 
## http://rotoguru1.com/cgi-bin/hyday.pl?mon=4&day=10&year=2019&game=fd

## Overview
* Create a simple scraper to obtain NBA player daily fantasy basketball stats from the RotoGuru site, using Scrapy
* We will focus on extracting fantasy stats for Fanduel only, but the same methodology can be used to extract stats for other platforms (i.e., Draftkings)
* The RotoGuru site uses pagination, storing data from one day's worth of games on a single page
* We will enable our crawler to identify the next page link (for the previous day's games) and recursively crawl these links until we reach the first day of the season
    * This will enable us to obtain an entire season's worth of data
* For the purposes of this demonstration, we will set a depth limit to prevent our scraper from crawling all the links in succession, which may overload the site's server
* Before defining our scraper, we must visually inspect the site's source code and identify where our target data is located
* Scrapy allows us to use CSS or XPath selectors; for this project we will use XPATH
* Navigating to the website and inspecting the source reveals that our required data is located in a table with the following XPath:
    * /html/body/table[1]//table[@cellspacing=5]
* Each row of data in this table contains the stats for a single player from that day's game
* Stats include standard NBA box score numbers, as well as total Fanduel points scored

### Import libraries


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
```

### Begin creating our scraper
* First declare the spider class
* Create a name for the spider
* Provide a list of start urls


```python
# Create a new spider class
class DFSpider(scrapy.Spider):
    # Name our spider
    name = "DFS"
    
    # URL(s) to start with.
    start_urls = [
        'http://rotoguru1.com/cgi-bin/hyday.pl?mon=4&day=10&year=2019&game=fd'
    ]
```

### Define the `parse` method
* Get all the table rows and store them in a list
* Remove the first two rows as they don't contain any player data


```python
# Create a new spider class
class DFSpider(scrapy.Spider):

    # ...

    # Use XPath to parse the response we get
    def parse(self, response):
        # Get the table rows
        trs = response.xpath('/html/body/table[1]//table[@cellspacing=5]//tr')
            
        # Remove first two rows since they don't contain any player data
        trs = trs[2:]
```

### Iterate across table rows and extract player stats
* Yield a dictionary containing our extracted data
* Use the XPath for the `td` elements located in each `tr` element of the data
* Extract only the text content from each `td` element (or from the appropriate enclosed element)


```python
# Create a new spider class
class DFSpider(scrapy.Spider):

    # ...

    # Parse the response with XPath
    def parse(self, response):

        # ...
        
        # Check if table rows exist
        if trs:        
            # Iterate over each row
            for tr in trs:
                # Yield a dictionary with our desired values
                yield {
                    # Extract each player's stats here
                    'Name': tr.xpath('./td[2]/a/text()').extract(),
                    'Position': tr.xpath('./td[1]/text()').extract(),
                    'FD Pts': tr.xpath('./td[3]/text()').extract(),
                    'FD Salary': tr.xpath('./td[4]/text()').extract(),
                    'Team': tr.xpath('./td[5]/text()').extract(),
                    'Opp': tr.xpath('./td[6]/text()').extract(),
                    'Score': tr.xpath('./td[7]/text()').extract(),
                    'Min': tr.xpath('./td[8]/text()').extract(),
                    'Stats': tr.xpath('./td[9]/text()').extract()
                }
```

### Search for the next page link
* We will call our parse function on the next page link
* In this manner, we will recursively crawl across the series of links until we either hit our depth (in settings), or reach the beginning of the archive (link to opening day's stats page)


```python
# Create a new spider class
class DFSpider(scrapy.Spider):

    # ...

    # Parse the response with XPath
    def parse(self, response):

        # ...
        
        # ...

        # Select the table containing the next page link
        table = response.xpath('//table[@border=0]')[6]

        # Get the next page link
        next_page = table.xpath('./tr[1]/td[1]/a/@href').extract_first()

        # Recursively call the parse function on the next page link
        if next_page is not None:
            print('Page completed. Going to next page.')
            yield scrapy.Request(next_page, callback=self.parse)
```

### Pass in desired settings and start the crawler
* Save the data in JSON format
* Set a download delay of 0.50 seconds to minimize server load


```python
# Create a new spider class
class DFSpider(scrapy.Spider):

    # ...

    # Parse the response with XPath
    def parse(self, response):

        # ...
        
        # ...

        # ...

# Pass in settings
process = CrawlerProcess({
    'FEED_FORMAT': 'json',                 # Save our data as json
    'FEED_URI': 'nba_fanduel_stats.json',  # Specify the json output file
    'DEPTH_LIMIT': 3,                      # Only traverse three links
    'DOWNLOAD_DELAY': 0.50,                # Set a delay of 0.5 seconds
    'LOG_ENABLED': False                   # For debugging, change this to true
})
```

### Our complete scrapy crawler


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
        'http://rotoguru1.com/cgi-bin/hyday.pl?mon=4&day=10&year=2019&game=fd'
    ]
    
    # Use XPath to parse the response we get.
    def parse(self, response):
        # Get the table rows
        trs = response.xpath('/html/body/table[1]//table[@cellspacing=5]//tr')
            
        # Remove first two rows since they don't contain any player data
        trs = trs[2:]   
        
        # Iterate over every element on the page
        if trs:
            for tr in trs:
                # Yield a dictionary with the values we want
                yield {
                    'Name': tr.xpath('./td[2]/a/text()').extract(),
                    'Position': tr.xpath('./td[1]/text()').extract(),
                    'FD Pts': tr.xpath('./td[3]/text()').extract(),
                    'FD Salary': tr.xpath('./td[4]/text()').extract(),
                    'Team': tr.xpath('./td[5]/text()').extract(),
                    'Opp': tr.xpath('./td[6]/text()').extract(),
                    'Score': tr.xpath('./td[7]/text()').extract(),
                    'Min': tr.xpath('./td[8]/text()').extract(),
                    'Stats': tr.xpath('./td[9]/text()').extract()
                }
                
        # Select the table containing the next page link
        table = response.xpath('//table[@border=0]')[6]
        
        # Get the next page link
        next_page = table.xpath('./tr[1]/td[1]/a/@href').extract_first()
        
        # Run the parse function recursively on the next page link
        if next_page is not None:
            print('Page completed. Going to next page.')
            yield scrapy.Request(next_page, callback=self.parse)
        
# Pass in settings
process = CrawlerProcess({
    'FEED_FORMAT': 'json',                 # Save our data as json
    'FEED_URI': 'nba_fanduel_stats.json',  # Specify the json output file
    'DEPTH_LIMIT': 3,                      # Only traverse three links
    'DOWNLOAD_DELAY': 0.50,                # Set a delay of 0.5 seconds
    'LOG_ENABLED': False                   # For debugging, change this to true
})
```

### Run the crawler


```python
# Start the crawler
process.crawl(RotoSpider)
process.start()
print('Scraping completed.')
```

    Page completed. Going to next page.
    Page completed. Going to next page.
    Page completed. Going to next page.
    Page completed. Going to next page.
    Scraping completed.


### Import JSON data into new pandas dataframe


```python
nba_fanduel_stats = pd.read_json('nba_fanduel_stats.json', orient='records')
print('Number of rows: {}'.format(nba_fanduel_stats.shape[0]))
print(nba_fanduel_stats.head())
```

    Number of rows: 1189
       FD Pts  FD Salary      Min                  Name      Opp Position  \
    0  [58.7]   [$3,500]  [48:00]    [Simons, Anfernee]  [v sac]     [SG]   
    1  [57.4]   [$3,500]  [40:43]      [Allen, Grayson]  [@ lac]     [SG]   
    2  [56.9]   [$9,800]  [40:09]       [Walker, Kemba]  [v orl]     [PG]   
    3  [55.7]   [$3,600]  [48:00]        [Frazier, Tim]  [v okc]     [PG]   
    4  [55.7]  [$12,100]  [36:28]  [Westbrook, Russell]  [@ mil]     [PG]   
    
            Score                                              Stats   Team  
    0  [ 136-131]      [   37pt 6rb 9as 1st 2to 7trey 13-21fg 4-6ft]  [por]  
    1  [ 137-143]  [   40pt 7rb 4as 1st 1bl 3to 5trey 11-30fg 13-...  [uta]  
    2  [ 114-122]      [   43pt 2rb 5as 2bl 2to 4trey 16-25fg 7-7ft]  [cha]  
    3  [ 116-127]     [   29pt 6rb 13as 1bl 3to 4trey 10-23fg 5-7ft]  [mil]  
    4  [ 127-116]  [   15pt 11rb 17as 1st 1bl 4to 1trey 7-10fg 0-...  [okc]  

