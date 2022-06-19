# Scrape NBA Player Real Plus-Minus (RPM) Stats from ESPN with lxml


```python
import csv
import time
import string
import requests
import lxml.html as lh
import json
import numpy
import pandas as pd
```

## Extract `<tr>` elements
* Create a GET request to the site's url
* Store the webpage contents
* Extract the `<tr>` elements


```python
url='http://www.espn.com/nba/statistics/rpm/_/sort/RPM'

# Create a GET request to the ESPN RPM site url
page = requests.get(url)

# Store the contents of the website
doc = lh.fromstring(page.content)

# Parse the data stored in all <tr> tags in the HTML
tr_elements = doc.xpath('//tr')

print('tr_elements: ', tr_elements)
```

    tr_elements:  [<Element tr at 0x1166d5b38>, <Element tr at 0x1167a3688>, <Element tr at 0x1167a3638>, <Element tr at 0x1167a3368>, <Element tr at 0x1167a3138>, <Element tr at 0x1167a32c8>, <Element tr at 0x1167a3188>, <Element tr at 0x1167a36d8>, <Element tr at 0x1166a5c28>, <Element tr at 0x11685b9a8>, <Element tr at 0x11685b9f8>, <Element tr at 0x11685b548>, <Element tr at 0x11685b868>, <Element tr at 0x11685b688>, <Element tr at 0x11685b778>, <Element tr at 0x11685bb88>, <Element tr at 0x11685bbd8>, <Element tr at 0x11685bc28>, <Element tr at 0x11685b7c8>, <Element tr at 0x11685bc78>, <Element tr at 0x11685bcc8>, <Element tr at 0x11685b728>, <Element tr at 0x11685bb38>, <Element tr at 0x11685b818>, <Element tr at 0x11685ba48>, <Element tr at 0x11685bdb8>, <Element tr at 0x11685b598>, <Element tr at 0x11685bae8>, <Element tr at 0x11685be58>, <Element tr at 0x11685be08>, <Element tr at 0x11685bd18>, <Element tr at 0x11685bd68>, <Element tr at 0x11685bf98>, <Element tr at 0x11685bf48>, <Element tr at 0x1166f0f98>, <Element tr at 0x116888368>, <Element tr at 0x1168884a8>, <Element tr at 0x1168882c8>, <Element tr at 0x116888318>, <Element tr at 0x116888548>, <Element tr at 0x116888188>]


## Check the length of each row of the table


```python
#Check the length of the first 12 rows
[len(T) for T in tr_elements[:12]]
```




    [9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9]



## Store first element (header) and empty list for each row


```python
# Create empty list
cols = []
i=0

# For each row, store each first element (header) and an empty list
for t in tr_elements[0]:
    i += 1        
    column_name = t.text_content()
    print('%d:"%s"'%(i, column_name))
    cols.append((column_name,[]))
```

    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"


## Extract links to all player pages


```python
# Extract player page links
links = doc.xpath('//*[@id="my-players-table"]/div/div[2]/table//a/@href')
# First four links are not player pages
links = links[4:]
print('links: ', links)
```

    links:  ['http://www.espn.com/nba/player/_/id/4251/paul-george', 'http://www.espn.com/nba/player/_/id/3992/james-harden', 'http://www.espn.com/nba/player/_/id/3975/stephen-curry', 'http://www.espn.com/nba/player/_/id/3032977/giannis-antetokounmpo', 'http://www.espn.com/nba/player/_/id/3112335/nikola-jokic', 'http://www.espn.com/nba/player/_/id/6583/anthony-davis', 'http://www.espn.com/nba/player/_/id/3059318/joel-embiid', 'http://www.espn.com/nba/player/_/id/6478/nikola-vucevic', 'http://www.espn.com/nba/player/_/id/1966/lebron-james', 'http://www.espn.com/nba/player/_/id/6606/damian-lillard', 'http://www.espn.com/nba/player/_/id/3202/kevin-durant', 'http://www.espn.com/nba/player/_/id/3988/danny-green', 'http://www.espn.com/nba/player/_/id/3012/kyle-lowry', 'http://www.espn.com/nba/player/_/id/3032976/rudy-gobert', 'http://www.espn.com/nba/player/_/id/2779/chris-paul', 'http://www.espn.com/nba/player/_/id/3995/jrue-holiday', 'http://www.espn.com/nba/player/_/id/3213/al-horford', 'http://www.espn.com/nba/player/_/id/6442/kyrie-irving', 'http://www.espn.com/nba/player/_/id/3448/brook-lopez', 'http://www.espn.com/nba/player/_/id/3015/paul-millsap', 'http://www.espn.com/nba/player/_/id/3149673/pascal-siakam', 'http://www.espn.com/nba/player/_/id/6430/jimmy-butler', 'http://www.espn.com/nba/player/_/id/3206/marc-gasol', 'http://www.espn.com/nba/player/_/id/3102530/jusuf-nurkic', 'http://www.espn.com/nba/player/_/id/3136195/karl-anthony-towns', 'http://www.espn.com/nba/player/_/id/4238/eric-bledsoe', 'http://www.espn.com/nba/player/_/id/3428/danilo-gallinari', 'http://www.espn.com/nba/player/_/id/3155535/kevon-looney', 'http://www.espn.com/nba/player/_/id/6450/kawhi-leonard', 'http://www.espn.com/nba/player/_/id/6589/draymond-green', 'http://www.espn.com/nba/player/_/id/3195/mike-conley', 'http://www.espn.com/nba/player/_/id/3989/blake-griffin', 'http://www.espn.com/nba/player/_/id/2968436/joe-ingles', 'http://www.espn.com/nba/player/_/id/6426/davis-bertans', 'http://www.espn.com/nba/player/_/id/4259/ed-davis', 'http://www.espn.com/nba/player/_/id/2490620/robert-covington', 'http://www.espn.com/nba/player/_/id/6479/kemba-walker', 'http://www.espn.com/nba/player/_/id/3964/patrick-beverley', 'http://www.espn.com/nba/player/_/id/2386/andre-iguodala', 'http://www.espn.com/nba/player/_/id/4257/derrick-favors']


## Store each `<tr>` element's data in a two-dimensional array
Each `<td>` element can be accessed using `iterchildren()`.


