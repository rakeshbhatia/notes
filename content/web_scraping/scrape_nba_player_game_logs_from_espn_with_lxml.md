# Scrape NBA Player Game Logs from ESPN with lxml

## Building the Scraper
### Import modules


```python
import csv
import time
import string
import datetime
import requests
import lxml.html as lh
import json
import numpy as np
import pandas as pd
```


```python
def parse_table(table):
    tr_elements = table.xpath('./tbody//tr')
    for tr_element in tr_elements:
        td_elements = tr_element.xpath('./td')
        if td_elements:
            for td_element in td_elements:
                print('td_element: ', td_element.text_content())
```

### Load RPM data and save to new DataFrame


```python
# Load rpm data to extract game log urls
rpm = pd.read_csv('nba_rpm_data_2018.csv')
rpm = rpm.drop(columns='Unnamed: 0')
rpm.head()
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
      <th>RK</th>
      <th>NAME</th>
      <th>POS</th>
      <th>TEAM</th>
      <th>GP</th>
      <th>MPG</th>
      <th>ORPM</th>
      <th>DRPM</th>
      <th>RPM</th>
      <th>WINS</th>
      <th>GAME LOG PAGE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Paul George</td>
      <td>SF</td>
      <td>OKC</td>
      <td>77</td>
      <td>36.9</td>
      <td>4.48</td>
      <td>3.08</td>
      <td>7.56</td>
      <td>19.73</td>
      <td>http://www.espn.com/nba/player/gamelog/_/id/42...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>James Harden</td>
      <td>PG</td>
      <td>HOU</td>
      <td>78</td>
      <td>36.8</td>
      <td>7.41</td>
      <td>0.00</td>
      <td>7.41</td>
      <td>18.53</td>
      <td>http://www.espn.com/nba/player/gamelog/_/id/39...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Stephen Curry</td>
      <td>PG</td>
      <td>GS</td>
      <td>69</td>
      <td>33.8</td>
      <td>5.92</td>
      <td>0.81</td>
      <td>6.73</td>
      <td>15.07</td>
      <td>http://www.espn.com/nba/player/gamelog/_/id/39...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Giannis Antetokounmpo</td>
      <td>PF</td>
      <td>MIL</td>
      <td>72</td>
      <td>32.8</td>
      <td>3.12</td>
      <td>3.39</td>
      <td>6.51</td>
      <td>14.94</td>
      <td>http://www.espn.com/nba/player/gamelog/_/id/30...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Joel Embiid</td>
      <td>C</td>
      <td>PHI</td>
      <td>64</td>
      <td>33.7</td>
      <td>2.70</td>
      <td>3.71</td>
      <td>6.41</td>
      <td>12.91</td>
      <td>http://www.espn.com/nba/player/gamelog/_/id/30...</td>
    </tr>
  </tbody>
</table>
</div>



### Save urls to list


```python
urls = list(rpm['GAME LOG PAGE'])
print('urls[:5]: ', urls[:5])
```

    urls[:5]:  ['http://www.espn.com/nba/player/gamelog/_/id/4251/paul-george', 'http://www.espn.com/nba/player/gamelog/_/id/3992/james-harden', 'http://www.espn.com/nba/player/gamelog/_/id/3975/stephen-curry', 'http://www.espn.com/nba/player/gamelog/_/id/3032977/giannis-antetokounmpo', 'http://www.espn.com/nba/player/gamelog/_/id/3059318/joel-embiid']


### Loop through the list and scrape each url


```python
for url in urls:
    page = requests.get(url)
    doc = lh.fromstring(page.content)
    div = doc.xpath('//*[@id="fittPageContainer"]/div[2]/div[5]/div/div[1]/div[1]/div/div[3]')[0]
    tables = div.xpath('//table[@class=\'Table2__table-scroller Table2__right-aligned Table2__table\']')
    tables = tables[2:-3]
    
    # ...
```


```python
url = 'http://www.espn.com/nba/player/gamelog/_/id/4251/paul-george'

page = requests.get(url)

doc = lh.fromstring(page.content)

div = doc.xpath('//*[@id="fittPageContainer"]/div[2]/div[5]/div/div[1]/div[1]/div/div[3]')

# Convert list to single item
div = div[0]

print('div: ', div)
```


```python
# Now get the divs that contain the stats tables for each month
divs = div.xpath('//div[@class=\'mb5\']')
print('divs: ', divs)
```


```python
tables = div.xpath('//table[@class=\'Table2__table-scroller Table2__right-aligned Table2__table\']')
tables = tables[2:-3]
print('tables: ', tables)
```

### Loop through each game log table and extract its `<td>` elements


```python
for url in urls:

    # ...
    
    # Create empty dataframe with columns
    df = pd.DataFrame(columns=['Date', 'Day of Week', 'Opp', 'Location', 'Result', 'Score', 'Min', 'FG', 'FG%', '3PT', '3PT%', 'FT', 'FT%', 'REB', 'AST', 'BLK', 'STL', 'PF', 'TO', 'PTS', 'FPTS'])
    months = ['october', 'november', 'december', 'january', 'february', 'march', 'april']
    game_log = []

    for table in tables:
        # Get <tr> elements
        tr_elements = table.xpath('./tbody//tr')
        for tr_element in tr_elements:
            # Get <td> elements
            td_elements = tr_element.xpath('./td[@class=\'Table2__td\']')
            
            # ...

```

### Parse each row of the table
* Extract the date
* Determine if game is at home or away
* Determine if game result was a win or loss
* Calculate total Fanduel points scored
* Add game to game log DataFrame


```python
for url in urls:

    # ...

    for table in tables:
    
        # ...

            if td_elements:
                # Ignore average stats row
                if td_elements[0].text_content() in months:
                    break
                count = 0
                game = []
                for td_element in td_elements:
                    # Get date
                    if count == 0:
                        game.append(td_element.text_content().split(' ')[1])
                        game.append(td_element.text_content().split(' ')[0])
                    # Determine if home/road
                    elif count == 1:
                        if 'vs' in td_element.text_content():
                            game.append(td_element.text_content().replace('vs', ''))
                            game.append('Home')
                        elif '@' in td_element.text_content():
                            game.append(td_element.text_content().replace('@', ''))
                            game.append('Away')
                    # Determine if win/loss
                    elif count == 2:
                        if 'W' in td_element.text_content():
                            game.append('W')
                            game.append(td_element.text_content().replace('W', ''))
                        elif 'L' in td_element.text_content():
                            game.append('L')
                            game.append(td_element.text_content().replace('L', ''))
                    elif count > 2:
                        game.append(td_element.text_content())
                    count += 1
                    
                    # ...
```

### Calculate Fanduel points and add row to game log DataFrame


```python
for url in urls:

    # ...

    for table in tables:
    
        # ...

            if td_elements:

                # ...
                
                for td_element in td_elements:

                    # ...
                    
                # Calculate total Fanduel points scored
                game.append(1.2*int(game[13])+1.5*int(game[14])+3*int(game[15])+3*int(game[16])+int(game[19])-int(game[18]))

                # Add to game log
                df.loc[len(df)] = game
```

## Our Complete Scraper


```python
import csv
import time
import string
import datetime
import requests
import lxml.html as lh
import json
import numpy as np
import pandas as pd

# Load rpm data to extract game log urls
rpm = pd.read_csv('nba_rpm_data_2018.csv')
rpm = rpm.drop(columns='Unnamed: 0')
rpm.head()

urls = list(rpm['GAME LOG PAGE'])

for url in urls:
    # Create empty dataframe with columns
    df = pd.DataFrame(columns=['Date', 'Day of Week', 'Opp', 'Location', 'Result', 'Score', 'Min', 'FG', 'FG%', '3PT', '3PT%', 'FT', 'FT%', 'REB', 'AST', 'BLK', 'STL', 'PF', 'TO', 'PTS', 'FPTS'])
    months = ['october', 'november', 'december', 'january', 'february', 'march', 'april']
    game_log = []

    for table in tables:
        # Get <tr> elements
        tr_elements = table.xpath('./tbody//tr')
        for tr_element in tr_elements:
            # Get <td> elements
            td_elements = tr_element.xpath('./td[@class=\'Table2__td\']')
            if td_elements:
                # Ignore average stats row
                if td_elements[0].text_content() in months:
                    break
                count = 0
                game = [] # to store data from a single game
                for td_element in td_elements:
                    print('td_element: ', td_element.text_content())
                    print('count: ', count)                
                    # Get date
                    if count == 0:
                        print('count = 0\n')
                        print('Date: {}\n'.format(td_element.text_content().split(' ')[1]))
                        print('Day of Week: {}\n'.format(td_element.text_content().split(' ')[0]))
                        game.append(td_element.text_content().split(' ')[1])
                        game.append(td_element.text_content().split(' ')[0])
                    # Determine if home/road
                    elif count == 1:
                        if 'vs' in td_element.text_content():
                            game.append(td_element.text_content().replace('vs', ''))
                            game.append('Home')
                        elif '@' in td_element.text_content():
                            game.append(td_element.text_content().replace('@', ''))
                            game.append('Away')
                    # Determine if win/loss
                    elif count == 2:
                        if 'W' in td_element.text_content():
                            game.append('W')
                            game.append(td_element.text_content().replace('W', ''))
                        elif 'L' in td_element.text_content():
                            game.append('L')
                            game.append(td_element.text_content().replace('L', ''))
                    elif count > 2:
                        game.append(td_element.text_content())
                    count += 1
                # Calculate total Fanduel points scored
                game.append(1.2*int(game[13])+1.5*int(game[14])+3*int(game[15])+3*int(game[16])+int(game[19])-int(game[18]))

                # Add to game log
                df.loc[len(df)] = game
    break

    df.head()