```python
# Our first row is the header (columns), so data is stored in the second row onwards
for j in range(1, len(tr_elements)):
    
    # T is our j'th row
    T = tr_elements[j]
    
    # If row is not of size 9, the //tr data is not from our table 
    if len(T) != 9:
        break
    
    # i is the index of our column
    i = 0
    
    # Iterate through each element of the row
    for t in T.iterchildren():
        data = t.text_content()
        print('data: ', data)
        # Check if row is empty
        if i > 0:
            # Extract link to player page
            if i == 1:
                link = t.xpath('/a/@href')
                print('link: ', link)
            # Convert any numerical value to integers
            try:
                data = int(data)
            except:
                pass
        # Append the data to the empty list of the i'th column
        col[i][1].append(data)
        # Increment i for the next column
        i += 1
```

    data:  1
    data:  Paul George, SF
    link:  []
    data:  OKC
    data:  77
    data:  36.9
    data:  4.49
    data:  3.09
    data:  7.58
    data:  19.77
    data:  2
    data:  James Harden, PG
    link:  []
    data:  HOU
    data:  78
    data:  36.8
    data:  7.38
    data:  0.09
    data:  7.47
    data:  18.63
    data:  3
    data:  Stephen Curry, PG
    link:  []
    data:  GS
    data:  69
    data:  33.8
    data:  5.86
    data:  0.68
    data:  6.54
    data:  14.78
    data:  4
    data:  Giannis Antetokounmpo, PF
    link:  []
    data:  MIL
    data:  72
    data:  32.8
    data:  3.13
    data:  3.09
    data:  6.22
    data:  14.49
    data:  5
    data:  Nikola Jokic, C
    link:  []
    data:  DEN
    data:  80
    data:  31.3
    data:  3.63
    data:  2.56
    data:  6.19
    data:  14.46
    data:  6
    data:  Anthony Davis, PF
    link:  []
    data:  NO
    data:  56
    data:  33.0
    data:  2.52
    data:  3.20
    data:  5.72
    data:  10.79
    data:  7
    data:  Joel Embiid, C
    link:  []
    data:  PHI
    data:  64
    data:  33.7
    data:  2.37
    data:  3.12
    data:  5.49
    data:  11.67
    data:  8
    data:  Nikola Vucevic, C
    link:  []
    data:  ORL
    data:  80
    data:  31.4
    data:  1.84
    data:  3.60
    data:  5.44
    data:  13.59
    data:  9
    data:  LeBron James, SF
    link:  []
    data:  LAL
    data:  55
    data:  35.2
    data:  3.48
    data:  1.84
    data:  5.32
    data:  10.89
    data:  10
    data:  Damian Lillard, PG
    link:  []
    data:  POR
    data:  80
    data:  35.5
    data:  5.69
    data:  -0.42
    data:  5.27
    data:  14.86
    data:  11
    data:  Kevin Durant, SF
    link:  []
    data:  GS
    data:  78
    data:  34.6
    data:  4.53
    data:  0.68
    data:  5.21
    data:  14.53
    data:  12
    data:  Danny Green, SG
    link:  []
    data:  TOR
    data:  80
    data:  27.7
    data:  2.92
    data:  2.16
    data:  5.08
    data:  11.73
    data:  13
    data:  Kyle Lowry, PG
    link:  []
    data:  TOR
    data:  65
    data:  34.0
    data:  2.83
    data:  2.10
    data:  4.93
    data:  11.36
    data:  14
    data:  Rudy Gobert, C
    link:  []
    data:  UTAH
    data:  81
    data:  31.8
    data:  0.26
    data:  4.44
    data:  4.70
    data:  13.20
    data:  15
    data:  Chris Paul, PG
    link:  []
    data:  HOU
    data:  58
    data:  32.0
    data:  2.40
    data:  2.27
    data:  4.67
    data:  9.06
    data:  16
    data:  Jrue Holiday, PG
    link:  []
    data:  NO
    data:  67
    data:  35.9
    data:  3.38
    data:  1.24
    data:  4.62
    data:  12.30
    data:  17
    data:  Al Horford, C
    link:  []
    data:  BOS
    data:  68
    data:  29.0
    data:  1.79
    data:  2.65
    data:  4.44
    data:  9.52
    data:  18
    data:  Kyrie Irving, PG
    link:  []
    data:  BOS
    data:  67
    data:  33.0
    data:  3.85
    data:  0.58
    data:  4.43
    data:  10.62
    data:  19
    data:  Brook Lopez, C
    link:  []
    data:  MIL
    data:  81
    data:  28.7
    data:  0.66
    data:  3.56
    data:  4.22
    data:  11.30
    data:  20
    data:  Paul Millsap, PF
    link:  []
    data:  DEN
    data:  70
    data:  27.1
    data:  1.50
    data:  2.65
    data:  4.15
    data:  8.58
    data:  21
    data:  Pascal Siakam, PF
    link:  []
    data:  TOR
    data:  80
    data:  31.9
    data:  1.91
    data:  2.23
    data:  4.14
    data:  11.86
    data:  22
    data:  Jimmy Butler, SG
    link:  []
    data:  MIN/PHI
    data:  65
    data:  33.6
    data:  2.27
    data:  1.82
    data:  4.09
    data:  10.11
    data:  23
    data:  Marc Gasol, C
    link:  []
    data:  TOR/MEM
    data:  79
    data:  30.8
    data:  0.95
    data:  2.98
    data:  3.93
    data:  10.54
    data:  24
    data:  Jusuf Nurkic, C
    link:  []
    data:  POR
    data:  72
    data:  27.4
    data:  0.73
    data:  3.12
    data:  3.85
    data:  8.66
    data:  25
    data:  Karl-Anthony Towns, C
    link:  []
    data:  MIN
    data:  77
    data:  33.1
    data:  2.99
    data:  0.84
    data:  3.83
    data:  11.32
    data:  26
    data:  Eric Bledsoe, PG
    link:  []
    data:  MIL
    data:  78
    data:  29.1
    data:  2.73
    data:  0.91
    data:  3.64
    data:  10.23
    data:  27
    data:  Danilo Gallinari, SF
    link:  []
    data:  LAC
    data:  68
    data:  30.3
    data:  3.47
    data:  0.00
    data:  3.47
    data:  8.55
    data:  28
    data:  Kevon Looney, C
    link:  []
    data:  GS
    data:  80
    data:  18.5
    data:  1.20
    data:  2.22
    data:  3.42
    data:  6.21
    data:  29
    data:  Kawhi Leonard, SF
    link:  []
    data:  TOR
    data:  60
    data:  34.0
    data:  3.31
    data:  0.01
    data:  3.32
    data:  8.40
    data:  30
    data:  Draymond Green, PF
    link:  []
    data:  GS
    data:  66
    data:  31.3
    data:  -0.32
    data:  3.52
    data:  3.20
    data:  8.39
    data:  31
    data:  Mike Conley, PG
    link:  []
    data:  MEM
    data:  70
    data:  33.5
    data:  3.54
    data:  -0.46
    data:  3.08
    data:  9.12
    data:  32
    data:  Blake Griffin, PF
    link:  []
    data:  DET
    data:  75
    data:  35.0
    data:  2.47
    data:  0.45
    data:  2.92
    data:  9.67
    data:  33
    data:  Joe Ingles, SF
    link:  []
    data:  UTAH
    data:  82
    data:  31.3
    data:  1.62
    data:  1.30
    data:  2.92
    data:  10.06
    data:  34
    data:  Davis Bertans, SF
    link:  []
    data:  SA
    data:  76
    data:  21.5
    data:  2.46
    data:  0.45
    data:  2.91
    data:  6.33
    data:  35
    data:  Ed Davis, PF
    link:  []
    data:  BKN
    data:  81
    data:  17.9
    data:  -1.42
    data:  4.20
    data:  2.78
    data:  5.42
    data:  36
    data:  Robert Covington, SF
    link:  []
    data:  MIN/PHI
    data:  35
    data:  34.4
    data:  -0.79
    data:  3.51
    data:  2.72
    data:  4.35
    data:  37
    data:  Kemba Walker, PG
    link:  []
    data:  CHA
    data:  82
    data:  34.9
    data:  4.17
    data:  -1.46
    data:  2.71
    data:  10.60
    data:  38
    data:  Patrick Beverley, PG
    link:  []
    data:  LAC
    data:  78
    data:  27.4
    data:  1.35
    data:  1.34
    data:  2.69
    data:  7.80
    data:  39
    data:  Andre Iguodala, SG
    link:  []
    data:  GS
    data:  68
    data:  23.2
    data:  0.72
    data:  1.90
    data:  2.62
    data:  5.64
    data:  40
    data:  Derrick Favors, PF
    link:  []
    data:  UTAH
    data:  76
    data:  23.2
    data:  -0.51
    data:  3.11
    data:  2.60
    data:  6.52


## Check the length of each column of data in the 2D array


```python
[len(C) for (title, C) in col]
```




    [891, 891, 891, 891, 891, 891, 891, 891, 891]



## Store the data in a new dataframe


```python
Dict = {title:column for (title, column) in col}
df = pd.DataFrame(Dict)
```

## Inspect the dataframe


```python
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
      <th>RK</th>
      <th>NAME</th>
      <th>TEAM</th>
      <th>GP</th>
      <th>MPG</th>
      <th>ORPM</th>
      <th>DRPM</th>
      <th>RPM</th>
      <th>WINS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Paul George, SF</td>
      <td>OKC</td>
      <td>77</td>
      <td>36.9</td>
      <td>4.49</td>
      <td>3.09</td>
      <td>7.58</td>
      <td>19.77</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>James Harden, PG</td>
      <td>HOU</td>
      <td>78</td>
      <td>36.8</td>
      <td>7.38</td>
      <td>0.09</td>
      <td>7.47</td>
      <td>18.63</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Stephen Curry, PG</td>
      <td>GS</td>
      <td>69</td>
      <td>33.8</td>
      <td>5.86</td>
      <td>0.68</td>
      <td>6.54</td>
      <td>14.78</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Giannis Antetokounmpo, PF</td>
      <td>MIL</td>
      <td>72</td>
      <td>32.8</td>
      <td>3.13</td>
      <td>3.09</td>
      <td>6.22</td>
      <td>14.49</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Nikola Jokic, C</td>
      <td>DEN</td>
      <td>80</td>
      <td>31.3</td>
      <td>3.63</td>
      <td>2.56</td>
      <td>6.19</td>
      <td>14.46</td>
    </tr>
  </tbody>
</table>
</div>



We now have our dataframe populated with the RPM data. However, we can see that under the 'NAME' column, the player name is combined with the player position. We want to separate the player position into a new column. However, before doing this, we need to handle pagination and collect the rest of the player data, as it is stored across 13 separate pages.

## Find the link to the next page


```python
# Extract the next page link
next_page_link = 'https:' + doc.xpath('//*[@id="my-players-table"]/div/div[2]/div/div[2]/a/@href')[0]
print(next_page_link)
```

    https://www.espn.com/nba/statistics/rpm/_/page/2


Let's put all of the above in its own method, which we can call repeatedly inside a wrapper function. Additionally, let's add a cleaning function to handle the separation of the player name and player position into separate features. Our complete scraper with all its methods are shown below.

## Complete scraper with data cleaning method and wrapper


```python
import csv
import string
import requests
import lxml.html as lh
import numpy
import pandas as pd