```

    td_element:  Sun 4/7
    count:  0
    count = 0
    
    Date: 4/7
    
    Day of Week: Sun
    
    td_element:  @BOS
    count:  1
    td_element:  W116-108
    count:  2
    td_element:  34
    count:  3
    td_element:  12-22
    count:  4
    td_element:  54.5
    count:  5
    td_element:  1-4
    count:  6
    td_element:  25.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  12
    count:  10
    td_element:  4
    count:  11
    td_element:  2
    count:  12
    td_element:  1
    count:  13
    td_element:  3
    count:  14
    td_element:  3
    count:  15
    td_element:  25
    count:  16
    td_element:  Fri 4/5
    count:  0
    count = 0
    
    Date: 4/5
    
    Day of Week: Fri
    
    td_element:  vsATL
    count:  1
    td_element:  W149-113
    count:  2
    td_element:  30
    count:  3
    td_element:  9-13
    count:  4
    td_element:  69.2
    count:  5
    td_element:  2-4
    count:  6
    td_element:  50.0
    count:  7
    td_element:  5-6
    count:  8
    td_element:  83.3
    count:  9
    td_element:  11
    count:  10
    td_element:  4
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  3
    count:  14
    td_element:  2
    count:  15
    td_element:  25
    count:  16
    td_element:  Wed 4/3
    count:  0
    count = 0
    
    Date: 4/3
    
    Day of Week: Wed
    
    td_element:  vsNY
    count:  1
    td_element:  W114-100
    count:  2
    td_element:  33
    count:  3
    td_element:  11-21
    count:  4
    td_element:  52.4
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  6-8
    count:  8
    td_element:  75.0
    count:  9
    td_element:  13
    count:  10
    td_element:  3
    count:  11
    td_element:  1
    count:  12
    td_element:  0
    count:  13
    td_element:  4
    count:  14
    td_element:  3
    count:  15
    td_element:  29
    count:  16
    td_element:  Mon 4/1
    count:  0
    count = 0
    
    Date: 4/1
    
    Day of Week: Mon
    
    td_element:  @TOR
    count:  1
    td_element:  L121-109
    count:  2
    td_element:  26
    count:  3
    td_element:  5-14
    count:  4
    td_element:  35.7
    count:  5
    td_element:  2-3
    count:  6
    td_element:  66.7
    count:  7
    td_element:  1-2
    count:  8
    td_element:  50.0
    count:  9
    td_element:  13
    count:  10
    td_element:  3
    count:  11
    td_element:  0
    count:  12
    td_element:  0
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  13
    count:  16
    td_element:  Sat 3/30
    count:  0
    count = 0
    
    Date: 3/30
    
    Day of Week: Sat
    
    td_element:  @IND
    count:  1
    td_element:  W121-116
    count:  2
    td_element:  32
    count:  3
    td_element:  8-18
    count:  4
    td_element:  44.4
    count:  5
    td_element:  0-3
    count:  6
    td_element:  0.0
    count:  7
    td_element:  3-4
    count:  8
    td_element:  75.0
    count:  9
    td_element:  2
    count:  10
    td_element:  2
    count:  11
    td_element:  0
    count:  12
    td_element:  2
    count:  13
    td_element:  2
    count:  14
    td_element:  3
    count:  15
    td_element:  19
    count:  16
    td_element:  Thu 3/28
    count:  0
    count = 0
    
    Date: 3/28
    
    Day of Week: Thu
    
    td_element:  @DET
    count:  1
    td_element:  L115-98
    count:  2
    td_element:  33
    count:  3
    td_element:  5-15
    count:  4
    td_element:  33.3
    count:  5
    td_element:  0-2
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  12
    count:  10
    td_element:  4
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  3
    count:  14
    td_element:  4
    count:  15
    td_element:  12
    count:  16
    td_element:  Tue 3/26
    count:  0
    count = 0
    
    Date: 3/26
    
    Day of Week: Tue
    
    td_element:  @MIA
    count:  1
    td_element:  W104-99
    count:  2
    td_element:  34
    count:  3
    td_element:  10-18
    count:  4
    td_element:  55.6
    count:  5
    td_element:  2-5
    count:  6
    td_element:  40.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  16
    count:  10
    td_element:  5
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  24
    count:  16
    td_element:  Mon 3/25
    count:  0
    count = 0
    
    Date: 3/25
    
    Day of Week: Mon
    
    td_element:  vsPHI
    count:  1
    td_element:  W119-98
    count:  2
    td_element:  29
    count:  3
    td_element:  11-21
    count:  4
    td_element:  52.4
    count:  5
    td_element:  2-5
    count:  6
    td_element:  40.0
    count:  7
    td_element:  4-4
    count:  8
    td_element:  100.0
    count:  9
    td_element:  11
    count:  10
    td_element:  4
    count:  11
    td_element:  0
    count:  12
    td_element:  0
    count:  13
    td_element:  0
    count:  14
    td_element:  0
    count:  15
    td_element:  28
    count:  16
    td_element:  Fri 3/22
    count:  0
    count = 0
    
    Date: 3/22
    
    Day of Week: Fri
    
    td_element:  vsMEM
    count:  1
    td_element:  W123-119 OT
    count:  2
    td_element:  41
    count:  3
    td_element:  12-23
    count:  4
    td_element:  52.2
    count:  5
    td_element:  0-4
    count:  6
    td_element:  0.0
    count:  7
    td_element:  1-2
    count:  8
    td_element:  50.0
    count:  9
    td_element:  4
    count:  10
    td_element:  3
    count:  11
    td_element:  2
    count:  12
    td_element:  2
    count:  13
    td_element:  2
    count:  14
    td_element:  0
    count:  15
    td_element:  25
    count:  16
    td_element:  Wed 3/20
    count:  0
    count = 0
    
    Date: 3/20
    
    Day of Week: Wed
    
    td_element:  vsNO
    count:  1
    td_element:  W119-96
    count:  2
    td_element:  26
    count:  3
    td_element:  5-14
    count:  4
    td_element:  35.7
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  4-4
    count:  8
    td_element:  100.0
    count:  9
    td_element:  17
    count:  10
    td_element:  3
    count:  11
    td_element:  3
    count:  12
    td_element:  0
    count:  13
    td_element:  1
    count:  14
    td_element:  3
    count:  15
    td_element:  15
    count:  16
    td_element:  Sun 3/17
    count:  0
    count = 0
    
    Date: 3/17
    
    Day of Week: Sun
    
    td_element:  vsATL
    count:  1
    td_element:  W101-91
    count:  2
    td_element:  34
    count:  3
    td_element:  10-20
    count:  4
    td_element:  50.0
    count:  5
    td_element:  0-3
    count:  6
    td_element:  0.0
    count:  7
    td_element:  7-7
    count:  8
    td_element:  100.0
    count:  9
    td_element:  20
    count:  10
    td_element:  3
    count:  11
    td_element:  0
    count:  12
    td_element:  3
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  27
    count:  16
    td_element:  Thu 3/14
    count:  0
    count = 0
    
    Date: 3/14
    
    Day of Week: Thu
    
    td_element:  vsCLE
    count:  1
    td_element:  W120-91
    count:  2
    td_element:  27
    count:  3
    td_element:  9-15
    count:  4
    td_element:  60.0
    count:  5
    td_element:  0-0
    count:  6
    td_element:  0.0
    count:  7
    td_element:  1-1
    count:  8
    td_element:  100.0
    count:  9
    td_element:  11
    count:  10
    td_element:  2
    count:  11
    td_element:  0
    count:  12
    td_element:  2
    count:  13
    td_element:  0
    count:  14
    td_element:  1
    count:  15
    td_element:  19
    count:  16
    td_element:  Wed 3/13
    count:  0
    count = 0
    
    Date: 3/13
    
    Day of Week: Wed
    
    td_element:  @WSH
    count:  1
    td_element:  L100-90
    count:  2
    td_element:  35
    count:  3
    td_element:  9-17
    count:  4
    td_element:  52.9
    count:  5
    td_element:  0-0
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  14
    count:  10
    td_element:  2
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  3
    count:  14
    td_element:  3
    count:  15
    td_element:  20
    count:  16
    td_element:  Sun 3/10
    count:  0
    count = 0
    
    Date: 3/10
    
    Day of Week: Sun
    
    td_element:  @MEM
    count:  1
    td_element:  L105-97
    count:  2
    td_element:  35
    count:  3
    td_element:  10-24
    count:  4
    td_element:  41.7
    count:  5
    td_element:  1-6
    count:  6
    td_element:  16.7
    count:  7
    td_element:  5-6
    count:  8
    td_element:  83.3
    count:  9
    td_element:  10
    count:  10
    td_element:  3
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  3
    count:  14
    td_element:  2
    count:  15
    td_element:  26
    count:  16
    td_element:  Fri 3/8
    count:  0
    count = 0
    
    Date: 3/8
    
    Day of Week: Fri
    
    td_element:  vsDAL
    count:  1
    td_element:  W111-106
    count:  2
    td_element:  35
    count:  3
    td_element:  8-17
    count:  4
    td_element:  47.1
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  3-4
    count:  8
    td_element:  75.0
    count:  9
    td_element:  13
    count:  10
    td_element:  6
    count:  11
    td_element:  1
    count:  12
    td_element:  2
    count:  13
    td_element:  2
    count:  14
    td_element:  3
    count:  15
    td_element:  20
    count:  16
    td_element:  Tue 3/5
    count:  0
    count = 0
    
    Date: 3/5
    
    Day of Week: Tue
    
    td_element:  @PHI
    count:  1
    td_element:  L114-106
    count:  2
    td_element:  34
    count:  3
    td_element:  5-15
    count:  4
    td_element:  33.3
    count:  5
    td_element:  0-1
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  12
    count:  10
    td_element:  2
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  12
    count:  16
    td_element:  Sun 3/3
    count:  0
    count = 0
    
    Date: 3/3
    
    Day of Week: Sun
    
    td_element:  @CLE
    count:  1
    td_element:  L107-93
    count:  2
    td_element:  36
    count:  3
    td_element:  13-16
    count:  4
    td_element:  81.3
    count:  5
    td_element:  0-0
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  13
    count:  10
    td_element:  6
    count:  11
    td_element:  2
    count:  12
    td_element:  2
    count:  13
    td_element:  2
    count:  14
    td_element:  3
    count:  15
    td_element:  28
    count:  16
    td_element:  Sat 3/2
    count:  0
    count = 0
    
    Date: 3/2
    
    Day of Week: Sat
    
    td_element:  @IND
    count:  1
    td_element:  W117-112
    count:  2
    td_element:  32
    count:  3
    td_element:  11-19
    count:  4
    td_element:  57.9
    count:  5
    td_element:  0-3
    count:  6
    td_element:  0.0
    count:  7
    td_element:  5-7
    count:  8
    td_element:  71.4
    count:  9
    td_element:  8
    count:  10
    td_element:  3
    count:  11
    td_element:  1
    count:  12
    td_element:  2
    count:  13
    td_element:  2
    count:  14
    td_element:  0
    count:  15
    td_element:  27
    count:  16
    td_element:  Thu 2/28
    count:  0
    count = 0
    
    Date: 2/28
    
    Day of Week: Thu
    
    td_element:  vsGS
    count:  1
    td_element:  W103-96
    count:  2
    td_element:  30
    count:  3
    td_element:  4-15
    count:  4
    td_element:  26.7
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  3-4
    count:  8
    td_element:  75.0
    count:  9
    td_element:  13
    count:  10
    td_element:  6
    count:  11
    td_element:  2
    count:  12
    td_element:  0
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  12
    count:  16
    td_element:  Tue 2/26
    count:  0
    count = 0
    
    Date: 2/26
    
    Day of Week: Tue
    
    td_element:  @NY
    count:  1
    td_element:  L108-103
    count:  2
    td_element:  34
    count:  3
    td_element:  12-19
    count:  4
    td_element:  63.2
    count:  5
    td_element:  2-2
    count:  6
    td_element:  100.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  11
    count:  10
    td_element:  6
    count:  11
    td_element:  3
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  1
    count:  15
    td_element:  26
    count:  16
    td_element:  Sun 2/24
    count:  0
    count = 0
    
    Date: 2/24
    
    Day of Week: Sun
    
    td_element:  @TOR
    count:  1
    td_element:  W113-98
    count:  2
    td_element:  31
    count:  3
    td_element:  10-17
    count:  4
    td_element:  58.8
    count:  5
    td_element:  3-4
    count:  6
    td_element:  75.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  12
    count:  10
    td_element:  4
    count:  11
    td_element:  1
    count:  12
    td_element:  2
    count:  13
    td_element:  1
    count:  14
    td_element:  0
    count:  15
    td_element:  23
    count:  16
    td_element:  Fri 2/22
    count:  0
    count = 0
    
    Date: 2/22
    
    Day of Week: Fri
    
    td_element:  vsCHI
    count:  1
    td_element:  L110-109
    count:  2
    td_element:  29
    count:  3
    td_element:  9-16
    count:  4
    td_element:  56.3
    count:  5
    td_element:  0-1
    count:  6
    td_element:  0.0
    count:  7
    td_element:  1-4
    count:  8
    td_element:  25.0
    count:  9
    td_element:  13
    count:  10
    td_element:  7
    count:  11
    td_element:  2
    count:  12
    td_element:  2
    count:  13
    td_element:  3
    count:  14
    td_element:  2
    count:  15
    td_element:  19
    count:  16
    td_element:  Sun 2/17
    count:  0
    count = 0
    
    Date: 2/17
    
    Day of Week: Sun
    
    td_element:  vsLEB*
    count:  1
    td_element:  L178-164
    count:  2
    td_element:  12
    count:  3
    td_element:  2-2
    count:  4
    td_element:  100.0
    count:  5
    td_element:  0-0
    count:  6
    td_element:  0.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  5
    count:  10
    td_element:  2
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  0
    count:  14
    td_element:  0
    count:  15
    td_element:  4
    count:  16
    td_element:  Thu 2/14
    count:  0
    count = 0
    
    Date: 2/14
    
    Day of Week: Thu
    
    td_element:  vsCHA
    count:  1
    td_element:  W127-89
    count:  2
    td_element:  26
    count:  3
    td_element:  7-13
    count:  4
    td_element:  53.8
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  2-3
    count:  8
    td_element:  66.7
    count:  9
    td_element:  11
    count:  10
    td_element:  4
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  17
    count:  16
    td_element:  Tue 2/12
    count:  0
    count = 0
    
    Date: 2/12
    
    Day of Week: Tue
    
    td_element:  @NO
    count:  1
    td_element:  W118-88
    count:  2
    td_element:  27
    count:  3
    td_element:  10-18
    count:  4
    td_element:  55.6
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  4-5
    count:  8
    td_element:  80.0
    count:  9
    td_element:  17
    count:  10
    td_element:  3
    count:  11
    td_element:  0
    count:  12
    td_element:  2
    count:  13
    td_element:  1
    count:  14
    td_element:  0
    count:  15
    td_element:  25
    count:  16
    td_element:  Sun 2/10
    count:  0
    count = 0
    
    Date: 2/10
    
    Day of Week: Sun
    
    td_element:  @ATL
    count:  1
    td_element:  W124-108
    count:  2
    td_element:  29
    count:  3
    td_element:  8-13
    count:  4
    td_element:  61.5
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  12
    count:  10
    td_element:  3
    count:  11
    td_element:  1
    count:  12
    td_element:  0
    count:  13
    td_element:  2
    count:  14
    td_element:  1
    count:  15
    td_element:  19
    count:  16
    td_element:  Sat 2/9
    count:  0
    count = 0
    
    Date: 2/9
    
    Day of Week: Sat
    
    td_element:  @MIL
    count:  1
    td_element:  W103-83
    count:  2
    td_element:  26
    count:  3
    td_element:  7-16
    count:  4
    td_element:  43.8
    count:  5
    td_element:  1-4
    count:  6
    td_element:  25.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  17
    count:  10
    td_element:  5
    count:  11
    td_element:  1
    count:  12
    td_element:  0
    count:  13
    td_element:  0
    count:  14
    td_element:  1
    count:  15
    td_element:  15
    count:  16
    td_element:  Thu 2/7
    count:  0
    count = 0
    
    Date: 2/7
    
    Day of Week: Thu
    
    td_element:  vsMIN
    count:  1
    td_element:  W122-112
    count:  2
    td_element:  31
    count:  3
    td_element:  9-16
    count:  4
    td_element:  56.3
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  10
    count:  10
    td_element:  3
    count:  11
    td_element:  2
    count:  12
    td_element:  0
    count:  13
    td_element:  1
    count:  14
    td_element:  1
    count:  15
    td_element:  21
    count:  16
    td_element:  Tue 2/5
    count:  0
    count = 0
    
    Date: 2/5
    
    Day of Week: Tue
    
    td_element:  @OKC
    count:  1
    td_element:  L132-122
    count:  2
    td_element:  34
    count:  3
    td_element:  8-18
    count:  4
    td_element:  44.4
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  9
    count:  10
    td_element:  5
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  4
    count:  15
    td_element:  17
    count:  16
    td_element:  Sat 2/2
    count:  0
    count = 0
    
    Date: 2/2
    
    Day of Week: Sat
    
    td_element:  vsBKN
    count:  1
    td_element:  W102-89
    count:  2
    td_element:  32
    count:  3
    td_element:  12-22
    count:  4
    td_element:  54.5
    count:  5
    td_element:  0-0
    count:  6
    td_element:  0.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  12
    count:  10
    td_element:  4
    count:  11
    td_element:  3
    count:  12
    td_element:  1
    count:  13
    td_element:  1
    count:  14
    td_element:  2
    count:  15
    td_element:  24
    count:  16
    td_element:  Thu 1/31
    count:  0
    count = 0
    
    Date: 1/31
    
    Day of Week: Thu
    
    td_element:  vsIND
    count:  1
    td_element:  W107-100
    count:  2
    td_element:  35
    count:  3
    td_element:  8-15
    count:  4
    td_element:  53.3
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  10
    count:  10
    td_element:  5
    count:  11
    td_element:  0
    count:  12
    td_element:  0
    count:  13
    td_element:  3
    count:  14
    td_element:  3
    count:  15
    td_element:  17
    count:  16
    td_element:  Tue 1/29
    count:  0
    count = 0
    
    Date: 1/29
    
    Day of Week: Tue
    
    td_element:  vsOKC
    count:  1
    td_element:  L126-117
    count:  2
    td_element:  34
    count:  3
    td_element:  12-20
    count:  4
    td_element:  60.0
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  11
    count:  10
    td_element:  5
    count:  11
    td_element:  2
    count:  12
    td_element:  1
    count:  13
    td_element:  1
    count:  14
    td_element:  3
    count:  15
    td_element:  27
    count:  16
    td_element:  Sun 1/27
    count:  0
    count = 0
    
    Date: 1/27
    
    Day of Week: Sun
    
    td_element:  @HOU
    count:  1
    td_element:  L103-98
    count:  2
    td_element:  34
    count:  3
    td_element:  8-19
    count:  4
    td_element:  42.1
    count:  5
    td_element:  0-2
    count:  6
    td_element:  0.0
    count:  7
    td_element:  3-4
    count:  8
    td_element:  75.0
    count:  9
    td_element:  17
    count:  10
    td_element:  5
    count:  11
    td_element:  1
    count:  12
    td_element:  2
    count:  13
    td_element:  4
    count:  14
    td_element:  4
    count:  15
    td_element:  19
    count:  16
    td_element:  Fri 1/25
    count:  0
    count = 0
    
    Date: 1/25
    
    Day of Week: Fri
    
    td_element:  vsWSH
    count:  1
    td_element:  L95-91
    count:  2
    td_element:  31
    count:  3
    td_element:  12-17
    count:  4
    td_element:  70.6
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  3-5
    count:  8
    td_element:  60.0
    count:  9
    td_element:  9
    count:  10
    td_element:  1
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  5
    count:  14
    td_element:  3
    count:  15
    td_element:  28
    count:  16
    td_element:  Wed 1/23
    count:  0
    count = 0
    
    Date: 1/23
    
    Day of Week: Wed
    
    td_element:  @BKN
    count:  1
    td_element:  L114-110
    count:  2
    td_element:  33
    count:  3
    td_element:  9-20
    count:  4
    td_element:  45.0
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  14
    count:  10
    td_element:  5
    count:  11
    td_element:  4
    count:  12
    td_element:  2
    count:  13
    td_element:  3
    count:  14
    td_element:  4
    count:  15
    td_element:  21
    count:  16
    td_element:  Mon 1/21
    count:  0
    count = 0
    
    Date: 1/21
    
    Day of Week: Mon
    
    td_element:  @ATL
    count:  1
    td_element:  W122-103
    count:  2
    td_element:  35
    count:  3
    td_element:  12-23
    count:  4
    td_element:  52.2
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  4-4
    count:  8
    td_element:  100.0
    count:  9
    td_element:  14
    count:  10
    td_element:  2
    count:  11
    td_element:  2
    count:  12
    td_element:  0
    count:  13
    td_element:  1
    count:  14
    td_element:  0
    count:  15
    td_element:  29
    count:  16
    td_element:  Sat 1/19
    count:  0
    count = 0
    
    Date: 1/19
    
    Day of Week: Sat
    
    td_element:  vsMIL
    count:  1
    td_element:  L118-108
    count:  2
    td_element:  30
    count:  3
    td_element:  11-24
    count:  4
    td_element:  45.8
    count:  5
    td_element:  2-5
    count:  6
    td_element:  40.0
    count:  7
    td_element:  3-3
    count:  8
    td_element:  100.0
    count:  9
    td_element:  6
    count:  10
    td_element:  4
    count:  11
    td_element:  2
    count:  12
    td_element:  0
    count:  13
    td_element:  0
    count:  14
    td_element:  0
    count:  15
    td_element:  27
    count:  16
    td_element:  Fri 1/18
    count:  0
    count = 0
    
    Date: 1/18
    
    Day of Week: Fri
    
    td_element:  vsBKN
    count:  1
    td_element:  L117-115
    count:  2
    td_element:  33
    count:  3
    td_element:  7-20
    count:  4
    td_element:  35.0
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  1-1
    count:  8
    td_element:  100.0
    count:  9
    td_element:  17
    count:  10
    td_element:  6
    count:  11
    td_element:  1
    count:  12
    td_element:  0
    count:  13
    td_element:  0
    count:  14
    td_element:  0
    count:  15
    td_element:  16
    count:  16
    td_element:  Wed 1/16
    count:  0
    count = 0
    
    Date: 1/16
    
    Day of Week: Wed
    
    td_element:  @DET
    count:  1
    td_element:  L120-115 OT
    count:  2
    td_element:  38
    count:  3
    td_element:  11-22
    count:  4
    td_element:  50.0
    count:  5
    td_element:  2-4
    count:  6
    td_element:  50.0
    count:  7
    td_element:  0-1
    count:  8
    td_element:  0.0
    count:  9
    td_element:  13
    count:  10
    td_element:  3
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  1
    count:  14
    td_element:  2
    count:  15
    td_element:  24
    count:  16
    td_element:  Sun 1/13
    count:  0
    count = 0
    
    Date: 1/13
    
    Day of Week: Sun
    
    td_element:  vsHOU
    count:  1
    td_element:  W116-109
    count:  2
    td_element:  35
    count:  3
    td_element:  9-16
    count:  4
    td_element:  56.3
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  3-4
    count:  8
    td_element:  75.0
    count:  9
    td_element:  9
    count:  10
    td_element:  6
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  4
    count:  14
    td_element:  2
    count:  15
    td_element:  22
    count:  16
    td_element:  Sat 1/12
    count:  0
    count = 0
    
    Date: 1/12
    
    Day of Week: Sat
    
    td_element:  vsBOS
    count:  1
    td_element:  W105-103
    count:  2
    td_element:  30
    count:  3
    td_element:  7-18
    count:  4
    td_element:  38.9
    count:  5
    td_element:  0-4
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-4
    count:  8
    td_element:  50.0
    count:  9
    td_element:  13
    count:  10
    td_element:  5
    count:  11
    td_element:  0
    count:  12
    td_element:  0
    count:  13
    td_element:  0
    count:  14
    td_element:  4
    count:  15
    td_element:  16
    count:  16
    td_element:  Wed 1/9
    count:  0
    count = 0
    
    Date: 1/9
    
    Day of Week: Wed
    
    td_element:  @UTAH
    count:  1
    td_element:  L106-93
    count:  2
    td_element:  31
    count:  3
    td_element:  8-17
    count:  4
    td_element:  47.1
    count:  5
    td_element:  3-4
    count:  6
    td_element:  75.0
    count:  7
    td_element:  1-2
    count:  8
    td_element:  50.0
    count:  9
    td_element:  8
    count:  10
    td_element:  3
    count:  11
    td_element:  0
    count:  12
    td_element:  2
    count:  13
    td_element:  4
    count:  14
    td_element:  2
    count:  15
    td_element:  20
    count:  16
    td_element:  Mon 1/7
    count:  0
    count = 0
    
    Date: 1/7
    
    Day of Week: Mon
    
    td_element:  @SAC
    count:  1
    td_element:  L111-95
    count:  2
    td_element:  25
    count:  3
    td_element:  7-14
    count:  4
    td_element:  50.0
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  3-4
    count:  8
    td_element:  75.0
    count:  9
    td_element:  13
    count:  10
    td_element:  3
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  1
    count:  14
    td_element:  1
    count:  15
    td_element:  18
    count:  16
    td_element:  Sun 1/6
    count:  0
    count = 0
    
    Date: 1/6
    
    Day of Week: Sun
    
    td_element:  @LAC
    count:  1
    td_element:  L106-96
    count:  2
    td_element:  32
    count:  3
    td_element:  7-17
    count:  4
    td_element:  41.2
    count:  5
    td_element:  2-5
    count:  6
    td_element:  40.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  24
    count:  10
    td_element:  8
    count:  11
    td_element:  2
    count:  12
    td_element:  0
    count:  13
    td_element:  1
    count:  14
    td_element:  3
    count:  15
    td_element:  16
    count:  16
    td_element:  Fri 1/4
    count:  0
    count = 0
    
    Date: 1/4
    
    Day of Week: Fri
    
    td_element:  @MIN
    count:  1
    td_element:  L120-103
    count:  2
    td_element:  27
    count:  3
    td_element:  10-16
    count:  4
    td_element:  62.5
    count:  5
    td_element:  1-4
    count:  6
    td_element:  25.0
    count:  7
    td_element:  1-1
    count:  8
    td_element:  100.0
    count:  9
    td_element:  7
    count:  10
    td_element:  3
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  1
    count:  15
    td_element:  22
    count:  16
    td_element:  Wed 1/2
    count:  0
    count = 0
    
    Date: 1/2
    
    Day of Week: Wed
    
    td_element:  @CHI
    count:  1
    td_element:  W112-84
    count:  2
    td_element:  26
    count:  3
    td_element:  10-15
    count:  4
    td_element:  66.7
    count:  5
    td_element:  1-1
    count:  6
    td_element:  100.0
    count:  7
    td_element:  1-3
    count:  8
    td_element:  33.3
    count:  9
    td_element:  12
    count:  10
    td_element:  3
    count:  11
    td_element:  3
    count:  12
    td_element:  2
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  22
    count:  16
    td_element:  Mon 12/31
    count:  0
    count = 0
    
    Date: 12/31
    
    Day of Week: Mon
    
    td_element:  @CHA
    count:  1
    td_element:  L125-100
    count:  2
    td_element:  23
    count:  3
    td_element:  5-13
    count:  4
    td_element:  38.5
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  1-4
    count:  8
    td_element:  25.0
    count:  9
    td_element:  5
    count:  10
    td_element:  2
    count:  11
    td_element:  2
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  0
    count:  15
    td_element:  12
    count:  16
    td_element:  Sun 12/30
    count:  0
    count = 0
    
    Date: 12/30
    
    Day of Week: Sun
    
    td_element:  vsDET
    count:  1
    td_element:  W109-107
    count:  2
    td_element:  34
    count:  3
    td_element:  10-15
    count:  4
    td_element:  66.7
    count:  5
    td_element:  0-2
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  11
    count:  10
    td_element:  4
    count:  11
    td_element:  2
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  3
    count:  15
    td_element:  22
    count:  16
    td_element:  Fri 12/28
    count:  0
    count = 0
    
    Date: 12/28
    
    Day of Week: Fri
    
    td_element:  vsTOR
    count:  1
    td_element:  W116-87
    count:  2
    td_element:  33
    count:  3
    td_element:  12-17
    count:  4
    td_element:  70.6
    count:  5
    td_element:  0-1
    count:  6
    td_element:  0.0
    count:  7
    td_element:  6-8
    count:  8
    td_element:  75.0
    count:  9
    td_element:  19
    count:  10
    td_element:  8
    count:  11
    td_element:  1
    count:  12
    td_element:  2
    count:  13
    td_element:  1
    count:  14
    td_element:  1
    count:  15
    td_element:  30
    count:  16
    td_element:  Wed 12/26
    count:  0
    count = 0
    
    Date: 12/26
    
    Day of Week: Wed
    
    td_element:  vsPHX
    count:  1
    td_element:  L122-120 OT
    count:  2
    td_element:  39
    count:  3
    td_element:  10-20
    count:  4
    td_element:  50.0
    count:  5
    td_element:  2-5
    count:  6
    td_element:  40.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  13
    count:  10
    td_element:  2
    count:  11
    td_element:  0
    count:  12
    td_element:  2
    count:  13
    td_element:  1
    count:  14
    td_element:  4
    count:  15
    td_element:  22
    count:  16
    td_element:  Sun 12/23
    count:  0
    count = 0
    
    Date: 12/23
    
    Day of Week: Sun
    
    td_element:  vsMIA
    count:  1
    td_element:  L115-91
    count:  2
    td_element:  29
    count:  3
    td_element:  3-12
    count:  4
    td_element:  25.0
    count:  5
    td_element:  0-1
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-5
    count:  8
    td_element:  40.0
    count:  9
    td_element:  7
    count:  10
    td_element:  0
    count:  11
    td_element:  1
    count:  12
    td_element:  0
    count:  13
    td_element:  1
    count:  14
    td_element:  3
    count:  15
    td_element:  8
    count:  16
    td_element:  Fri 12/21
    count:  0
    count = 0
    
    Date: 12/21
    
    Day of Week: Fri
    
    td_element:  @CHI
    count:  1
    td_element:  L90-80
    count:  2
    td_element:  37
    count:  3
    td_element:  8-19
    count:  4
    td_element:  42.1
    count:  5
    td_element:  1-5
    count:  6
    td_element:  20.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  19
    count:  10
    td_element:  4
    count:  11
    td_element:  4
    count:  12
    td_element:  0
    count:  13
    td_element:  3
    count:  14
    td_element:  0
    count:  15
    td_element:  19
    count:  16
    td_element:  Sat 12/15
    count:  0
    count = 0
    
    Date: 12/15
    
    Day of Week: Sat
    
    td_element:  vsUTAH
    count:  1
    td_element:  W96-89
    count:  2
    td_element:  30
    count:  3
    td_element:  5-14
    count:  4
    td_element:  35.7
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  4-4
    count:  8
    td_element:  100.0
    count:  9
    td_element:  19
    count:  10
    td_element:  5
    count:  11
    td_element:  2
    count:  12
    td_element:  0
    count:  13
    td_element:  3
    count:  14
    td_element:  2
    count:  15
    td_element:  15
    count:  16
    td_element:  Thu 12/13
    count:  0
    count = 0
    
    Date: 12/13
    
    Day of Week: Thu
    
    td_element:  vsCHI
    count:  1
    td_element:  W97-91
    count:  2
    td_element:  33
    count:  3
    td_element:  11-21
    count:  4
    td_element:  52.4
    count:  5
    td_element:  3-6
    count:  6
    td_element:  50.0
    count:  7
    td_element:  1-2
    count:  8
    td_element:  50.0
    count:  9
    td_element:  10
    count:  10
    td_element:  2
    count:  11
    td_element:  1
    count:  12
    td_element:  2
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  26
    count:  16
    td_element:  Mon 12/10
    count:  0
    count = 0
    
    Date: 12/10
    
    Day of Week: Mon
    
    td_element:  @DAL
    count:  1
    td_element:  L101-76
    count:  2
    td_element:  26
    count:  3
    td_element:  4-15
    count:  4
    td_element:  26.7
    count:  5
    td_element:  0-1
    count:  6
    td_element:  0.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  16
    count:  10
    td_element:  4
    count:  11
    td_element:  1
    count:  12
    td_element:  2
    count:  13
    td_element:  1
    count:  14
    td_element:  3
    count:  15
    td_element:  8
    count:  16
    td_element:  Fri 12/7
    count:  0
    count = 0
    
    Date: 12/7
    
    Day of Week: Fri
    
    td_element:  vsIND
    count:  1
    td_element:  L112-90
    count:  2
    td_element:  26
    count:  3
    td_element:  9-17
    count:  4
    td_element:  52.9
    count:  5
    td_element:  1-4
    count:  6
    td_element:  25.0
    count:  7
    td_element:  3-3
    count:  8
    td_element:  100.0
    count:  9
    td_element:  10
    count:  10
    td_element:  2
    count:  11
    td_element:  0
    count:  12
    td_element:  0
    count:  13
    td_element:  1
    count:  14
    td_element:  2
    count:  15
    td_element:  22
    count:  16
    td_element:  Wed 12/5
    count:  0
    count = 0
    
    Date: 12/5
    
    Day of Week: Wed
    
    td_element:  vsDEN
    count:  1
    td_element:  L124-118 OT
    count:  2
    td_element:  37
    count:  3
    td_element:  11-19
    count:  4
    td_element:  57.9
    count:  5
    td_element:  2-4
    count:  6
    td_element:  50.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  15
    count:  10
    td_element:  4
    count:  11
    td_element:  0
    count:  12
    td_element:  0
    count:  13
    td_element:  2
    count:  14
    td_element:  1
    count:  15
    td_element:  24
    count:  16
    td_element:  Tue 12/4
    count:  0
    count = 0
    
    Date: 12/4
    
    Day of Week: Tue
    
    td_element:  @MIA
    count:  1
    td_element:  W105-90
    count:  2
    td_element:  33
    count:  3
    td_element:  8-16
    count:  4
    td_element:  50.0
    count:  5
    td_element:  2-5
    count:  6
    td_element:  40.0
    count:  7
    td_element:  1-2
    count:  8
    td_element:  50.0
    count:  9
    td_element:  10
    count:  10
    td_element:  3
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  1
    count:  14
    td_element:  1
    count:  15
    td_element:  19
    count:  16
    td_element:  Fri 11/30
    count:  0
    count = 0
    
    Date: 11/30
    
    Day of Week: Fri
    
    td_element:  @PHX
    count:  1
    td_element:  W99-85
    count:  2
    td_element:  32
    count:  3
    td_element:  11-20
    count:  4
    td_element:  55.0
    count:  5
    td_element:  2-5
    count:  6
    td_element:  40.0
    count:  7
    td_element:  1-2
    count:  8
    td_element:  50.0
    count:  9
    td_element:  15
    count:  10
    td_element:  3
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  3
    count:  14
    td_element:  2
    count:  15
    td_element:  25
    count:  16
    td_element:  Wed 11/28
    count:  0
    count = 0
    
    Date: 11/28
    
    Day of Week: Wed
    
    td_element:  @POR
    count:  1
    td_element:  L115-112
    count:  2
    td_element:  29
    count:  3
    td_element:  8-12
    count:  4
    td_element:  66.7
    count:  5
    td_element:  2-2
    count:  6
    td_element:  100.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  8
    count:  10
    td_element:  7
    count:  11
    td_element:  2
    count:  12
    td_element:  0
    count:  13
    td_element:  6
    count:  14
    td_element:  1
    count:  15
    td_element:  20
    count:  16
    td_element:  Mon 11/26
    count:  0
    count = 0
    
    Date: 11/26
    
    Day of Week: Mon
    
    td_element:  @GS
    count:  1
    td_element:  L116-110
    count:  2
    td_element:  33
    count:  3
    td_element:  12-21
    count:  4
    td_element:  57.1
    count:  5
    td_element:  1-4
    count:  6
    td_element:  25.0
    count:  7
    td_element:  5-6
    count:  8
    td_element:  83.3
    count:  9
    td_element:  12
    count:  10
    td_element:  6
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  4
    count:  14
    td_element:  1
    count:  15
    td_element:  30
    count:  16
    td_element:  Sun 11/25
    count:  0
    count = 0
    
    Date: 11/25
    
    Day of Week: Sun
    
    td_element:  @LAL
    count:  1
    td_element:  W108-104
    count:  2
    td_element:  36
    count:  3
    td_element:  10-20
    count:  4
    td_element:  50.0
    count:  5
    td_element:  3-8
    count:  6
    td_element:  37.5
    count:  7
    td_element:  8-8
    count:  8
    td_element:  100.0
    count:  9
    td_element:  15
    count:  10
    td_element:  7
    count:  11
    td_element:  3
    count:  12
    td_element:  2
    count:  13
    td_element:  2
    count:  14
    td_element:  3
    count:  15
    td_element:  31
    count:  16
    td_element:  Fri 11/23
    count:  0
    count = 0
    
    Date: 11/23
    
    Day of Week: Fri
    
    td_element:  @DEN
    count:  1
    td_element:  L112-87
    count:  2
    td_element:  28
    count:  3
    td_element:  7-14
    count:  4
    td_element:  50.0
    count:  5
    td_element:  0-2
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  8
    count:  10
    td_element:  0
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  1
    count:  14
    td_element:  2
    count:  15
    td_element:  16
    count:  16
    td_element:  Tue 11/20
    count:  0
    count = 0
    
    Date: 11/20
    
    Day of Week: Tue
    
    td_element:  vsTOR
    count:  1
    td_element:  L93-91
    count:  2
    td_element:  34
    count:  3
    td_element:  6-12
    count:  4
    td_element:  50.0
    count:  5
    td_element:  0-3
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  18
    count:  10
    td_element:  4
    count:  11
    td_element:  0
    count:  12
    td_element:  0
    count:  13
    td_element:  2
    count:  14
    td_element:  3
    count:  15
    td_element:  14
    count:  16
    td_element:  Sun 11/18
    count:  0
    count = 0
    
    Date: 11/18
    
    Day of Week: Sun
    
    td_element:  vsNY
    count:  1
    td_element:  W131-117
    count:  2
    td_element:  32
    count:  3
    td_element:  10-15
    count:  4
    td_element:  66.7
    count:  5
    td_element:  2-3
    count:  6
    td_element:  66.7
    count:  7
    td_element:  6-6
    count:  8
    td_element:  100.0
    count:  9
    td_element:  10
    count:  10
    td_element:  9
    count:  11
    td_element:  2
    count:  12
    td_element:  0
    count:  13
    td_element:  2
    count:  14
    td_element:  3
    count:  15
    td_element:  28
    count:  16
    td_element:  Sat 11/17
    count:  0
    count = 0
    
    Date: 11/17
    
    Day of Week: Sat
    
    td_element:  vsLAL
    count:  1
    td_element:  W130-117
    count:  2
    td_element:  31
    count:  3
    td_element:  15-23
    count:  4
    td_element:  65.2
    count:  5
    td_element:  2-5
    count:  6
    td_element:  40.0
    count:  7
    td_element:  4-5
    count:  8
    td_element:  80.0
    count:  9
    td_element:  13
    count:  10
    td_element:  0
    count:  11
    td_element:  0
    count:  12
    td_element:  2
    count:  13
    td_element:  1
    count:  14
    td_element:  3
    count:  15
    td_element:  36
    count:  16
    td_element:  Wed 11/14
    count:  0
    count = 0
    
    Date: 11/14
    
    Day of Week: Wed
    
    td_element:  vsPHI
    count:  1
    td_element:  W111-106
    count:  2
    td_element:  39
    count:  3
    td_element:  10-19
    count:  4
    td_element:  52.6
    count:  5
    td_element:  3-6
    count:  6
    td_element:  50.0
    count:  7
    td_element:  7-8
    count:  8
    td_element:  87.5
    count:  9
    td_element:  8
    count:  10
    td_element:  4
    count:  11
    td_element:  0
    count:  12
    td_element:  2
    count:  13
    td_element:  1
    count:  14
    td_element:  1
    count:  15
    td_element:  30
    count:  16
    td_element:  Mon 11/12
    count:  0
    count = 0
    
    Date: 11/12
    
    Day of Week: Mon
    
    td_element:  @WSH
    count:  1
    td_element:  L117-109
    count:  2
    td_element:  32
    count:  3
    td_element:  8-17
    count:  4
    td_element:  47.1
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  11
    count:  10
    td_element:  3
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  17
    count:  16
    td_element:  Sun 11/11
    count:  0
    count = 0
    
    Date: 11/11
    
    Day of Week: Sun
    
    td_element:  @NY
    count:  1
    td_element:  W115-89
    count:  2
    td_element:  26
    count:  3
    td_element:  10-17
    count:  4
    td_element:  58.8
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  14
    count:  10
    td_element:  1
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  1
    count:  14
    td_element:  2
    count:  15
    td_element:  21
    count:  16
    td_element:  Fri 11/9
    count:  0
    count = 0
    
    Date: 11/9
    
    Day of Week: Fri
    
    td_element:  vsWSH
    count:  1
    td_element:  W117-108
    count:  2
    td_element:  31
    count:  3
    td_element:  10-16
    count:  4
    td_element:  62.5
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  14
    count:  10
    td_element:  3
    count:  11
    td_element:  2
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  21
    count:  16
    td_element:  Wed 11/7
    count:  0
    count = 0
    
    Date: 11/7
    
    Day of Week: Wed
    
    td_element:  vsDET
    count:  1
    td_element:  L103-96
    count:  2
    td_element:  34
    count:  3
    td_element:  6-14
    count:  4
    td_element:  42.9
    count:  5
    td_element:  0-3
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  9
    count:  10
    td_element:  4
    count:  11
    td_element:  1
    count:  12
    td_element:  0
    count:  13
    td_element:  5
    count:  14
    td_element:  4
    count:  15
    td_element:  14
    count:  16
    td_element:  Mon 11/5
    count:  0
    count = 0
    
    Date: 11/5
    
    Day of Week: Mon
    
    td_element:  vsCLE
    count:  1
    td_element:  W102-100
    count:  2
    td_element:  27
    count:  3
    td_element:  6-13
    count:  4
    td_element:  46.2
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  1-2
    count:  8
    td_element:  50.0
    count:  9
    td_element:  10
    count:  10
    td_element:  1
    count:  11
    td_element:  2
    count:  12
    td_element:  1
    count:  13
    td_element:  0
    count:  14
    td_element:  2
    count:  15
    td_element:  14
    count:  16
    td_element:  Sun 11/4
    count:  0
    count = 0
    
    Date: 11/4
    
    Day of Week: Sun
    
    td_element:  @SA
    count:  1
    td_element:  W117-110
    count:  2
    td_element:  32
    count:  3
    td_element:  6-11
    count:  4
    td_element:  54.5
    count:  5
    td_element:  0-1
    count:  6
    td_element:  0.0
    count:  7
    td_element:  1-3
    count:  8
    td_element:  33.3
    count:  9
    td_element:  8
    count:  10
    td_element:  3
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  1
    count:  14
    td_element:  0
    count:  15
    td_element:  13
    count:  16
    td_element:  Fri 11/2
    count:  0
    count = 0
    
    Date: 11/2
    
    Day of Week: Fri
    
    td_element:  vsLAC
    count:  1
    td_element:  L120-95
    count:  2
    td_element:  32
    count:  3
    td_element:  10-21
    count:  4
    td_element:  47.6
    count:  5
    td_element:  1-3
    count:  6
    td_element:  33.3
    count:  7
    td_element:  1-2
    count:  8
    td_element:  50.0
    count:  9
    td_element:  11
    count:  10
    td_element:  3
    count:  11
    td_element:  3
    count:  12
    td_element:  0
    count:  13
    td_element:  1
    count:  14
    td_element:  1
    count:  15
    td_element:  22
    count:  16
    td_element:  Tue 10/30
    count:  0
    count = 0
    
    Date: 10/30
    
    Day of Week: Tue
    
    td_element:  vsSAC
    count:  1
    td_element:  L107-99
    count:  2
    td_element:  27
    count:  3
    td_element:  5-11
    count:  4
    td_element:  45.5
    count:  5
    td_element:  0-2
    count:  6
    td_element:  0.0
    count:  7
    td_element:  5-6
    count:  8
    td_element:  83.3
    count:  9
    td_element:  15
    count:  10
    td_element:  5
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  1
    count:  14
    td_element:  3
    count:  15
    td_element:  15
    count:  16
    td_element:  Sat 10/27
    count:  0
    count = 0
    
    Date: 10/27
    
    Day of Week: Sat
    
    td_element:  @MIL
    count:  1
    td_element:  L113-91
    count:  2
    td_element:  21
    count:  3
    td_element:  7-10
    count:  4
    td_element:  70.0
    count:  5
    td_element:  0-0
    count:  6
    td_element:  0.0
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  9
    count:  10
    td_element:  1
    count:  11
    td_element:  1
    count:  12
    td_element:  1
    count:  13
    td_element:  4
    count:  14
    td_element:  3
    count:  15
    td_element:  16
    count:  16
    td_element:  Thu 10/25
    count:  0
    count = 0
    
    Date: 10/25
    
    Day of Week: Thu
    
    td_element:  vsPOR
    count:  1
    td_element:  L128-114
    count:  2
    td_element:  31
    count:  3
    td_element:  10-16
    count:  4
    td_element:  62.5
    count:  5
    td_element:  2-3
    count:  6
    td_element:  66.7
    count:  7
    td_element:  2-2
    count:  8
    td_element:  100.0
    count:  9
    td_element:  11
    count:  10
    td_element:  3
    count:  11
    td_element:  1
    count:  12
    td_element:  2
    count:  13
    td_element:  5
    count:  14
    td_element:  3
    count:  15
    td_element:  24
    count:  16
    td_element:  Mon 10/22
    count:  0
    count = 0
    
    Date: 10/22
    
    Day of Week: Mon
    
    td_element:  @BOS
    count:  1
    td_element:  W93-90
    count:  2
    td_element:  33
    count:  3
    td_element:  11-18
    count:  4
    td_element:  61.1
    count:  5
    td_element:  1-2
    count:  6
    td_element:  50.0
    count:  7
    td_element:  1-2
    count:  8
    td_element:  50.0
    count:  9
    td_element:  12
    count:  10
    td_element:  1
    count:  11
    td_element:  0
    count:  12
    td_element:  3
    count:  13
    td_element:  1
    count:  14
    td_element:  0
    count:  15
    td_element:  24
    count:  16
    td_element:  Sat 10/20
    count:  0
    count = 0
    
    Date: 10/20
    
    Day of Week: Sat
    
    td_element:  @PHI
    count:  1
    td_element:  L116-115
    count:  2
    td_element:  38
    count:  3
    td_element:  10-15
    count:  4
    td_element:  66.7
    count:  5
    td_element:  4-4
    count:  6
    td_element:  100.0
    count:  7
    td_element:  3-3
    count:  8
    td_element:  100.0
    count:  9
    td_element:  13
    count:  10
    td_element:  12
    count:  11
    td_element:  0
    count:  12
    td_element:  2
    count:  13
    td_element:  2
    count:  14
    td_element:  2
    count:  15
    td_element:  27
    count:  16
    td_element:  Fri 10/19
    count:  0
    count = 0
    
    Date: 10/19
    
    Day of Week: Fri
    
    td_element:  vsCHA
    count:  1
    td_element:  L120-88
    count:  2
    td_element:  22
    count:  3
    td_element:  6-11
    count:  4
    td_element:  54.5
    count:  5
    td_element:  0-0
    count:  6
    td_element:  0.0
    count:  7
    td_element:  0-0
    count:  8
    td_element:  0.0
    count:  9
    td_element:  8
    count:  10
    td_element:  4
    count:  11
    td_element:  4
    count:  12
    td_element:  0
    count:  13
    td_element:  1
    count:  14
    td_element:  2
    count:  15
    td_element:  12
    count:  16
    td_element:  Wed 10/17
    count:  0
    count = 0
    
    Date: 10/17
    
    Day of Week: Wed
    
    td_element:  vsMIA
    count:  1
    td_element:  W104-101
    count:  2
    td_element:  28
    count:  3
    td_element:  4-12
    count:  4
    td_element:  33.3
    count:  5
    td_element:  0-0
    count:  6
    td_element:  0.0
    count:  7
    td_element:  4-5
    count:  8
    td_element:  80.0
    count:  9
    td_element:  8
    count:  10
    td_element:  4
    count:  11
    td_element:  0
    count:  12
    td_element:  1
    count:  13
    td_element:  2
    count:  14
    td_element:  4
    count:  15
    td_element:  12
    count:  16


## Data Engineering

### Remove All-Star exhibition from game log


```python
# Remove All-Star game from gamelog
df = df[df.Opp != 'LEB*']
df = df[df.Opp != 'GIA*']
```

### Add year to 'Date' column


```python
# Data engineering
df['Date'] = df['Date'].apply(lambda x: x + '/19' if int(x.split('/')[0]) > 0 and int(x.split('/')[0]) <= 4 else x + '/18')
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
      <th>Date</th>
      <th>Day of Week</th>
      <th>Opp</th>
      <th>Location</th>
      <th>Result</th>
      <th>Score</th>
      <th>Min</th>
      <th>FG</th>
      <th>FG%</th>
      <th>3PT</th>
      <th>...</th>
      <th>FT</th>
      <th>FT%</th>
      <th>REB</th>
      <th>AST</th>
      <th>BLK</th>
      <th>STL</th>
      <th>PF</th>
      <th>TO</th>
      <th>PTS</th>
      <th>FPTS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4/7/19</td>
      <td>Sun</td>
      <td>BOS</td>
      <td>Away</td>
      <td>W</td>
      <td>116-108</td>
      <td>34</td>
      <td>12-22</td>
      <td>54.5</td>
      <td>1-4</td>
      <td>...</td>
      <td>0-0</td>
      <td>0.0</td>
      <td>12</td>
      <td>4</td>
      <td>2</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>25</td>
      <td>51.4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4/5/19</td>
      <td>Fri</td>
      <td>ATL</td>
      <td>Home</td>
      <td>W</td>
      <td>149-113</td>
      <td>30</td>
      <td>9-13</td>
      <td>69.2</td>
      <td>2-4</td>
      <td>...</td>
      <td>5-6</td>
      <td>83.3</td>
      <td>11</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>25</td>
      <td>45.2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4/3/19</td>
      <td>Wed</td>
      <td>NY</td>
      <td>Home</td>
      <td>W</td>
      <td>114-100</td>
      <td>33</td>
      <td>11-21</td>
      <td>52.4</td>
      <td>1-3</td>
      <td>...</td>
      <td>6-8</td>
      <td>75.0</td>
      <td>13</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>3</td>
      <td>29</td>
      <td>49.1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4/1/19</td>
      <td>Mon</td>
      <td>TOR</td>
      <td>Away</td>
      <td>L</td>
      <td>121-109</td>
      <td>26</td>
      <td>5-14</td>
      <td>35.7</td>
      <td>2-3</td>
      <td>...</td>
      <td>1-2</td>
      <td>50.0</td>
      <td>13</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>13</td>
      <td>31.1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3/30/19</td>
      <td>Sat</td>
      <td>IND</td>
      <td>Away</td>
      <td>W</td>
      <td>121-116</td>
      <td>32</td>
      <td>8-18</td>
      <td>44.4</td>
      <td>0-3</td>
      <td>...</td>
      <td>3-4</td>
      <td>75.0</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>19</td>
      <td>27.4</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>



### Add 'Days Rest' column


```python
# Insert column with empty cells
df.insert(2, 'Days Rest', '')

# Convert date to datetime format
df['Date'] = df['Date'].apply(lambda x: datetime.datetime.strptime(x, '%m/%d/%y'))

# Calculate days rest since previous game
df['Days Rest'] = (df['Date'] - df['Date'].shift(-1)).dt.days

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
      <th>Date</th>
      <th>Day of Week</th>
      <th>Days Rest</th>
      <th>Opp</th>
      <th>Location</th>
      <th>Result</th>
      <th>Score</th>
      <th>Min</th>
      <th>FG</th>
      <th>FG%</th>
      <th>...</th>
      <th>FT</th>
      <th>FT%</th>
      <th>REB</th>
      <th>AST</th>
      <th>BLK</th>
      <th>STL</th>
      <th>PF</th>
      <th>TO</th>
      <th>PTS</th>
      <th>FPTS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019-04-07</td>
      <td>Sun</td>
      <td>2.0</td>
      <td>BOS</td>
      <td>Away</td>
      <td>W</td>
      <td>116-108</td>
      <td>34</td>
      <td>12-22</td>
      <td>54.5</td>
      <td>...</td>
      <td>0-0</td>
      <td>0.0</td>
      <td>12</td>
      <td>4</td>
      <td>2</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>25</td>
      <td>51.4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-04-05</td>
      <td>Fri</td>
      <td>2.0</td>
      <td>ATL</td>
      <td>Home</td>
      <td>W</td>
      <td>149-113</td>
      <td>30</td>
      <td>9-13</td>
      <td>69.2</td>
      <td>...</td>
      <td>5-6</td>
      <td>83.3</td>
      <td>11</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>25</td>
      <td>45.2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-04-03</td>
      <td>Wed</td>
      <td>2.0</td>
      <td>NY</td>
      <td>Home</td>
      <td>W</td>
      <td>114-100</td>
      <td>33</td>
      <td>11-21</td>
      <td>52.4</td>
      <td>...</td>
      <td>6-8</td>
      <td>75.0</td>
      <td>13</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>3</td>
      <td>29</td>
      <td>49.1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019-04-01</td>
      <td>Mon</td>
      <td>2.0</td>
      <td>TOR</td>
      <td>Away</td>
      <td>L</td>
      <td>121-109</td>
      <td>26</td>
      <td>5-14</td>
      <td>35.7</td>
      <td>...</td>
      <td>1-2</td>
      <td>50.0</td>
      <td>13</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>13</td>
      <td>31.1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-03-30</td>
      <td>Sat</td>
      <td>2.0</td>
      <td>IND</td>
      <td>Away</td>
      <td>W</td>
      <td>121-116</td>
      <td>32</td>
      <td>8-18</td>
      <td>44.4</td>
      <td>...</td>
      <td>3-4</td>
      <td>75.0</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>19</td>
      <td>27.4</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>



### Do rolling calculations


```python
# Do rolling calculations
# Rolling calculation template
# df[''] = df[''].iloc[::-1].rolling().mean()