def clean_data(df):
    # Extract player positions into separate list
    positions = []
    for index, row in df.iterrows():
        positions.append(row['NAME'].split(', ')[1])
    
    print('total positions: ', len(positions))
    print('dataframe size: ', len(df.index))
    
    # Fix player name and add new column for player position
    df['NAME'] = df['NAME'].map(lambda x: x.split(', ')[0])
    df.insert(2, 'POS', positions)
    return df

# Main scraping method
def rpm_scraper_single_page(url, flag):
    # Create a get request to the site url
    page = requests.get(url)
    
    # Store the contents of the website
    doc = lh.fromstring(page.content)
    
    # Parse the data stored in all <tr> tags in the HTML
    tr_elements = doc.xpath('//tr')
    #print('tr_elements: ', tr_elements)
    
    #Check the length of the first 12 rows
    [len(T) for T in tr_elements[:12]]

    # Create empty list
    cols = []
    i=0

    # For each row, store each first element (header) and an empty list
    for t in tr_elements[0]:
        i += 1        
        column_name = t.text_content()
        print('%d:"%s"'%(i, column_name))
        cols.append((column_name,[]))

    # Extract player page links
    links = doc.xpath('//*[@id="my-players-table"]/div/div[2]/table//a/@href')
    # First four links are not player pages
    links = links[4:]
    # Add string 'gamelog/' to url
    links = ['gamelog/_'.join(link.split('_')) for link in links]
    print('links: ', links)
    print('total links: ', len(links))

    # Our first row is the header (columns), so data is stored in the second row onwards
    for j in range(1, len(tr_elements)):

        # T is our j'th row
        T = tr_elements[j]

        # If row is not of size 9, the //tr data is not from our table 
        if len(T) != 9:
            break

        # i is the index of our column
        i = 0

        # Iterate through each element of the row
        for t in T.iterchildren():
            data = t.text_content() 
            # Check if row is empty
            if i > 0:
                # Convert any numerical value to integers
                try:
                    data = int(data)
                except:
                    pass
            # Append the data to the empty list of the i'th column
            cols[i][1].append(data)
            # Increment i for the next column
            i += 1
        
    # Check the lengths
    [len(C) for (title, C) in cols]

    # Store the data in a dataframe
    Dict = {title:column for (title, column) in cols}
    rpm = pd.DataFrame(Dict)
    rpm.head()

    # Data cleaning
    rpm = clean_data(rpm)
    rpm.head()

    # Add links column to dataframe
    rpm['GAME LOG PAGE'] = links
        
    # Extract the next page link
    if flag:
        next_page_link = 'https:' + doc.xpath('//*[@id="my-players-table"]/div/div[2]/div/div[2]/a/@href')[0]
    else:      
        try:
            next_page_link = 'https:' + doc.xpath('//*[@id="my-players-table"]/div/div[2]/div/div[2]/a/@href')[1]
        except IndexError:
            next_page_link = None
    
    print(next_page_link)

    # Return a tuple with the dataframe and next page link
    return (rpm, next_page_link)

# Wrapper method
def rpm_scraper(base_url):
    url = None
    flag = False
    rpm_pages = []
    for i in range(0, 13):
        if i == 0:
            url = base_url
            flag = True
        else:
            flag = False
            
        if url:
            # Scrape all results on the current page
            rpm_tuple = rpm_scraper_single_page(url, flag)
            # Append the dataframe to list
            rpm_pages.append(rpm_tuple[0])
            # Update url to next page link
            url = rpm_tuple[1]

    return pd.concat(rpm_pages, ignore_index=False)

# Run scraper
base_url = 'http://www.espn.com/nba/statistics/rpm/_/sort/RPM'
rpm_data = rpm_scraper(base_url)

# Check results
rpm_data.head()
```

    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/4251/paul-george', 'http://www.espn.com/nba/player/gamelog/_/id/3992/james-harden', 'http://www.espn.com/nba/player/gamelog/_/id/3975/stephen-curry', 'http://www.espn.com/nba/player/gamelog/_/id/3032977/giannis-antetokounmpo', 'http://www.espn.com/nba/player/gamelog/_/id/3059318/joel-embiid', 'http://www.espn.com/nba/player/gamelog/_/id/3112335/nikola-jokic', 'http://www.espn.com/nba/player/gamelog/_/id/6583/anthony-davis', 'http://www.espn.com/nba/player/gamelog/_/id/6478/nikola-vucevic', 'http://www.espn.com/nba/player/gamelog/_/id/1966/lebron-james', 'http://www.espn.com/nba/player/gamelog/_/id/6606/damian-lillard', 'http://www.espn.com/nba/player/gamelog/_/id/3988/danny-green', 'http://www.espn.com/nba/player/gamelog/_/id/3202/kevin-durant', 'http://www.espn.com/nba/player/gamelog/_/id/3012/kyle-lowry', 'http://www.espn.com/nba/player/gamelog/_/id/3032976/rudy-gobert', 'http://www.espn.com/nba/player/gamelog/_/id/2779/chris-paul', 'http://www.espn.com/nba/player/gamelog/_/id/3995/jrue-holiday', 'http://www.espn.com/nba/player/gamelog/_/id/6430/jimmy-butler', 'http://www.espn.com/nba/player/gamelog/_/id/3213/al-horford', 'http://www.espn.com/nba/player/gamelog/_/id/3015/paul-millsap', 'http://www.espn.com/nba/player/gamelog/_/id/6442/kyrie-irving', 'http://www.espn.com/nba/player/gamelog/_/id/3102530/jusuf-nurkic', 'http://www.espn.com/nba/player/gamelog/_/id/3448/brook-lopez', 'http://www.espn.com/nba/player/gamelog/_/id/3149673/pascal-siakam', 'http://www.espn.com/nba/player/gamelog/_/id/3136195/karl-anthony-towns', 'http://www.espn.com/nba/player/gamelog/_/id/3206/marc-gasol', 'http://www.espn.com/nba/player/gamelog/_/id/4238/eric-bledsoe', 'http://www.espn.com/nba/player/gamelog/_/id/3428/danilo-gallinari', 'http://www.espn.com/nba/player/gamelog/_/id/6589/draymond-green', 'http://www.espn.com/nba/player/gamelog/_/id/6450/kawhi-leonard', 'http://www.espn.com/nba/player/gamelog/_/id/3155535/kevon-looney', 'http://www.espn.com/nba/player/gamelog/_/id/3195/mike-conley', 'http://www.espn.com/nba/player/gamelog/_/id/2968436/joe-ingles', 'http://www.espn.com/nba/player/gamelog/_/id/3989/blake-griffin', 'http://www.espn.com/nba/player/gamelog/_/id/6426/davis-bertans', 'http://www.espn.com/nba/player/gamelog/_/id/4259/ed-davis', 'http://www.espn.com/nba/player/gamelog/_/id/6479/kemba-walker', 'http://www.espn.com/nba/player/gamelog/_/id/2386/andre-iguodala', 'http://www.espn.com/nba/player/gamelog/_/id/3964/patrick-beverley', 'http://www.espn.com/nba/player/gamelog/_/id/2490620/robert-covington', 'http://www.espn.com/nba/player/gamelog/_/id/4257/derrick-favors']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/2
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/4258/demarcus-cousins', 'http://www.espn.com/nba/player/gamelog/_/id/3468/russell-westbrook', 'http://www.espn.com/nba/player/gamelog/_/id/6585/andre-drummond', 'http://www.espn.com/nba/player/gamelog/_/id/3005/rudy-gay', 'http://www.espn.com/nba/player/gamelog/_/id/2531367/dwight-powell', 'http://www.espn.com/nba/player/gamelog/_/id/2990992/marcus-smart', 'http://www.espn.com/nba/player/gamelog/_/id/2527963/victor-oladipo', 'http://www.espn.com/nba/player/gamelog/_/id/6609/khris-middleton', 'http://www.espn.com/nba/player/gamelog/_/id/3438/george-hill', 'http://www.espn.com/nba/player/gamelog/_/id/2489663/kelly-olynyk', 'http://www.espn.com/nba/player/gamelog/_/id/3135047/justise-winslow', 'http://www.espn.com/nba/player/gamelog/_/id/3908809/donovan-mitchell', 'http://www.espn.com/nba/player/gamelog/_/id/3078576/derrick-white', 'http://www.espn.com/nba/player/gamelog/_/id/3244/thaddeus-young', 'http://www.espn.com/nba/player/gamelog/_/id/2991235/steven-adams', 'http://www.espn.com/nba/player/gamelog/_/id/3102529/clint-capela', 'http://www.espn.com/nba/player/gamelog/_/id/4269/nemanja-bjelica', 'http://www.espn.com/nba/player/gamelog/_/id/3907387/ben-simmons', 'http://www.espn.com/nba/player/gamelog/_/id/2581190/josh-richardson', 'http://www.espn.com/nba/player/gamelog/_/id/2991282/willie-cauley-stein', 'http://www.espn.com/nba/player/gamelog/_/id/3059310/monte-morris', "http://www.espn.com/nba/player/gamelog/_/id/4066259/de'aaron-fox", 'http://www.espn.com/nba/player/gamelog/_/id/2451037/daniel-theis', 'http://www.espn.com/nba/player/gamelog/_/id/6440/tobias-harris', 'http://www.espn.com/nba/player/gamelog/_/id/3442/deandre-jordan', 'http://www.espn.com/nba/player/gamelog/_/id/6459/nikola-mirotic', 'http://www.espn.com/nba/player/gamelog/_/id/4248/al-farouq-aminu', 'http://www.espn.com/nba/player/gamelog/_/id/2566769/malcolm-brogdon', 'http://www.espn.com/nba/player/gamelog/_/id/4262/hassan-whiteside', 'http://www.espn.com/nba/player/gamelog/_/id/3133628/myles-turner', 'http://www.espn.com/nba/player/gamelog/_/id/2580913/dewayne-dedmon', 'http://www.espn.com/nba/player/gamelog/_/id/6603/jeremy-lamb', "http://www.espn.com/nba/player/gamelog/_/id/3136776/d'angelo-russell", 'http://www.espn.com/nba/player/gamelog/_/id/3028/thabo-sefolosha', 'http://www.espn.com/nba/player/gamelog/_/id/3155942/domantas-sabonis', 'http://www.espn.com/nba/player/gamelog/_/id/2983/lamarcus-aldridge', 'http://www.espn.com/nba/player/gamelog/_/id/2960236/maxi-kleber', 'http://www.espn.com/nba/player/gamelog/_/id/6580/bradley-beal', 'http://www.espn.com/nba/player/gamelog/_/id/4065648/jayson-tatum', 'http://www.espn.com/nba/player/gamelog/_/id/4066261/bam-adebayo']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/3
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/4011/ricky-rubio', 'http://www.espn.com/nba/player/gamelog/_/id/2991230/fred-vanvleet', 'http://www.espn.com/nba/player/gamelog/_/id/3449/kevin-love', 'http://www.espn.com/nba/player/gamelog/_/id/3064560/luke-kornet', 'http://www.espn.com/nba/player/gamelog/_/id/2767/ersan-ilyasova', 'http://www.espn.com/nba/player/gamelog/_/id/3945274/luka-doncic', 'http://www.espn.com/nba/player/gamelog/_/id/2490089/mike-muscala', 'http://www.espn.com/nba/player/gamelog/_/id/2594922/otto-porter-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/3064447/delon-wright', 'http://www.espn.com/nba/player/gamelog/_/id/3936299/jamal-murray', 'http://www.espn.com/nba/player/gamelog/_/id/3024/jj-redick', 'http://www.espn.com/nba/player/gamelog/_/id/3415/d.j.-augustin', 'http://www.espn.com/nba/player/gamelog/_/id/1713/nene-hilario', 'http://www.espn.com/nba/player/gamelog/_/id/2991280/nerlens-noel', 'http://www.espn.com/nba/player/gamelog/_/id/3224/joakim-noah', 'http://www.espn.com/nba/player/gamelog/_/id/2488653/mason-plumlee', 'http://www.espn.com/nba/player/gamelog/_/id/2580365/larry-nance-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/3033/pj-tucker', 'http://www.espn.com/nba/player/gamelog/_/id/2490149/cj-mccollum', 'http://www.espn.com/nba/player/gamelog/_/id/3135046/tyus-jones', 'http://www.espn.com/nba/player/gamelog/_/id/2991155/danuel-house-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/3934723/thomas-bryant', 'http://www.espn.com/nba/player/gamelog/_/id/3973/darren-collison', 'http://www.espn.com/nba/player/gamelog/_/id/4376/boban-marjanovic', 'http://www.espn.com/nba/player/gamelog/_/id/3134908/jakob-poeltl', 'http://www.espn.com/nba/player/gamelog/_/id/6446/cory-joseph', 'http://www.espn.com/nba/player/gamelog/_/id/3064290/aaron-gordon', "http://www.espn.com/nba/player/gamelog/_/id/2583632/royce-o'neale", 'http://www.espn.com/nba/player/gamelog/_/id/2429/luol-deng', 'http://www.espn.com/nba/player/gamelog/_/id/2578240/khem-birch', 'http://www.espn.com/nba/player/gamelog/_/id/3423/goran-dragic', 'http://www.espn.com/nba/player/gamelog/_/id/3078284/noah-vonleh', 'http://www.espn.com/nba/player/gamelog/_/id/6477/jonas-valanciunas', 'http://www.espn.com/nba/player/gamelog/_/id/6591/maurice-harkless', 'http://www.espn.com/nba/player/gamelog/_/id/2968439/aron-baynes', 'http://www.espn.com/nba/player/gamelog/_/id/2991070/jerami-grant', 'http://www.espn.com/nba/player/gamelog/_/id/3136193/devin-booker', 'http://www.espn.com/nba/player/gamelog/_/id/2990984/buddy-hield', 'http://www.espn.com/nba/player/gamelog/_/id/4066328/jarrett-allen', 'http://www.espn.com/nba/player/gamelog/_/id/6621/tomas-satoransky']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/4
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/2797/marvin-williams', 'http://www.espn.com/nba/player/gamelog/_/id/984/tyson-chandler', 'http://www.espn.com/nba/player/gamelog/_/id/4249/gordon-hayward', 'http://www.espn.com/nba/player/gamelog/_/id/6619/terrence-ross', 'http://www.espn.com/nba/player/gamelog/_/id/3431/eric-gordon', 'http://www.espn.com/nba/player/gamelog/_/id/2991055/montrezl-harrell', 'http://www.espn.com/nba/player/gamelog/_/id/6507/brad-wanamaker', 'http://www.espn.com/nba/player/gamelog/_/id/3914044/landry-shamet', 'http://www.espn.com/nba/player/gamelog/_/id/2999547/gary-harris', 'http://www.espn.com/nba/player/gamelog/_/id/4261/ekpe-udoh', 'http://www.espn.com/nba/player/gamelog/_/id/2530572/langston-galloway', 'http://www.espn.com/nba/player/gamelog/_/id/3554/omri-casspi', 'http://www.espn.com/nba/player/gamelog/_/id/3978/demar-derozan', 'http://www.espn.com/nba/player/gamelog/_/id/6443/reggie-jackson', 'http://www.espn.com/nba/player/gamelog/_/id/3593/bojan-bogdanovic', 'http://www.espn.com/nba/player/gamelog/_/id/2968361/raul-neto', 'http://www.espn.com/nba/player/gamelog/_/id/6605/meyers-leonard', 'http://www.espn.com/nba/player/gamelog/_/id/2579258/cody-zeller', 'http://www.espn.com/nba/player/gamelog/_/id/2578239/pat-connaughton', 'http://www.espn.com/nba/player/gamelog/_/id/2016/zaza-pachulia', 'http://www.espn.com/nba/player/gamelog/_/id/4004/patty-mills', 'http://www.espn.com/nba/player/gamelog/_/id/3064514/julius-randle', 'http://www.espn.com/nba/player/gamelog/_/id/2596107/alex-len', 'http://www.espn.com/nba/player/gamelog/_/id/3201/jared-dudley', 'http://www.espn.com/nba/player/gamelog/_/id/2991047/ryan-arcidiacono', 'http://www.espn.com/nba/player/gamelog/_/id/2993370/richaun-holmes', 'http://www.espn.com/nba/player/gamelog/_/id/2489693/ryan-broekhoff', 'http://www.espn.com/nba/player/gamelog/_/id/2991350/alex-caruso', 'http://www.espn.com/nba/player/gamelog/_/id/4066336/lauri-markkanen', 'http://www.espn.com/nba/player/gamelog/_/id/996/pau-gasol', 'http://www.espn.com/nba/player/gamelog/_/id/2579294/frank-kaminsky', 'http://www.espn.com/nba/player/gamelog/_/id/3037789/bogdan-bogdanovic', 'http://www.espn.com/nba/player/gamelog/_/id/3134907/kyle-kuzma', 'http://www.espn.com/nba/player/gamelog/_/id/4351852/mitchell-robinson', 'http://www.espn.com/nba/player/gamelog/_/id/6631/tyler-zeller', 'http://www.espn.com/nba/player/gamelog/_/id/3416/nicolas-batum', 'http://www.espn.com/nba/player/gamelog/_/id/3062679/josh-hart', 'http://www.espn.com/nba/player/gamelog/_/id/3998/jonas-jerebko', 'http://www.espn.com/nba/player/gamelog/_/id/2326307/seth-curry', 'http://www.espn.com/nba/player/gamelog/_/id/2799/lou-williams']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/5
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/3986/taj-gibson', 'http://www.espn.com/nba/player/gamelog/_/id/2528794/joe-harris', 'http://www.espn.com/nba/player/gamelog/_/id/3908845/john-collins', 'http://www.espn.com/nba/player/gamelog/_/id/6433/kenneth-faried', 'http://www.espn.com/nba/player/gamelog/_/id/2531352/eric-moreland', 'http://www.espn.com/nba/player/gamelog/_/id/3191/corey-brewer', 'http://www.espn.com/nba/player/gamelog/_/id/2968458/salah-mejri', 'http://www.espn.com/nba/player/gamelog/_/id/6578/harrison-barnes', 'http://www.espn.com/nba/player/gamelog/_/id/2530780/shabazz-napier', 'http://www.espn.com/nba/player/gamelog/_/id/4277961/jaren-jackson-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/4017837/ivica-zubac', 'http://www.espn.com/nba/player/gamelog/_/id/4278129/deandre-ayton', 'http://www.espn.com/nba/player/gamelog/_/id/6475/klay-thompson', 'http://www.espn.com/nba/player/gamelog/_/id/6592/john-henson', 'http://www.espn.com/nba/player/gamelog/_/id/2993874/kyle-anderson', 'http://www.espn.com/nba/player/gamelog/_/id/4066421/lonzo-ball', 'http://www.espn.com/nba/player/gamelog/_/id/3155536/thomas-welsh', 'http://www.espn.com/nba/player/gamelog/_/id/6581/jae-crowder', 'http://www.espn.com/nba/player/gamelog/_/id/2579326/alan-williams', 'http://www.espn.com/nba/player/gamelog/_/id/3062744/david-nwaba', 'http://www.espn.com/nba/player/gamelog/_/id/2579260/trey-burke', 'http://www.espn.com/nba/player/gamelog/_/id/4066650/zach-collins', 'http://www.espn.com/nba/player/gamelog/_/id/4065654/jonathan-isaac', 'http://www.espn.com/nba/player/gamelog/_/id/4066372/kevin-huerter', 'http://www.espn.com/nba/player/gamelog/_/id/3064559/damian-jones', 'http://www.espn.com/nba/player/gamelog/_/id/3064440/zach-lavine', 'http://www.espn.com/nba/player/gamelog/_/id/3919335/cheick-diallo', 'http://www.espn.com/nba/player/gamelog/_/id/3136479/angel-delgado', 'http://www.espn.com/nba/player/gamelog/_/id/2581177/rodney-hood', 'http://www.espn.com/nba/player/gamelog/_/id/6610/darius-miller', 'http://www.espn.com/nba/player/gamelog/_/id/3451/luc-mbah-a-moute', 'http://www.espn.com/nba/player/gamelog/_/id/2991043/caris-levert', 'http://www.espn.com/nba/player/gamelog/_/id/3947078/tyler-davis', 'http://www.espn.com/nba/player/gamelog/_/id/4032/wesley-matthews', 'http://www.espn.com/nba/player/gamelog/_/id/4023/garrett-temple', 'http://www.espn.com/nba/player/gamelog/_/id/3064517/jarell-martin', 'http://www.espn.com/nba/player/gamelog/_/id/3064528/johnathan-williams', 'http://www.espn.com/nba/player/gamelog/_/id/3147657/mikal-bridges', 'http://www.espn.com/nba/player/gamelog/_/id/3058254/christian-wood', 'http://www.espn.com/nba/player/gamelog/_/id/3970/demarre-carroll']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/6
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/3055/j.j.-barea', 'http://www.espn.com/nba/player/gamelog/_/id/2982249/amile-jefferson', 'http://www.espn.com/nba/player/gamelog/_/id/3906522/henry-ellenson', 'http://www.espn.com/nba/player/gamelog/_/id/3138156/justin-jackson', 'http://www.espn.com/nba/player/gamelog/_/id/2774/ian-mahinmi', 'http://www.espn.com/nba/player/gamelog/_/id/6464/donatas-motiejunas', 'http://www.espn.com/nba/player/gamelog/_/id/3912332/justin-patton', 'http://www.espn.com/nba/player/gamelog/_/id/4017839/juan-hernangomez', 'http://www.espn.com/nba/player/gamelog/_/id/2384/dwight-howard', 'http://www.espn.com/nba/player/gamelog/_/id/3439/serge-ibaka', 'http://www.espn.com/nba/player/gamelog/_/id/4015/jeff-teague', 'http://www.espn.com/nba/player/gamelog/_/id/6588/evan-fournier', 'http://www.espn.com/nba/player/gamelog/_/id/3209/jeff-green', 'http://www.espn.com/nba/player/gamelog/_/id/2581018/kentavious-caldwell-pope', 'http://www.espn.com/nba/player/gamelog/_/id/6462/marcus-morris', 'http://www.espn.com/nba/player/gamelog/_/id/3983/tyreke-evans', 'http://www.espn.com/nba/player/gamelog/_/id/6576/quincy-acy', 'http://www.espn.com/nba/player/gamelog/_/id/3937101/deyonta-davis', 'http://www.espn.com/nba/player/gamelog/_/id/2994526/bryn-forbes', 'http://www.espn.com/nba/player/gamelog/_/id/4066490/kostas-antetokounmpo', 'http://www.espn.com/nba/player/gamelog/_/id/2011/kyle-korver', 'http://www.espn.com/nba/player/gamelog/_/id/3074752/terry-rozier', 'http://www.espn.com/nba/player/gamelog/_/id/3892894/zhou-qi', 'http://www.espn.com/nba/player/gamelog/_/id/6601/michael-kidd-gilchrist', 'http://www.espn.com/nba/player/gamelog/_/id/2982334/t.j.-warren', 'http://www.espn.com/nba/player/gamelog/_/id/3908806/ray-spalding', 'http://www.espn.com/nba/player/gamelog/_/id/3981/wayne-ellington', 'http://www.espn.com/nba/player/gamelog/_/id/4222252/isaiah-hartenstein', 'http://www.espn.com/nba/player/gamelog/_/id/4355392/emanuel-terry', 'http://www.espn.com/nba/player/gamelog/_/id/3914285/drew-eubanks', 'http://www.espn.com/nba/player/gamelog/_/id/4066425/t.j.-leaf', 'http://www.espn.com/nba/player/gamelog/_/id/4066424/ike-anigbogu', 'http://www.espn.com/nba/player/gamelog/_/id/2528588/doug-mcdermott', 'http://www.espn.com/nba/player/gamelog/_/id/6616/miles-plumlee', 'http://www.espn.com/nba/player/gamelog/_/id/2595516/norman-powell', 'http://www.espn.com/nba/player/gamelog/_/id/3113297/bruno-caboclo', 'http://www.espn.com/nba/player/gamelog/_/id/3934670/marcus-derrickson', 'http://www.espn.com/nba/player/gamelog/_/id/2753/raymond-felton', 'http://www.espn.com/nba/player/gamelog/_/id/3907822/malik-beasley', 'http://www.espn.com/nba/player/gamelog/_/id/3232/jason-smith']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/7
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/3934662/tyler-lydon', 'http://www.espn.com/nba/player/gamelog/_/id/3917376/jaylen-brown', 'http://www.espn.com/nba/player/gamelog/_/id/136/vince-carter', 'http://www.espn.com/nba/player/gamelog/_/id/2608891/jakarr-sampson', 'http://www.espn.com/nba/player/gamelog/_/id/3056602/semi-ojeleye', 'http://www.espn.com/nba/player/gamelog/_/id/3136183/yante-maten', 'http://www.espn.com/nba/player/gamelog/_/id/4278077/jarred-vanderbilt', 'http://www.espn.com/nba/player/gamelog/_/id/3074765/isaiah-hicks', 'http://www.espn.com/nba/player/gamelog/_/id/4017838/ante-zizic', 'http://www.espn.com/nba/player/gamelog/_/id/3133603/kelly-oubre-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/3948153/chris-boucher', 'http://www.espn.com/nba/player/gamelog/_/id/4065673/tony-bradley', 'http://www.espn.com/nba/player/gamelog/_/id/3418/michael-beasley', 'http://www.espn.com/nba/player/gamelog/_/id/6617/austin-rivers', 'http://www.espn.com/nba/player/gamelog/_/id/4065663/josh-okogie', 'http://www.espn.com/nba/player/gamelog/_/id/4003/jodie-meeks', 'http://www.espn.com/nba/player/gamelog/_/id/3936099/derrick-jones-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/2528210/tim-hardaway-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/2184/udonis-haslem', 'http://www.espn.com/nba/player/gamelog/_/id/3138256/devaughn-akoon-purcell', 'http://www.espn.com/nba/player/gamelog/_/id/4348696/dzanan-musa', 'http://www.espn.com/nba/player/gamelog/_/id/3059319/andrew-wiggins', 'http://www.espn.com/nba/player/gamelog/_/id/6644/scott-machado', 'http://www.espn.com/nba/player/gamelog/_/id/3032978/dario-saric', 'http://www.espn.com/nba/player/gamelog/_/id/3129674/donte-grantham', 'http://www.espn.com/nba/player/gamelog/_/id/3155533/jonah-bolden', 'http://www.espn.com/nba/player/gamelog/_/id/3456/derrick-rose', 'http://www.espn.com/nba/player/gamelog/_/id/3136989/vincent-edwards', 'http://www.espn.com/nba/player/gamelog/_/id/3926491/isaac-humphries', 'http://www.espn.com/nba/player/gamelog/_/id/4067469/jemerrio-jones', 'http://www.espn.com/nba/player/gamelog/_/id/2578185/dorian-finney-smith', 'http://www.espn.com/nba/player/gamelog/_/id/2982268/jake-layman', 'http://www.espn.com/nba/player/gamelog/_/id/3059358/deonte-burton', 'http://www.espn.com/nba/player/gamelog/_/id/4253/quincy-pondexter', "http://www.espn.com/nba/player/gamelog/_/id/6615/kyle-o'quinn", 'http://www.espn.com/nba/player/gamelog/_/id/2758/marcin-gortat', 'http://www.espn.com/nba/player/gamelog/_/id/3999/james-johnson', 'http://www.espn.com/nba/player/gamelog/_/id/3059273/demetrius-jackson', 'http://www.espn.com/nba/player/gamelog/_/id/4305/ish-smith', 'http://www.espn.com/nba/player/gamelog/_/id/2580349/ron-baker']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/8
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/6628/dion-waiters', 'http://www.espn.com/nba/player/gamelog/_/id/2528693/torrey-craig', 'http://www.espn.com/nba/player/gamelog/_/id/2991018/yogi-ferrell', 'http://www.espn.com/nba/player/gamelog/_/id/3922230/aaron-holiday', 'http://www.espn.com/nba/player/gamelog/_/id/2579476/jalen-jones', 'http://www.espn.com/nba/player/gamelog/_/id/4230548/rodions-kurucs', 'http://www.espn.com/nba/player/gamelog/_/id/4278073/shai-gilgeous-alexander', 'http://www.espn.com/nba/player/gamelog/_/id/3447/robin-lopez', 'http://www.espn.com/nba/player/gamelog/_/id/3057240/cameron-reynolds', 'http://www.espn.com/nba/player/gamelog/_/id/2534781/gorgui-dieng', 'http://www.espn.com/nba/player/gamelog/_/id/2761/gerald-green', 'http://www.espn.com/nba/player/gamelog/_/id/4237/john-wall', 'http://www.espn.com/nba/player/gamelog/_/id/3136491/d.j.-wilson', 'http://www.espn.com/nba/player/gamelog/_/id/2426/trevor-ariza', 'http://www.espn.com/nba/player/gamelog/_/id/3059306/johnathan-motley', 'http://www.espn.com/nba/player/gamelog/_/id/3032979/dennis-schroder', 'http://www.espn.com/nba/player/gamelog/_/id/3074743/troy-caupain', 'http://www.espn.com/nba/player/gamelog/_/id/3133626/kenrich-williams', 'http://www.espn.com/nba/player/gamelog/_/id/3137798/devin-robinson', 'http://www.espn.com/nba/player/gamelog/_/id/3923250/pj-dozier', 'http://www.espn.com/nba/player/gamelog/_/id/3913174/luke-kennard', 'http://www.espn.com/nba/player/gamelog/_/id/4192/milos-teodosic', 'http://www.espn.com/nba/player/gamelog/_/id/3934673/donte-divincenzo', 'http://www.espn.com/nba/player/gamelog/_/id/3952343/jaylen-morris', 'http://www.espn.com/nba/player/gamelog/_/id/2528353/tony-snell', 'http://www.espn.com/nba/player/gamelog/_/id/2531797/jordan-mcrae', 'http://www.espn.com/nba/player/gamelog/_/id/2992257/tahjere-mccall', 'http://www.espn.com/nba/player/gamelog/_/id/2995702/alex-abrines', 'http://www.espn.com/nba/player/gamelog/_/id/2141179/andre-ingram', 'http://www.espn.com/nba/player/gamelog/_/id/2579492/julian-washburn', 'http://www.espn.com/nba/player/gamelog/_/id/3001309/dusty-hannahs', 'http://www.espn.com/nba/player/gamelog/_/id/3907821/dwayne-bacon', 'http://www.espn.com/nba/player/gamelog/_/id/4017843/thon-maker', 'http://www.espn.com/nba/player/gamelog/_/id/2444/jr-smith', 'http://www.espn.com/nba/player/gamelog/_/id/2580782/spencer-dinwiddie', 'http://www.espn.com/nba/player/gamelog/_/id/3059262/davon-reed', 'http://www.espn.com/nba/player/gamelog/_/id/2489563/dj-stephens', 'http://www.espn.com/nba/player/gamelog/_/id/3056600/jabari-parker', 'http://www.espn.com/nba/player/gamelog/_/id/3913176/brandon-ingram', 'http://www.espn.com/nba/player/gamelog/_/id/6447/enes-kanter']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/9
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/2769/amir-johnson', 'http://www.espn.com/nba/player/gamelog/_/id/3134880/kadeem-allen', 'http://www.espn.com/nba/player/gamelog/_/id/3243/nick-young', 'http://www.espn.com/nba/player/gamelog/_/id/3064320/george-king', 'http://www.espn.com/nba/player/gamelog/_/id/3445/courtney-lee', 'http://www.espn.com/nba/player/gamelog/_/id/2595592/jordan-loyd', 'http://www.espn.com/nba/player/gamelog/_/id/3934672/jalen-brunson', 'http://www.espn.com/nba/player/gamelog/_/id/3936296/skal-labissiere', 'http://www.espn.com/nba/player/gamelog/_/id/4067045/alize-johnson', 'http://www.espn.com/nba/player/gamelog/_/id/3929325/furkan-korkmaz', 'http://www.espn.com/nba/player/gamelog/_/id/3059280/bj-johnson', 'http://www.espn.com/nba/player/gamelog/_/id/4066383/miles-bridges', 'http://www.espn.com/nba/player/gamelog/_/id/2489716/matthew-dellavedova', 'http://www.espn.com/nba/player/gamelog/_/id/3059430/zach-lofton', "http://www.espn.com/nba/player/gamelog/_/id/3062667/deandre'-bembry", 'http://www.espn.com/nba/player/gamelog/_/id/3136196/trey-lyles', 'http://www.espn.com/nba/player/gamelog/_/id/4017844/guerschon-yabusele', 'http://www.espn.com/nba/player/gamelog/_/id/6468/iman-shumpert', 'http://www.espn.com/nba/player/gamelog/_/id/3058269/joe-chealey', 'http://www.espn.com/nba/player/gamelog/_/id/2991139/kris-dunn', 'http://www.espn.com/nba/player/gamelog/_/id/4277847/wendell-carter-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/4065697/dennis-smith-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/2488945/tim-frazier', 'http://www.espn.com/nba/player/gamelog/_/id/3064482/bobby-portis', 'http://www.espn.com/nba/player/gamelog/_/id/3190/marco-belinelli', 'http://www.espn.com/nba/player/gamelog/_/id/2579279/jordan-sibert', 'http://www.espn.com/nba/player/gamelog/_/id/2583639/elfrid-payton', 'http://www.espn.com/nba/player/gamelog/_/id/3135048/jahlil-okafor', 'http://www.espn.com/nba/player/gamelog/_/id/4244/lance-stephenson', 'http://www.espn.com/nba/player/gamelog/_/id/3136483/j.p.-macura', 'http://www.espn.com/nba/player/gamelog/_/id/3138154/theo-pinson', 'http://www.espn.com/nba/player/gamelog/_/id/2530276/tyler-johnson', 'http://www.espn.com/nba/player/gamelog/_/id/4348697/isaac-bonga', 'http://www.espn.com/nba/player/gamelog/_/id/3276/anthony-tolliver', 'http://www.espn.com/nba/player/gamelog/_/id/6622/mike-scott', 'http://www.espn.com/nba/player/gamelog/_/id/2531047/jerian-grant', 'http://www.espn.com/nba/player/gamelog/_/id/2982240/jaron-blossomgame', 'http://www.espn.com/nba/player/gamelog/_/id/3452/javale-mcgee', 'http://www.espn.com/nba/player/gamelog/_/id/3915560/tyler-dorsey', 'http://www.espn.com/nba/player/gamelog/_/id/4277923/zhaire-smith']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/10
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/4277842/trevon-duval', 'http://www.espn.com/nba/player/gamelog/_/id/6637/kent-bazemore', 'http://www.espn.com/nba/player/gamelog/_/id/2991283/alex-poythress', 'http://www.espn.com/nba/player/gamelog/_/id/3102528/dante-exum', 'http://www.espn.com/nba/player/gamelog/_/id/3913546/melvin-frazier-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/1987/dwyane-wade', 'http://www.espn.com/nba/player/gamelog/_/id/3136194/tyler-ulis', 'http://www.espn.com/nba/player/gamelog/_/id/3137795/chris-chiozza', 'http://www.espn.com/nba/player/gamelog/_/id/3150844/moritz-wagner', 'http://www.espn.com/nba/player/gamelog/_/id/2326411/james-nunnally', 'http://www.espn.com/nba/player/gamelog/_/id/3157465/duncan-robinson', 'http://www.espn.com/nba/player/gamelog/_/id/2489530/troy-daniels', 'http://www.espn.com/nba/player/gamelog/_/id/2578213/ben-mclemore', 'http://www.espn.com/nba/player/gamelog/_/id/2982340/justin-anderson', 'http://www.espn.com/nba/player/gamelog/_/id/3194/wilson-chandler', 'http://www.espn.com/nba/player/gamelog/_/id/3057198/brandon-goodwin', 'http://www.espn.com/nba/player/gamelog/_/id/4066211/robert-williams-iii', 'http://www.espn.com/nba/player/gamelog/_/id/2325499/c.j.-williams', 'http://www.espn.com/nba/player/gamelog/_/id/2595231/treveon-graham', 'http://www.espn.com/nba/player/gamelog/_/id/2990969/georges-niang', 'http://www.espn.com/nba/player/gamelog/_/id/3917378/ivan-rabb', 'http://www.espn.com/nba/player/gamelog/_/id/1975/carmelo-anthony', 'http://www.espn.com/nba/player/gamelog/_/id/3134881/stanley-johnson', 'http://www.espn.com/nba/player/gamelog/_/id/1015/tony-parker', 'http://www.espn.com/nba/player/gamelog/_/id/3412/ryan-anderson', 'http://www.espn.com/nba/player/gamelog/_/id/2488826/rodney-mcgruder', 'http://www.espn.com/nba/player/gamelog/_/id/4278508/troy-brown-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/2284101/justin-holiday', 'http://www.espn.com/nba/player/gamelog/_/id/4065670/bruce-brown', 'http://www.espn.com/nba/player/gamelog/_/id/2528779/reggie-bullock', 'http://www.espn.com/nba/player/gamelog/_/id/6452/jon-leuer', 'http://www.espn.com/nba/player/gamelog/_/id/2778/cj-miles', 'http://www.espn.com/nba/player/gamelog/_/id/6466/chandler-parsons', 'http://www.espn.com/nba/player/gamelog/_/id/2991184/sam-dekker', 'http://www.espn.com/nba/player/gamelog/_/id/3057187/sterling-brown', 'http://www.espn.com/nba/player/gamelog/_/id/3064511/andrew-harrison', 'http://www.espn.com/nba/player/gamelog/_/id/3064291/rondae-hollis-jefferson', 'http://www.espn.com/nba/player/gamelog/_/id/2991149/damyean-dotson', 'http://www.espn.com/nba/player/gamelog/_/id/3133838/yuta-watanabe', 'http://www.espn.com/nba/player/gamelog/_/id/4291678/haywood-highsmith']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/11
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/3155526/dillon-brooks', 'http://www.espn.com/nba/player/gamelog/_/id/4066243/daryl-macon', 'http://www.espn.com/nba/player/gamelog/_/id/2595209/damion-lee', 'http://www.espn.com/nba/player/gamelog/_/id/3064539/sindarius-thornwell', 'http://www.espn.com/nba/player/gamelog/_/id/3132042/gary-clark', 'http://www.espn.com/nba/player/gamelog/_/id/2579321/james-ennis-iii', 'http://www.espn.com/nba/player/gamelog/_/id/4066636/markelle-fultz', 'http://www.espn.com/nba/player/gamelog/_/id/4066262/malik-monk', 'http://www.espn.com/nba/player/gamelog/_/id/3912334/khyri-thomas', "http://www.espn.com/nba/player/gamelog/_/id/6460/e'twaun-moore", 'http://www.espn.com/nba/player/gamelog/_/id/2580898/chasson-randle', 'http://www.espn.com/nba/player/gamelog/_/id/3906671/caleb-swanigan', 'http://www.espn.com/nba/player/gamelog/_/id/609/dirk-nowitzki', 'http://www.espn.com/nba/player/gamelog/_/id/3137730/patrick-mccaw', 'http://www.espn.com/nba/player/gamelog/_/id/4239/evan-turner', 'http://www.espn.com/nba/player/gamelog/_/id/3133874/jaylen-adams', 'http://www.espn.com/nba/player/gamelog/_/id/2531210/allen-crabbe', 'http://www.espn.com/nba/player/gamelog/_/id/2999409/willy-hernangomez', 'http://www.espn.com/nba/player/gamelog/_/id/3078286/troy-williams', 'http://www.espn.com/nba/player/gamelog/_/id/3892818/emmanuel-mudiay', 'http://www.espn.com/nba/player/gamelog/_/id/3934663/malachi-richardson', 'http://www.espn.com/nba/player/gamelog/_/id/2393/shaun-livingston', 'http://www.espn.com/nba/player/gamelog/_/id/3147351/daniel-hamilton', 'http://www.espn.com/nba/player/gamelog/_/id/3915195/shake-milton', 'http://www.espn.com/nba/player/gamelog/_/id/4065649/harry-giles-iii', 'http://www.espn.com/nba/player/gamelog/_/id/4260/greg-monroe', 'http://www.espn.com/nba/player/gamelog/_/id/2490589/isaiah-canaan', 'http://www.espn.com/nba/player/gamelog/_/id/2990962/taurean-prince', 'http://www.espn.com/nba/player/gamelog/_/id/4277905/trae-young', 'http://www.espn.com/nba/player/gamelog/_/id/3138192/bonzie-colson', 'http://www.espn.com/nba/player/gamelog/_/id/3908807/deng-adel', 'http://www.espn.com/nba/player/gamelog/_/id/4277843/gary-trent-jr.', 'http://www.espn.com/nba/player/gamelog/_/id/3444/kosta-koufos', 'http://www.espn.com/nba/player/gamelog/_/id/2382/devin-harris', 'http://www.espn.com/nba/player/gamelog/_/id/6427/bismack-biyombo', 'http://www.espn.com/nba/player/gamelog/_/id/3133602/sviatoslav-mykhailiuk', 'http://www.espn.com/nba/player/gamelog/_/id/3914283/chimezie-metu', 'http://www.espn.com/nba/player/gamelog/_/id/4299/jeremy-lin', 'http://www.espn.com/nba/player/gamelog/_/id/4066334/rawle-alkins', 'http://www.espn.com/nba/player/gamelog/_/id/6461/markieff-morris']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/12
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/6474/tristan-thompson', 'http://www.espn.com/nba/player/gamelog/_/id/6428/marshon-brooks', 'http://www.espn.com/nba/player/gamelog/_/id/3133843/jared-terrell', 'http://www.espn.com/nba/player/gamelog/_/id/2528787/lorenzo-brown', 'http://www.espn.com/nba/player/gamelog/_/id/4230546/terrance-ferguson', 'http://www.espn.com/nba/player/gamelog/_/id/2528426/jordan-clarkson', 'http://www.espn.com/nba/player/gamelog/_/id/3936294/isaiah-briscoe', 'http://www.espn.com/nba/player/gamelog/_/id/2991039/glenn-robinson-iii', 'http://www.espn.com/nba/player/gamelog/_/id/2596108/michael-carter-williams', 'http://www.espn.com/nba/player/gamelog/_/id/2530923/alfonzo-mckinnie', 'http://www.espn.com/nba/player/gamelog/_/id/4351851/anfernee-simons', 'http://www.espn.com/nba/player/gamelog/_/id/3002137/tyrone-wallace', 'http://www.espn.com/nba/player/gamelog/_/id/6472/isaiah-thomas', 'http://www.espn.com/nba/player/gamelog/_/id/3907385/brandon-sampson', 'http://www.espn.com/nba/player/gamelog/_/id/3974/dante-cunningham', 'http://www.espn.com/nba/player/gamelog/_/id/2991274/shaquille-harrison', 'http://www.espn.com/nba/player/gamelog/_/id/3912854/jawun-evans', 'http://www.espn.com/nba/player/gamelog/_/id/2754/channing-frye', 'http://www.espn.com/nba/player/gamelog/_/id/2995706/mario-hezonja', 'http://www.espn.com/nba/player/gamelog/_/id/3149010/chandler-hutchison', "http://www.espn.com/nba/player/gamelog/_/id/4066436/de'anthony-melton", 'http://www.espn.com/nba/player/gamelog/_/id/2530530/t.j.-mcconnell', 'http://www.espn.com/nba/player/gamelog/_/id/6594/john-jenkins', 'http://www.espn.com/nba/player/gamelog/_/id/3943606/jerome-robinson', 'http://www.espn.com/nba/player/gamelog/_/id/4240/avery-bradley', 'http://www.espn.com/nba/player/gamelog/_/id/3136779/keita-bates-diop', 'http://www.espn.com/nba/player/gamelog/_/id/3064427/jordan-bell', 'http://www.espn.com/nba/player/gamelog/_/id/3133635/jevon-carter', 'http://www.espn.com/nba/player/gamelog/_/id/6429/alec-burks', 'http://www.espn.com/nba/player/gamelog/_/id/3136485/edmond-sumner', 'http://www.espn.com/nba/player/gamelog/_/id/3934621/jacob-evans', 'http://www.espn.com/nba/player/gamelog/_/id/2327577/jamychal-green', 'http://www.espn.com/nba/player/gamelog/_/id/2585637/dairis-bertans', 'http://www.espn.com/nba/player/gamelog/_/id/3074797/wes-iwundu', 'http://www.espn.com/nba/player/gamelog/_/id/3934719/og-anunoby', 'http://www.espn.com/nba/player/gamelog/_/id/4066297/josh-jackson', 'http://www.espn.com/nba/player/gamelog/_/id/3026/rajon-rondo', 'http://www.espn.com/nba/player/gamelog/_/id/3136696/wade-baldwin-iv', 'http://www.espn.com/nba/player/gamelog/_/id/4277848/marvin-bagley-iii', 'http://www.espn.com/nba/player/gamelog/_/id/3064230/cameron-payne']
    total links:  40
    total positions:  40
    dataframe size:  40
    https://www.espn.com/nba/statistics/rpm/_/page/13
    1:"RK"
    2:"NAME"
    3:"TEAM"
    4:"GP"
    5:"MPG"
    6:"ORPM"
    7:"DRPM"
    8:"RPM"
    9:"WINS"
    links:  ['http://www.espn.com/nba/player/gamelog/_/id/2489785/ian-clark', 'http://www.espn.com/nba/player/gamelog/_/id/6448/brandon-knight', 'http://www.espn.com/nba/player/gamelog/_/id/6485/lance-thomas', 'http://www.espn.com/nba/player/gamelog/_/id/2488958/solomon-hill', "http://www.espn.com/nba/player/gamelog/_/id/3133601/devonte'-graham", 'http://www.espn.com/nba/player/gamelog/_/id/2566745/quinn-cook', 'http://www.espn.com/nba/player/gamelog/_/id/4011991/dragan-bender', 'http://www.espn.com/nba/player/gamelog/_/id/3907525/allonzo-trier', 'http://www.espn.com/nba/player/gamelog/_/id/4065836/omari-spellman', 'http://www.espn.com/nba/player/gamelog/_/id/3893016/cedi-osman', 'http://www.espn.com/nba/player/gamelog/_/id/4080610/hamidou-diallo', 'http://www.espn.com/nba/player/gamelog/_/id/2595435/abdel-nader', 'http://www.espn.com/nba/player/gamelog/_/id/4264/patrick-patterson', 'http://www.espn.com/nba/player/gamelog/_/id/4247/wesley-johnson', 'http://www.espn.com/nba/player/gamelog/_/id/3113587/cristiano-felicio', 'http://www.espn.com/nba/player/gamelog/_/id/6454/shelvin-mack', 'http://www.espn.com/nba/player/gamelog/_/id/3059315/frank-mason-iii', 'http://www.espn.com/nba/player/gamelog/_/id/4065651/frank-jackson', 'http://www.espn.com/nba/player/gamelog/_/id/4277919/mo-bamba', 'http://www.espn.com/nba/player/gamelog/_/id/2806/jose-calderon', 'http://www.espn.com/nba/player/gamelog/_/id/4230550/elie-okobo', 'http://www.espn.com/nba/player/gamelog/_/id/6579/will-barton', 'http://www.espn.com/nba/player/gamelog/_/id/2579466/jonathon-simmons', 'http://www.espn.com/nba/player/gamelog/_/id/2991042/nik-stauskas', 'http://www.espn.com/nba/player/gamelog/_/id/4230547/frank-ntilikina', 'http://www.espn.com/nba/player/gamelog/_/id/3059316/wayne-selden', 'http://www.espn.com/nba/player/gamelog/_/id/3893019/timothe-luwawu-cabarrot', 'http://www.espn.com/nba/player/gamelog/_/id/4277811/collin-sexton', 'http://www.espn.com/nba/player/gamelog/_/id/3417/jerryd-bayless', 'http://www.espn.com/nba/player/gamelog/_/id/3135045/grayson-allen', 'http://www.espn.com/nba/player/gamelog/_/id/165/jamal-crawford', 'http://www.espn.com/nba/player/gamelog/_/id/3907487/marquese-chriss', 'http://www.espn.com/nba/player/gamelog/_/id/3907386/antonio-blakeney', 'http://www.espn.com/nba/player/gamelog/_/id/4278075/kevin-knox']
    total links:  34
    total positions:  34
    dataframe size:  34
    None





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



## Save data to csv file


```python
rpm_data.to_csv('nba_rpm_data_2018.csv')
```