df['FPTS LAST 3'] = df['FPTS'].iloc[::-1].rolling(3).mean()
df['FPTS LAST 5'] = df['FPTS'].iloc[::-1].rolling(5).mean()
df['FPTS LAST 10'] = df['FPTS'].iloc[::-1].rolling(10).mean()

df.head(100)
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
      <th>Date</th>
      <th>Day of Week</th>
      <th>Days Rest</th>
      <th>Opp</th>
      <th>Location</th>
      <th>Result</th>
      <th>Score</th>
      <th>Min</th>
      <th>FG</th>
      <th>FG%</th>
      <th>...</th>
      <th>AST</th>
      <th>BLK</th>
      <th>STL</th>
      <th>PF</th>
      <th>TO</th>
      <th>PTS</th>
      <th>FPTS</th>
      <th>FPTS LAST 3</th>
      <th>FPTS LAST 5</th>
      <th>FPTS LAST 10</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019-04-07</td>
      <td>Sun</td>
      <td>2.0</td>
      <td>BOS</td>
      <td>Away</td>
      <td>W</td>
      <td>116-108</td>
      <td>34</td>
      <td>12-22</td>
      <td>54.5</td>
      <td>...</td>
      <td>4</td>
      <td>2</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>25</td>
      <td>51.4</td>
      <td>48.566667</td>
      <td>40.84</td>
      <td>42.97</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-04-05</td>
      <td>Fri</td>
      <td>2.0</td>
      <td>ATL</td>
      <td>Home</td>
      <td>W</td>
      <td>149-113</td>
      <td>30</td>
      <td>9-13</td>
      <td>69.2</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>25</td>
      <td>45.2</td>
      <td>41.800000</td>
      <td>36.84</td>
      <td>44.08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-04-03</td>
      <td>Wed</td>
      <td>2.0</td>
      <td>NY</td>
      <td>Home</td>
      <td>W</td>
      <td>114-100</td>
      <td>33</td>
      <td>11-21</td>
      <td>52.4</td>
      <td>...</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>4</td>
      <td>3</td>
      <td>29</td>
      <td>49.1</td>
      <td>35.866667</td>
      <td>38.74</td>
      <td>43.58</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019-04-01</td>
      <td>Mon</td>
      <td>2.0</td>
      <td>TOR</td>
      <td>Away</td>
      <td>L</td>
      <td>121-109</td>
      <td>26</td>
      <td>5-14</td>
      <td>35.7</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>13</td>
      <td>31.1</td>
      <td>29.966667</td>
      <td>38.36</td>
      <td>42.95</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-03-30</td>
      <td>Sat</td>
      <td>2.0</td>
      <td>IND</td>
      <td>Away</td>
      <td>W</td>
      <td>121-116</td>
      <td>32</td>
      <td>8-18</td>
      <td>44.4</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>19</td>
      <td>27.4</td>
      <td>37.833333</td>
      <td>41.40</td>
      <td>44.49</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2019-03-28</td>
      <td>Thu</td>
      <td>2.0</td>
      <td>DET</td>
      <td>Away</td>
      <td>L</td>
      <td>115-98</td>
      <td>33</td>
      <td>5-15</td>
      <td>33.3</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>4</td>
      <td>12</td>
      <td>31.4</td>
      <td>44.433333</td>
      <td>45.10</td>
      <td>46.81</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2019-03-26</td>
      <td>Tue</td>
      <td>1.0</td>
      <td>MIA</td>
      <td>Away</td>
      <td>W</td>
      <td>104-99</td>
      <td>34</td>
      <td>10-18</td>
      <td>55.6</td>
      <td>...</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>24</td>
      <td>54.7</td>
      <td>49.400000</td>
      <td>51.32</td>
      <td>46.71</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2019-03-25</td>
      <td>Mon</td>
      <td>3.0</td>
      <td>PHI</td>
      <td>Home</td>
      <td>W</td>
      <td>119-98</td>
      <td>29</td>
      <td>11-21</td>
      <td>52.4</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>28</td>
      <td>47.2</td>
      <td>46.466667</td>
      <td>48.42</td>
      <td>47.40</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2019-03-22</td>
      <td>Fri</td>
      <td>2.0</td>
      <td>MEM</td>
      <td>Home</td>
      <td>W</td>
      <td>123-119 OT</td>
      <td>41</td>
      <td>12-23</td>
      <td>52.2</td>
      <td>...</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>25</td>
      <td>46.3</td>
      <td>51.566667</td>
      <td>47.54</td>
      <td>47.69</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2019-03-20</td>
      <td>Wed</td>
      <td>3.0</td>
      <td>NO</td>
      <td>Home</td>
      <td>W</td>
      <td>119-96</td>
      <td>26</td>
      <td>5-14</td>
      <td>35.7</td>
      <td>...</td>
      <td>3</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>15</td>
      <td>45.9</td>
      <td>49.533333</td>
      <td>47.58</td>
      <td>47.12</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2019-03-17</td>
      <td>Sun</td>
      <td>3.0</td>
      <td>ATL</td>
      <td>Home</td>
      <td>W</td>
      <td>101-91</td>
      <td>34</td>
      <td>10-20</td>
      <td>50.0</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>27</td>
      <td>62.5</td>
      <td>48.500000</td>
      <td>48.52</td>
      <td>48.45</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2019-03-14</td>
      <td>Thu</td>
      <td>1.0</td>
      <td>CLE</td>
      <td>Home</td>
      <td>W</td>
      <td>120-91</td>
      <td>27</td>
      <td>9-15</td>
      <td>60.0</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>19</td>
      <td>40.2</td>
      <td>43.166667</td>
      <td>42.10</td>
      <td>47.44</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2019-03-13</td>
      <td>Wed</td>
      <td>3.0</td>
      <td>WSH</td>
      <td>Away</td>
      <td>L</td>
      <td>100-90</td>
      <td>35</td>
      <td>9-17</td>
      <td>52.9</td>
      <td>...</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>20</td>
      <td>42.8</td>
      <td>46.633333</td>
      <td>46.38</td>
      <td>48.93</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2019-03-10</td>
      <td>Sun</td>
      <td>2.0</td>
      <td>MEM</td>
      <td>Away</td>
      <td>L</td>
      <td>105-97</td>
      <td>35</td>
      <td>10-24</td>
      <td>41.7</td>
      <td>...</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>26</td>
      <td>46.5</td>
      <td>42.500000</td>
      <td>47.84</td>
      <td>48.67</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2019-03-08</td>
      <td>Fri</td>
      <td>3.0</td>
      <td>DAL</td>
      <td>Home</td>
      <td>W</td>
      <td>111-106</td>
      <td>35</td>
      <td>8-17</td>
      <td>47.1</td>
      <td>...</td>
      <td>6</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>20</td>
      <td>50.6</td>
      <td>47.533333</td>
      <td>46.66</td>
      <td>49.61</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2019-03-05</td>
      <td>Tue</td>
      <td>2.0</td>
      <td>PHI</td>
      <td>Away</td>
      <td>L</td>
      <td>114-106</td>
      <td>34</td>
      <td>5-15</td>
      <td>33.3</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>12</td>
      <td>30.4</td>
      <td>47.366667</td>
      <td>48.38</td>
      <td>48.54</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2019-03-03</td>
      <td>Sun</td>
      <td>1.0</td>
      <td>CLE</td>
      <td>Away</td>
      <td>L</td>
      <td>107-93</td>
      <td>36</td>
      <td>13-16</td>
      <td>81.3</td>
      <td>...</td>
      <td>6</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>28</td>
      <td>61.6</td>
      <td>50.766667</td>
      <td>52.78</td>
      <td>49.99</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2019-03-02</td>
      <td>Sat</td>
      <td>2.0</td>
      <td>IND</td>
      <td>Away</td>
      <td>W</td>
      <td>117-112</td>
      <td>32</td>
      <td>11-19</td>
      <td>57.9</td>
      <td>...</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>27</td>
      <td>50.1</td>
      <td>49.966667</td>
      <td>51.48</td>
      <td>48.08</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2019-02-28</td>
      <td>Thu</td>
      <td>2.0</td>
      <td>GS</td>
      <td>Home</td>
      <td>W</td>
      <td>103-96</td>
      <td>30</td>
      <td>4-15</td>
      <td>26.7</td>
      <td>...</td>
      <td>6</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>12</td>
      <td>40.6</td>
      <td>50.733333</td>
      <td>49.50</td>
      <td>46.50</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2019-02-26</td>
      <td>Tue</td>
      <td>2.0</td>
      <td>NY</td>
      <td>Away</td>
      <td>L</td>
      <td>108-103</td>
      <td>34</td>
      <td>12-19</td>
      <td>63.2</td>
      <td>...</td>
      <td>6</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>26</td>
      <td>59.2</td>
      <td>55.566667</td>
      <td>52.56</td>
      <td>47.88</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2019-02-24</td>
      <td>Sun</td>
      <td>2.0</td>
      <td>TOR</td>
      <td>Away</td>
      <td>W</td>
      <td>113-98</td>
      <td>31</td>
      <td>10-17</td>
      <td>58.8</td>
      <td>...</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>23</td>
      <td>52.4</td>
      <td>49.233333</td>
      <td>48.70</td>
      <td>45.31</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2019-02-22</td>
      <td>Fri</td>
      <td>8.0</td>
      <td>CHI</td>
      <td>Home</td>
      <td>L</td>
      <td>110-109</td>
      <td>29</td>
      <td>9-16</td>
      <td>56.3</td>
      <td>...</td>
      <td>7</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>2</td>
      <td>19</td>
      <td>55.1</td>
      <td>50.400000</td>
      <td>47.20</td>
      <td>45.44</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2019-02-14</td>
      <td>Thu</td>
      <td>2.0</td>
      <td>CHA</td>
      <td>Home</td>
      <td>W</td>
      <td>127-89</td>
      <td>26</td>
      <td>7-13</td>
      <td>53.8</td>
      <td>...</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>17</td>
      <td>40.2</td>
      <td>45.333333</td>
      <td>44.68</td>
      <td>45.12</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2019-02-12</td>
      <td>Tue</td>
      <td>2.0</td>
      <td>NO</td>
      <td>Away</td>
      <td>W</td>
      <td>118-88</td>
      <td>27</td>
      <td>10-18</td>
      <td>55.6</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>25</td>
      <td>55.9</td>
      <td>46.900000</td>
      <td>43.50</td>
      <td>45.13</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2019-02-10</td>
      <td>Sun</td>
      <td>1.0</td>
      <td>ATL</td>
      <td>Away</td>
      <td>W</td>
      <td>124-108</td>
      <td>29</td>
      <td>8-13</td>
      <td>61.5</td>
      <td>...</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>19</td>
      <td>39.9</td>
      <td>42.433333</td>
      <td>43.20</td>
      <td>45.47</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2019-02-09</td>
      <td>Sat</td>
      <td>2.0</td>
      <td>MIL</td>
      <td>Away</td>
      <td>W</td>
      <td>103-83</td>
      <td>26</td>
      <td>7-16</td>
      <td>43.8</td>
      <td>...</td>
      <td>5</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>15</td>
      <td>44.9</td>
      <td>40.566667</td>
      <td>41.92</td>
      <td>46.96</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2019-02-07</td>
      <td>Thu</td>
      <td>2.0</td>
      <td>MIN</td>
      <td>Home</td>
      <td>W</td>
      <td>122-112</td>
      <td>31</td>
      <td>9-16</td>
      <td>56.3</td>
      <td>...</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>21</td>
      <td>42.5</td>
      <td>43.733333</td>
      <td>43.68</td>
      <td>47.09</td>
    </tr>
    <tr>
      <th>28</th>
      <td>2019-02-05</td>
      <td>Tue</td>
      <td>3.0</td>
      <td>OKC</td>
      <td>Away</td>
      <td>L</td>
      <td>132-122</td>
      <td>34</td>
      <td>8-18</td>
      <td>44.4</td>
      <td>...</td>
      <td>5</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>4</td>
      <td>17</td>
      <td>34.3</td>
      <td>40.733333</td>
      <td>45.56</td>
      <td>47.68</td>
    </tr>
    <tr>
      <th>29</th>
      <td>2019-02-02</td>
      <td>Sat</td>
      <td>2.0</td>
      <td>BKN</td>
      <td>Home</td>
      <td>W</td>
      <td>102-89</td>
      <td>32</td>
      <td>12-22</td>
      <td>54.5</td>
      <td>...</td>
      <td>4</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>24</td>
      <td>54.4</td>
      <td>47.200000</td>
      <td>46.76</td>
      <td>48.76</td>
    </tr>
    <tr>
      <th>30</th>
      <td>2019-01-31</td>
      <td>Thu</td>
      <td>2.0</td>
      <td>IND</td>
      <td>Home</td>
      <td>W</td>
      <td>107-100</td>
      <td>35</td>
      <td>8-15</td>
      <td>53.3</td>
      <td>...</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>3</td>
      <td>17</td>
      <td>33.5</td>
      <td>46.366667</td>
      <td>47.74</td>
      <td>47.60</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>51</th>
      <td>2018-12-21</td>
      <td>Fri</td>
      <td>6.0</td>
      <td>CHI</td>
      <td>Away</td>
      <td>L</td>
      <td>90-80</td>
      <td>37</td>
      <td>8-19</td>
      <td>42.1</td>
      <td>...</td>
      <td>4</td>
      <td>4</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>19</td>
      <td>59.8</td>
      <td>52.366667</td>
      <td>46.26</td>
      <td>46.78</td>
    </tr>
    <tr>
      <th>52</th>
      <td>2018-12-15</td>
      <td>Sat</td>
      <td>2.0</td>
      <td>UTAH</td>
      <td>Home</td>
      <td>W</td>
      <td>96-89</td>
      <td>30</td>
      <td>5-14</td>
      <td>35.7</td>
      <td>...</td>
      <td>5</td>
      <td>2</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
      <td>15</td>
      <td>49.3</td>
      <td>45.500000</td>
      <td>43.70</td>
      <td>47.95</td>
    </tr>
    <tr>
      <th>53</th>
      <td>2018-12-13</td>
      <td>Thu</td>
      <td>3.0</td>
      <td>CHI</td>
      <td>Home</td>
      <td>W</td>
      <td>97-91</td>
      <td>33</td>
      <td>11-21</td>
      <td>52.4</td>
      <td>...</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>26</td>
      <td>48.0</td>
      <td>40.733333</td>
      <td>41.34</td>
      <td>45.68</td>
    </tr>
    <tr>
      <th>54</th>
      <td>2018-12-10</td>
      <td>Mon</td>
      <td>3.0</td>
      <td>DAL</td>
      <td>Away</td>
      <td>L</td>
      <td>101-76</td>
      <td>26</td>
      <td>4-15</td>
      <td>26.7</td>
      <td>...</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>3</td>
      <td>8</td>
      <td>39.2</td>
      <td>40.400000</td>
      <td>42.04</td>
      <td>44.74</td>
    </tr>
    <tr>
      <th>55</th>
      <td>2018-12-07</td>
      <td>Fri</td>
      <td>2.0</td>
      <td>IND</td>
      <td>Home</td>
      <td>L</td>
      <td>112-90</td>
      <td>26</td>
      <td>9-17</td>
      <td>52.9</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>22</td>
      <td>35.0</td>
      <td>39.833333</td>
      <td>43.22</td>
      <td>46.47</td>
    </tr>
    <tr>
      <th>56</th>
      <td>2018-12-05</td>
      <td>Wed</td>
      <td>1.0</td>
      <td>DEN</td>
      <td>Home</td>
      <td>L</td>
      <td>124-118 OT</td>
      <td>37</td>
      <td>11-19</td>
      <td>57.9</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>24</td>
      <td>47.0</td>
      <td>45.333333</td>
      <td>47.30</td>
      <td>48.43</td>
    </tr>
    <tr>
      <th>57</th>
      <td>2018-12-04</td>
      <td>Tue</td>
      <td>4.0</td>
      <td>MIA</td>
      <td>Away</td>
      <td>W</td>
      <td>105-90</td>
      <td>33</td>
      <td>8-16</td>
      <td>50.0</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>19</td>
      <td>37.5</td>
      <td>44.700000</td>
      <td>52.20</td>
      <td>48.79</td>
    </tr>
    <tr>
      <th>58</th>
      <td>2018-11-30</td>
      <td>Fri</td>
      <td>2.0</td>
      <td>PHX</td>
      <td>Away</td>
      <td>W</td>
      <td>99-85</td>
      <td>32</td>
      <td>11-20</td>
      <td>55.0</td>
      <td>...</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>25</td>
      <td>51.5</td>
      <td>50.666667</td>
      <td>50.02</td>
      <td>48.61</td>
    </tr>
    <tr>
      <th>59</th>
      <td>2018-11-28</td>
      <td>Wed</td>
      <td>2.0</td>
      <td>POR</td>
      <td>Away</td>
      <td>L</td>
      <td>115-112</td>
      <td>29</td>
      <td>8-12</td>
      <td>66.7</td>
      <td>...</td>
      <td>7</td>
      <td>2</td>
      <td>0</td>
      <td>6</td>
      <td>1</td>
      <td>20</td>
      <td>45.1</td>
      <td>57.333333</td>
      <td>47.44</td>
      <td>47.79</td>
    </tr>
    <tr>
      <th>60</th>
      <td>2018-11-26</td>
      <td>Mon</td>
      <td>1.0</td>
      <td>GS</td>
      <td>Away</td>
      <td>L</td>
      <td>116-110</td>
      <td>33</td>
      <td>12-21</td>
      <td>57.1</td>
      <td>...</td>
      <td>6</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>30</td>
      <td>55.4</td>
      <td>51.166667</td>
      <td>49.72</td>
      <td>48.21</td>
    </tr>
    <tr>
      <th>61</th>
      <td>2018-11-25</td>
      <td>Sun</td>
      <td>2.0</td>
      <td>LAL</td>
      <td>Away</td>
      <td>W</td>
      <td>108-104</td>
      <td>36</td>
      <td>10-20</td>
      <td>50.0</td>
      <td>...</td>
      <td>7</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>31</td>
      <td>71.5</td>
      <td>45.566667</td>
      <td>49.56</td>
      <td>45.65</td>
    </tr>
    <tr>
      <th>62</th>
      <td>2018-11-23</td>
      <td>Fri</td>
      <td>3.0</td>
      <td>DEN</td>
      <td>Away</td>
      <td>L</td>
      <td>112-87</td>
      <td>28</td>
      <td>7-14</td>
      <td>50.0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>16</td>
      <td>26.6</td>
      <td>40.566667</td>
      <td>45.38</td>
      <td>41.95</td>
    </tr>
    <tr>
      <th>63</th>
      <td>2018-11-20</td>
      <td>Tue</td>
      <td>2.0</td>
      <td>TOR</td>
      <td>Home</td>
      <td>L</td>
      <td>93-91</td>
      <td>34</td>
      <td>6-12</td>
      <td>50.0</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>14</td>
      <td>38.6</td>
      <td>49.900000</td>
      <td>47.20</td>
      <td>42.60</td>
    </tr>
    <tr>
      <th>64</th>
      <td>2018-11-18</td>
      <td>Sun</td>
      <td>1.0</td>
      <td>NY</td>
      <td>Home</td>
      <td>W</td>
      <td>131-117</td>
      <td>32</td>
      <td>10-15</td>
      <td>66.7</td>
      <td>...</td>
      <td>9</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>28</td>
      <td>56.5</td>
      <td>53.900000</td>
      <td>48.14</td>
      <td>43.51</td>
    </tr>
    <tr>
      <th>65</th>
      <td>2018-11-17</td>
      <td>Sat</td>
      <td>3.0</td>
      <td>LAL</td>
      <td>Home</td>
      <td>W</td>
      <td>130-117</td>
      <td>31</td>
      <td>15-23</td>
      <td>65.2</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>3</td>
      <td>36</td>
      <td>54.6</td>
      <td>46.966667</td>
      <td>46.70</td>
      <td>42.21</td>
    </tr>
    <tr>
      <th>66</th>
      <td>2018-11-14</td>
      <td>Wed</td>
      <td>2.0</td>
      <td>PHI</td>
      <td>Home</td>
      <td>W</td>
      <td>111-106</td>
      <td>39</td>
      <td>10-19</td>
      <td>52.6</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>30</td>
      <td>50.6</td>
      <td>43.200000</td>
      <td>41.74</td>
      <td>39.88</td>
    </tr>
    <tr>
      <th>67</th>
      <td>2018-11-12</td>
      <td>Mon</td>
      <td>1.0</td>
      <td>WSH</td>
      <td>Away</td>
      <td>L</td>
      <td>117-109</td>
      <td>32</td>
      <td>8-17</td>
      <td>47.1</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>17</td>
      <td>35.7</td>
      <td>42.766667</td>
      <td>38.52</td>
      <td>39.59</td>
    </tr>
    <tr>
      <th>68</th>
      <td>2018-11-11</td>
      <td>Sun</td>
      <td>2.0</td>
      <td>NY</td>
      <td>Away</td>
      <td>W</td>
      <td>115-89</td>
      <td>26</td>
      <td>10-17</td>
      <td>58.8</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>21</td>
      <td>43.3</td>
      <td>40.800000</td>
      <td>38.00</td>
      <td>40.91</td>
    </tr>
    <tr>
      <th>69</th>
      <td>2018-11-09</td>
      <td>Fri</td>
      <td>2.0</td>
      <td>WSH</td>
      <td>Home</td>
      <td>W</td>
      <td>117-108</td>
      <td>31</td>
      <td>10-16</td>
      <td>62.5</td>
      <td>...</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
      <td>2</td>
      <td>21</td>
      <td>49.3</td>
      <td>37.866667</td>
      <td>38.88</td>
      <td>43.04</td>
    </tr>
    <tr>
      <th>70</th>
      <td>2018-11-07</td>
      <td>Wed</td>
      <td>2.0</td>
      <td>DET</td>
      <td>Home</td>
      <td>L</td>
      <td>103-96</td>
      <td>34</td>
      <td>6-14</td>
      <td>42.9</td>
      <td>...</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>5</td>
      <td>4</td>
      <td>14</td>
      <td>29.8</td>
      <td>32.466667</td>
      <td>37.72</td>
      <td>41.87</td>
    </tr>
    <tr>
      <th>71</th>
      <td>2018-11-05</td>
      <td>Mon</td>
      <td>1.0</td>
      <td>CLE</td>
      <td>Home</td>
      <td>W</td>
      <td>102-100</td>
      <td>27</td>
      <td>6-13</td>
      <td>46.2</td>
      <td>...</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>14</td>
      <td>34.5</td>
      <td>38.433333</td>
      <td>38.02</td>
      <td>41.55</td>
    </tr>
    <tr>
      <th>72</th>
      <td>2018-11-04</td>
      <td>Sun</td>
      <td>2.0</td>
      <td>SA</td>
      <td>Away</td>
      <td>W</td>
      <td>117-110</td>
      <td>32</td>
      <td>6-11</td>
      <td>54.5</td>
      <td>...</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>13</td>
      <td>33.1</td>
      <td>41.433333</td>
      <td>40.66</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>73</th>
      <td>2018-11-02</td>
      <td>Fri</td>
      <td>3.0</td>
      <td>LAC</td>
      <td>Home</td>
      <td>L</td>
      <td>120-95</td>
      <td>32</td>
      <td>10-21</td>
      <td>47.6</td>
      <td>...</td>
      <td>3</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>22</td>
      <td>47.7</td>
      <td>40.833333</td>
      <td>43.82</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>74</th>
      <td>2018-10-30</td>
      <td>Tue</td>
      <td>3.0</td>
      <td>SAC</td>
      <td>Home</td>
      <td>L</td>
      <td>107-99</td>
      <td>27</td>
      <td>5-11</td>
      <td>45.5</td>
      <td>...</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>15</td>
      <td>43.5</td>
      <td>40.833333</td>
      <td>47.20</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>75</th>
      <td>2018-10-27</td>
      <td>Sat</td>
      <td>2.0</td>
      <td>MIL</td>
      <td>Away</td>
      <td>L</td>
      <td>113-91</td>
      <td>21</td>
      <td>7-10</td>
      <td>70.0</td>
      <td>...</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>3</td>
      <td>16</td>
      <td>31.3</td>
      <td>42.633333</td>
      <td>46.02</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>76</th>
      <td>2018-10-25</td>
      <td>Thu</td>
      <td>3.0</td>
      <td>POR</td>
      <td>Home</td>
      <td>L</td>
      <td>128-114</td>
      <td>31</td>
      <td>10-16</td>
      <td>62.5</td>
      <td>...</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>5</td>
      <td>3</td>
      <td>24</td>
      <td>47.7</td>
      <td>53.733333</td>
      <td>45.08</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>77</th>
      <td>2018-10-22</td>
      <td>Mon</td>
      <td>2.0</td>
      <td>BOS</td>
      <td>Away</td>
      <td>W</td>
      <td>93-90</td>
      <td>33</td>
      <td>11-18</td>
      <td>61.1</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>24</td>
      <td>48.9</td>
      <td>50.366667</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>78</th>
      <td>2018-10-20</td>
      <td>Sat</td>
      <td>1.0</td>
      <td>PHI</td>
      <td>Away</td>
      <td>L</td>
      <td>116-115</td>
      <td>38</td>
      <td>10-15</td>
      <td>66.7</td>
      <td>...</td>
      <td>12</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>27</td>
      <td>64.6</td>
      <td>42.933333</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>79</th>
      <td>2018-10-19</td>
      <td>Fri</td>
      <td>2.0</td>
      <td>CHA</td>
      <td>Home</td>
      <td>L</td>
      <td>120-88</td>
      <td>22</td>
      <td>6-11</td>
      <td>54.5</td>
      <td>...</td>
      <td>4</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>12</td>
      <td>37.6</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>80</th>
      <td>2018-10-17</td>
      <td>Wed</td>
      <td>NaN</td>
      <td>MIA</td>
      <td>Home</td>
      <td>W</td>
      <td>104-101</td>
      <td>28</td>
      <td>4-12</td>
      <td>33.3</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>4</td>
      <td>12</td>
      <td>26.6</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>80 rows × 25 columns</p>
</div>


