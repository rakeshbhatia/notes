```python
import os
import pymysql
import mysql.connector
from mysql.connector import Error
import json
import csv
import time
from time import sleep
import string
import random
import urllib
from urllib.request import Request, urlopen
from itertools import cycle
import traceback
import requests
import numpy
import pandas as pd
import bs4
from bs4 import BeautifulSoup
from lxml import html
from lxml.html import fromstring
from collections import deque
from fake_useragent import UserAgent
from requests.auth import HTTPBasicAuth
from requests_html import HTMLSession, AsyncHTMLSession
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import Select, WebDriverWait
#from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.firefox.options import Options
from selenium.common.exceptions import NoSuchElementException
from IPython.display import display_html
from platform import python_version 
from sqlalchemy import create_engine

print(python_version())
```

    3.6.7



```python
def random_wait():
    time_delay = random.randrange(0, 5)
    print('Waiting', time_delay, 's')
    sleep(time_delay)

def init():
    # Set options
    options = webdriver.ChromeOptions()
    #options.add_argument('--headless')
    options.accept_untrusted_certs = True
    options.add_argument('--ignore-certificate-errors')
    options.add_argument('--test-type')
    
    # Initialize webdriver
    driver = webdriver.Chrome(chrome_options=options)
    driver.get('http://www.matweb.com/search/PropertySearch.aspx')
    
    return driver
    
def select_material_category(d):
    random_wait()
    toggle_metal_button = d.find_element_by_id("ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewt3")
    toggle_metal_button.click()
    
def select_material_properties(d):
    random_wait()
    
    # Select first material property
    material_property_select = Select(d.find_element_by_name("ctl00$ContentMain$ucPropertyDropdown1$drpPropertyList"))
    material_property_select.select_by_value("215")

    random_wait()
        
    material_property_min = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit1$txtpMin")
    material_property_min.clear()
    material_property_min.send_keys("1")
    
    random_wait()
    
    material_property_max = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit1$txtpMax")
    material_property_max.send_keys("20")

    random_wait()
    
    material_property_select = Select(d.find_element_by_name("ctl00$ContentMain$ucPropertyDropdown2$drpPropertyList"))
    material_property_select.select_by_value("263")

    random_wait()

    material_property_min = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit2$txtpMin")
    material_property_min.clear()
    material_property_min.send_keys("0.1")
    
    random_wait()
    
    material_property_max = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit2$txtpMax")
    material_property_max.send_keys("2000")
    
    random_wait()
    
    material_property_select = Select(d.find_element_by_name("ctl00$ContentMain$ucPropertyDropdown3$drpPropertyList"))
    material_property_select.select_by_value("743")
    
    random_wait()
    
    material_property_min = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit3$txtpMin")
    material_property_min.clear()
    material_property_min.send_keys("100")
    
    random_wait()
    
    material_property_max = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit3$txtpMax")
    material_property_max.send_keys("6000")

def submit_query(d):
    random_wait()
    submit_button = d.find_element_by_name("ctl00$ContentMain$btnSubmit")
    submit_button.click()

def select_view(d, desired_view):
    random_wait()
    material_property_select = Select(d.find_element_by_id("ctl00_ContentMain_UcSearchResults1_drpPageSize2"))
    material_property_select.select_by_value(desired_view)

def go_next_page(d):
    random_wait()
    try:
        next_page_button = d.find_element_by_name("ctl00_ContentMain_UcSearchResults1_lnkNextPage2").click()
    except NoSuchElementException:
        next_page_button = None
    return next_page_button

def extract_links(d, links, option):
    random_wait()
    #links = {}
    #links = pd.DataFrame(columns=['material', 'url'])
    
    if option == 1:
        rows = d.find_elements_by_xpath('/html/body/form[2]/div[4]/div[2]/div/table[3]/tbody/tr')
        for row in rows[1:]:
            a = row.find_element_by_tag_name('a')
            links[a.text] = a.get_attribute('href')
            #links.append({'material': a.text, 'url': a.get_attribute('href')}, ignore_index=True)
    elif option == 2:
        soup = BeautifulSoup(d.page_source, 'lxml')
        table = soup.find('table', attrs={'id': 'tblResults'})
        rows = table.find_all('tr')
        print('rows[0]: ', rows[0])
        for row in rows[1:]:
            a = row.find('a')
            print('a.text: ', a.text)
            links[a.text] = 'http://www.matweb.com' + a['href']
            #links.append({'material': a.text, 'url': 'http://www.matweb.com' + a.get_attribute('href')}, ignore_index=True)
    else:
        print('Option must be either 1 or 2.')
    return links

def query_matweb(d):
    # Create dataframe to store all links
    links = {}
    #links = pd.DataFrame(columns=['material', 'url'])
    
    # Select material category
    select_material_category(d)
    
    # Select material properties
    select_material_properties(d)

    # Submit query
    submit_query(d)
    
    # Select max view of 200 per page
    select_view(d, '200')
    
    # Extract first page of material property links
    extract_links(d, links, 2)
    
    # Go to next page and continue
    #while(go_next_page(d)):
    #    extract_links(d, links, 2)
    
    return links

def scrape_material_properties(links, dfs):
    for k, v in links.items():
        #Pick a random user agent
        user_agent = random.choice(user_agent_list)
        #Set the headers 
        headers = {'User-Agent': user_agent}
        
        response = requests.get(v, headers=headers)
        response.raise_for_status()

        print('response.url: ', response.url)

        soup = BeautifulSoup(response.text, 'lxml')
        table = soup.find('table', attrs={'cellspacing': '0'})
        #print('table: ', table)
        #rows = soup.find_all('table').find('tbody').find_all('tr')
        # Extract DataFrame
        print('table.prettify(): ', table.prettify())
        df = pd.read_html(table.prettify())[0]

        # Drop excess rows and columns
        df = df.drop([4, 5], axis=1)
        df = df.drop([0], axis=0)
        dfs[k] = df

def transfer_links_to_db(df):
    # create sqlalchemy engine
    engine = create_engine("mysql+pymysql://{user}:{pw}@localhost/{db}"
                           .format(user="root",
                                   pw="carnival01",
                                   db="decisive_mfg"))

    # Insert whole DataFrame into MySQL
    df.to_sql('matweb_scraped_links', con = engine, if_exists = 'replace', chunksize = 1000)

```


```python
def main():
    print("Matweb Scraper - Querying Links")
    driver = init()
    urls = query_matweb(driver)
    print('urls: \n', urls)
    df = pd.DataFrame([urls])
    print('df.head(): \n', df.head())
    print('df.tail(): \n', df.tail())
    driver.quit()
    #transfer_links_to_db(df)
    dfs = {}
    scrape_material_properties(urls, dfs)

if __name__ == '__main__':
    main()
```

    Matweb Scraper - Querying Links


    /Users/rakeshbhatia/anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:15: DeprecationWarning: use options instead of chrome_options
      from ipykernel import kernelapp as app


    Waiting 4 s
    Waiting 2 s
    Waiting 3 s
    Waiting 1 s
    Waiting 1 s
    Waiting 0 s
    Waiting 3 s
    Waiting 3 s
    Waiting 0 s
    Waiting 4 s
    Waiting 0 s
    Waiting 2 s
    Waiting 0 s
    rows[0]:  <tr class="">
    <th style="width:65px; white-space:nowrap;">Select</th>
    <th style="width:10px;">
    <!--Discontinued and Found Via Interpolated Indicators -->
    </th>
    <th style="width:auto;">
    <a disabled="disabled" id="ctl00_ContentMain_UcSearchResults1_lnkSortMatName" title="Click to Sort By Material Name">Material Name</a>
    <br/>
    </th>
    <th class="propheader" colspan="2" id="ctl00_ContentMain_UcSearchResults1_thProp1">
    <a disabled="disabled" id="ctl00_ContentMain_UcSearchResults1_lnkProp1" title="Click to sort by this property">Density (g/cc)</a><br/>
    </th>
    <th class="propheader" colspan="2" id="ctl00_ContentMain_UcSearchResults1_thProp2">
    <a disabled="disabled" id="ctl00_ContentMain_UcSearchResults1_lnkProp2" title="Click to sort by this property">Elongation at Break (%)</a><br/>
    </th>
    <th class="propheader" colspan="2" id="ctl00_ContentMain_UcSearchResults1_thProp3">
    <a disabled="disabled" id="ctl00_ContentMain_UcSearchResults1_lnkProp3" title="Click to sort by this property">Tensile Strength, Ultimate (MPa)</a><br/>
    </th>
    </tr>
    a.text:  Gold, Au
    a.text:  Beryllium, Be
    a.text:  Calcium, Ca; Rolled
    a.text:  Calcium, Ca; Annealed
    a.text:  Cadmium, Cd
    a.text:  Chromium, Cr; As-Swaged
    a.text:  Chromium, Cr; Recrystallized
    a.text:  Copper, Cu; Annealed
    a.text:  Copper, Cu; Cold Drawn
    a.text:  Dysprosium, Dy
    a.text:  Erbium, Er
    a.text:  Gadolinium, Gd
    a.text:  Hafnium, Hf; Rod
    a.text:  Hafnium, Hf; Plate
    a.text:  Hafnium, Hf; Strip
    a.text:  Lanthanum, La
    a.text:  Lutetium, Lu
    a.text:  Magnesium, Mg; Sand Cast
    a.text:  Magnesium, Mg; Extruded
    a.text:  Magnesium, Mg; Hard-Rolled Sheet
    a.text:  Magnesium, Mg; Annealed Sheet
    a.text:  Molybdenum, Mo, Stress Relieved
    a.text:  Molybdenum, Mo, Recrystallized
    a.text:  Niobium, Nb (Columbium, Cb); Annealed Sample
    a.text:  Niobium, Nb (Columbium, Cb); Wrought
    a.text:  Neodymium, Nd
    a.text:  Nickel, Ni
    a.text:  Palladium, Pd
    a.text:  Promethium, Pm
    a.text:  Praseodymium, Pr
    a.text:  Plutonium, Pu
    a.text:  Scandium, Sc
    a.text:  Samarium, Sm
    a.text:  Tantalum, UNS R05200
    a.text:  Tantalum, UNS R05400
    a.text:  Terbium, Tb
    a.text:  Technetium, Tc; As-Rolled
    a.text:  Technetium, Tc; Annealed
    a.text:  Thorium, Th
    a.text:  Titanium, Ti
    a.text:  Thallium, Tl
    a.text:  Thulium, Tm
    a.text:  Uranium, U; Cast
    a.text:  Uranium, U; Wrought Alpha Phase
    a.text:  Vanadium, V; Cold Rolled
    a.text:  Vanadium, V; Vacuum Annealed Wire
    a.text:  Vanadium, V; Hot Rolled Bar
    a.text:  Vanadium, V; Vacuum Annealed Sheet
    a.text:  Vanadium, V; Cold Drawn Wire
    a.text:  Yttrium, Y; Annealed Rod
    urls: 
     {'Gold, Au': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e', 'Beryllium, Be': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8a6a0df6122349b7bdc92662658d4a4f', 'Calcium, Ca; Rolled': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=15135dd30619457f902229b03619841a', 'Calcium, Ca; Annealed': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=0c5b857dfabb4920a539103f32386103', 'Cadmium, Cd': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=ca862f5c59594be3b9a2d250460d2dba', 'Chromium, Cr; As-Swaged': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=25ce9b7f40364cf79d54ed0db5c8e41f', 'Chromium, Cr; Recrystallized': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=e15e12ccafc949078b2e19e3942e0200', 'Copper, Cu; Annealed': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=9aebe83845c04c1db5126fada6f76f7e', 'Copper, Cu; Cold Drawn': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1980eb23287a4408adc404dd39293942', 'Dysprosium, Dy': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8cdce331dd324e80ab4a9fd801466a92', 'Erbium, Er': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1fc9adc121df4b33a4263065d602a4e1', 'Gadolinium, Gd': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=750a3dd8d69b44b79468fbaf72a2beef', 'Hafnium, Hf; Rod': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd', 'Hafnium, Hf; Plate': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=129f6c5f72db498eaee987ef6870c52d', 'Hafnium, Hf; Strip': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=7bfa1bc5f7f04e41934cb1e48a8c112b', 'Lanthanum, La': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=092dbe0933ff4b25a4bfceaa24f9e81b', 'Lutetium, Lu': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=eef75b3432c94cb7a5b34eff09bfcbb0', 'Magnesium, Mg; Sand Cast': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=ff6d4e6d529e4b3d97c77d6538b29693', 'Magnesium, Mg; Extruded': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=dd0594df641645d6a48dae83f3caf6d0', 'Magnesium, Mg; Hard-Rolled Sheet': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=31abaf783eb64e41bd9d74b18ca242ae', 'Magnesium, Mg; Annealed Sheet': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=7b49605d472d40d393ffe87ea224980c', 'Molybdenum, Mo, Stress Relieved': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=ac6761febc3a43c0817ce38f6f5f526c', 'Molybdenum, Mo, Recrystallized': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=20341f89fd8e43f1995b4b9a2a8d9dbe', 'Niobium, Nb (Columbium, Cb); Annealed Sample': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=63dbf7fcefbd42b99a190cad7e109056', 'Niobium, Nb (Columbium, Cb); Wrought': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=fa7e71660515494d8a5d728653681c01', 'Neodymium, Nd': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=61dbbf5b1f34464984735709ef1ee38c', 'Nickel, Ni': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=e6eb83327e534850a062dbca3bc758dc', 'Palladium, Pd': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=5cab8ad368a44544affae0b71f967b3d', 'Promethium, Pm': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=2289354b7b07483bb495a93d52280762', 'Praseodymium, Pr': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=4c9fbea67a6347e0a43f6b7da54a78f1', 'Plutonium, Pu': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=57def1ede7f94ef882d73d27653ee76e', 'Scandium, Sc': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=d1e54a8ff92d48e18f94ff58c4fc9c4a', 'Samarium, Sm': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=121d3f8172f147ada769e4d90c8710d8', 'Tantalum, UNS R05200': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=65fa537b18ae421db9c25e61e8f7eb54', 'Tantalum, UNS R05400': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=7d664b2d02034fa9b8873f9dd828000d', 'Terbium, Tb': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=6041c127d1a24724b21ff69551e57576', 'Technetium, Tc; As-Rolled': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=c4a0b09ca0ec42ccbb7e5f4281f6d7aa', 'Technetium, Tc; Annealed': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=2629eceea53a4c0dbb9279fac6e6007d', 'Thorium, Th': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=c29823b5976f4c319cabe0ae8c4075c3', 'Titanium, Ti': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=66a15d609a3f4c829cb6ad08f0dafc01', 'Thallium, Tl': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=00f69978bc284635ae39c34271133bb4', 'Thulium, Tm': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=dac0f38bfebc4bac8819a10780ffc31f', 'Uranium, U; Cast': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=6311bc909b6a42c8b7a75173245309a4', 'Uranium, U; Wrought Alpha Phase': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=d483c5a8a0fe46099ee1950cee466916', 'Vanadium, V; Cold Rolled': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=2dc2a76245cd4c1caf9aacec8a9bf6dd', 'Vanadium, V; Vacuum Annealed Wire': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=00c6b423cca04269a3aa2572a8c38854', 'Vanadium, V; Hot Rolled Bar': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=6c7427744ca8484f94cf8bd83823ade0', 'Vanadium, V; Vacuum Annealed Sheet': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=0b63679b24784b40a8c1ac69bea8b9be', 'Vanadium, V; Cold Drawn Wire': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1e90689a21bc4ba59f2037b7f9513ac8', 'Yttrium, Y; Annealed Rod': 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=255acda1c0e24a179327e44b26d78652'}
    df.head(): 
                                                 Gold, Au  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                           Beryllium, Be  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                     Calcium, Ca; Rolled  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                   Calcium, Ca; Annealed  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                             Cadmium, Cd  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                 Chromium, Cr; As-Swaged  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                            Chromium, Cr; Recrystallized  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                    Copper, Cu; Annealed  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                  Copper, Cu; Cold Drawn  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                          Dysprosium, Dy  ...  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...  ...   
    
                                            Thallium, Tl  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                             Thulium, Tm  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                        Uranium, U; Cast  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                         Uranium, U; Wrought Alpha Phase  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                Vanadium, V; Cold Rolled  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                       Vanadium, V; Vacuum Annealed Wire  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                             Vanadium, V; Hot Rolled Bar  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                      Vanadium, V; Vacuum Annealed Sheet  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                            Vanadium, V; Cold Drawn Wire  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                Yttrium, Y; Annealed Rod  
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...  
    
    [1 rows x 50 columns]
    df.tail(): 
                                                 Gold, Au  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                           Beryllium, Be  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                     Calcium, Ca; Rolled  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                   Calcium, Ca; Annealed  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                             Cadmium, Cd  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                 Chromium, Cr; As-Swaged  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                            Chromium, Cr; Recrystallized  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                    Copper, Cu; Annealed  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                  Copper, Cu; Cold Drawn  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                          Dysprosium, Dy  ...  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...  ...   
    
                                            Thallium, Tl  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                             Thulium, Tm  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                        Uranium, U; Cast  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                         Uranium, U; Wrought Alpha Phase  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                Vanadium, V; Cold Rolled  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                       Vanadium, V; Vacuum Annealed Wire  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                             Vanadium, V; Hot Rolled Bar  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                      Vanadium, V; Vacuum Annealed Sheet  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                            Vanadium, V; Cold Drawn Wire  \
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...   
    
                                Yttrium, Y; Annealed Rod  
    0  http://www.matweb.com/search/DataSheet.aspx?Ma...  
    
    [1 rows x 50 columns]



```python
# create sqlalchemy engine
engine = create_engine("mysql+pymysql://{user}:{pw}@localhost/{db}"
                       .format(user="root",
                               pw="carnival01",
                               db="decisive_mfg"))

# Insert whole DataFrame into MySQL
df.to_sql('matweb_scraped_links', con = engine, if_exists = 'replace', chunksize = 1000)
```


```python
# Open database connection
db = pymysql.connect("localhost", "root", "carnival01", "decisive_mfg" )

print("Connection established sucessfully")

# prepare a cursor object using cursor() method
cursor = db.cursor()

cursor.execute("SHOW TABLES")

# Drop table if it already exist using execute() method.
cursor.execute("DROP TABLE IF EXISTS matweb_scraped_links")

print("Creating table matweb_scraped_links")

# Create table as per requirement
sql = """CREATE TABLE matweb_scraped_links (
   id INT AUTO_INCREMENT PRIMARY KEY,
   url  CHAR(255))"""

cursor.execute(sql)

# Prepare SQL query to INSERT a record into the database.
sql = """INSERT INTO matweb_scraped_links(url)
   VALUES ('http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd')"""

try:
   # Execute the SQL command
   cursor.execute(sql)
   # Commit your changes in the database
   db.commit()
except:
   # Rollback in case there is any error
   db.rollback()

# disconnect from server
db.close()
```

    Connection established sucessfully
    Creating table matweb_scraped_links



```python
# Open database connection
db = pymysql.connect("localhost","root","carnival01","decisive_mfg" )

print("Connection established sucessfully")

# prepare a cursor object using cursor() method
cursor = db.cursor()

cursor.execute("SHOW TABLES")

#Fetch all the rows
rows = cursor.fetchall()

for row in rows:
    print(row)

# disconnect from server
db.close()
```

    Connection established sucessfully
    ('matweb_scraped_links',)
    ('query_links',)



```python
# Open database connection
db = pymysql.connect("localhost","root","carnival01","decisive_mfg" )

print("Connection established sucessfully")

# Create a Cursor object to execute queries.
cursor = db.cursor()

# Select data from table using SQL query.
cursor.execute("SELECT * FROM matweb_scraped_links")

# print the first and second columns      
for row in cursor.fetchall() :
    print(row[0], " ", row[1])
```

    Connection established sucessfully
    1   http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd



```python
#!/usr/bin/python3

import pymysql

# Open database connection
db = pymysql.connect("localhost","root","carnival01","decisive_mfg")
print("Connection established sucessfully")

# prepare a cursor object using cursor() method
cursor = db.cursor()

# execute SQL query using execute() method.
cursor.execute("SELECT VERSION()")

# Fetch a single row using fetchone() method.
data = cursor.fetchone()
print ("Database version : %s " % data)

# disconnect from server
db.close()
```

    Connection established sucessfully
    Database version : 8.0.11 



```python
#!/usr/bin/python3

import pymysql

# Open database connection
db = pymysql.connect("localhost","root","carnival01","decisive_mfg" )

# prepare a cursor object using cursor() method
cursor = db.cursor()

# Prepare SQL query to INSERT a record into the database.
sql = """INSERT INTO query_links(url)
   VALUES (http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd)"""
try:
   # Execute the SQL command
   cursor.execute(sql)
   # Commit your changes in the database
   db.commit()
except:
   # Rollback in case there is any error
   db.rollback()

# disconnect from server
db.close()
```


```python
def main():
    delay = 10
    time_delay = random.randrange(0, delay)
    time.sleep(time_delay)
    
    #Pick a random user agent
    user_agent = random.choice(user_agent_list)
    #Set the headers 
    headers = {'User-Agent': user_agent}

    url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd'

    response = requests.get(url, headers=headers)
    response.raise_for_status()
    
    print('response.url: ', response.url)
    
    soup = BeautifulSoup(response.text, 'lxml')
    table = soup.find('table', attrs={'cellspacing': '0'})
    print('table: ', table)
    #rows = soup.find_all('table').find('tbody').find_all('tr')
    # Extract DataFrame
    print('table.prettify(): ', table.prettify())
    df = pd.read_html(table.prettify())[0]

    # Drop excess rows and columns
    df = df.drop([4, 5], axis=1)
    df = df.drop([0], axis=0)
    df
    
    print('df.head(20): ', df.head(20))
    
if __name__ == '__main__':
    main()
```

    response.url:  http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd&ckck=1
    table:  <table cellspacing="0" class="tabledataformat"><tr><td colspan="4"> </td></tr><tr><th>Physical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Density </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=43&amp;fromValue=13.31" title="Click to see this value in other UOMs">13.31</a> g/cc<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=87&amp;fromValue=0.4809" title="Click to see this value in other UOMs">0.4809</a> lb/in³<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Chemical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Atomic Mass </td><td class="dataCell" style="vertical-align:top;">178.49<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">178.49<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">1995</td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Atomic Number </td><td class="dataCell" style="vertical-align:top;">72<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">72<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">X-ray Absorption Edge </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.18981" title="Click to see this value in other UOMs">0.18981</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.18981" title="Click to see this value in other UOMs">0.18981</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">K</td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.09864" title="Click to see this value in other UOMs">1.09864</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.09864" title="Click to see this value in other UOMs">1.09864</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">L<sub>I</sub></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.15311" title="Click to see this value in other UOMs">1.15311</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.15311" title="Click to see this value in other UOMs">1.15311</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">L<sub>II</sub></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.2957" title="Click to see this value in other UOMs">1.2957</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.2957" title="Click to see this value in other UOMs">1.2957</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">L<sub>III</sub></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Electronegativity </td><td class="dataCell" style="vertical-align:top;">1.3<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">1.3<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">Pauling</td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Ionic Radius </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.780" title="Click to see this value in other UOMs">0.780</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.780" title="Click to see this value in other UOMs">0.780</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">Crystal Ionic Radius for Valence +4</td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Mechanical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Tensile Strength, Ultimate </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=485" title="Click to see this value in other UOMs">485</a> MPa<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=70300" title="Click to see this value in other UOMs">70300</a> psi<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;"> <a href="/search/GraphConditionalData.aspx?matguid=a438ce15c0964513aa34c7973fce82fd&amp;propid=743&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=310" title="Click to see this value in other UOMs">310</a> MPa<span class="dataCondition"><br/>@Temperature 315 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=45000" title="Click to see this value in other UOMs">45000</a> psi<span class="dataCondition"><br/>@Temperature 599 °F</span></td><td class="dataComment" style="vertical-align:top;">Longitudinal</td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Tensile Strength, Yield <a href="/search/GraphConditionalData.aspx?matguid=a438ce15c0964513aa34c7973fce82fd&amp;propid=745&amp;sigid=31" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=125" title="Click to see this value in other UOMs">125</a> MPa<span class="dataCondition"><br/>@Strain 0.0200 %,<br/> Temperature 315 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=18100" title="Click to see this value in other UOMs">18100</a> psi<span class="dataCondition"><br/>@Strain 0.0200 %,<br/> Temperature 599 °F</span></td><td class="dataComment" style="vertical-align:top;">Longitudinal</td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=240" title="Click to see this value in other UOMs">240</a> MPa<span class="dataCondition"><br/>@Strain 0.0200 %,<br/> Temperature 23.0 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=34800" title="Click to see this value in other UOMs">34800</a> psi<span class="dataCondition"><br/>@Strain 0.0200 %,<br/> Temperature 73.4 °F</span></td><td class="dataComment" style="vertical-align:top;">Longitudinal</td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Elongation at Break </td><td class="dataCell" style="vertical-align:top;">25 %<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">25 %<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Electrical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Electrical Resistivity </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.0000355" title="Click to see this value in other UOMs">0.0000355</a> ohm-cm<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.0000355" title="Click to see this value in other UOMs">0.0000355</a> ohm-cm<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Magnetic Susceptibility </td><td class="dataCell" style="vertical-align:top;">4.2e-7<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">4.2e-7<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">cgs/g</td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Critical Magnetic Field Strength, Oersted </td><td class="dataCell" style="vertical-align:top;">12.7<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">12.7<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Critical Superconducting Temperature </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=69&amp;fromValue=0.128" title="Click to see this value in other UOMs">0.128</a> K<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=69&amp;fromValue=0.128" title="Click to see this value in other UOMs">0.128</a> K<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Thermal Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">CTE, linear </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=5.90" title="Click to see this value in other UOMs">5.90</a> µm/m-°C<span class="dataCondition"><br/>@Temperature 20.0 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=3.28" title="Click to see this value in other UOMs">3.28</a> µin/in-°F<span class="dataCondition"><br/>@Temperature 68.0 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Specific Heat Capacity </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.144" title="Click to see this value in other UOMs">0.144</a> J/g-°C<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0344" title="Click to see this value in other UOMs">0.0344</a> BTU/lb-°F<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Thermal Conductivity </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=136&amp;fromValue=22.0" title="Click to see this value in other UOMs">22.0</a> W/m-K<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=10&amp;fromValue=153" title="Click to see this value in other UOMs">153</a> BTU-in/hr-ft²-°F<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Melting Point </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=2207" title="Click to see this value in other UOMs">2207</a> - <a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=2247" title="Click to see this value in other UOMs">2247</a> °C<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=4005" title="Click to see this value in other UOMs">4005</a> - <a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=4077" title="Click to see this value in other UOMs">4077</a> °F<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Boiling Point </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=3100" title="Click to see this value in other UOMs">3100</a> °C<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=5610" title="Click to see this value in other UOMs">5610</a> °F<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Component Elements Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Hafnium, Hf </td><td class="dataCell" style="vertical-align:top;">100 %<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">100 %<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr><td colspan="4"> </td></tr><tr><th align="left" class="" colspan="6">Descriptive Properties</th></tr><tr class="altrow"><td>Alpha phase Crystal Structure</td><td class="dataCell" colspan="2">hcp</td><td class="dataComment" colspan="3">&lt; 1760°C</td></tr><tr class=""><td>Atomic weight</td><td class="dataCell" colspan="2">178.5</td><td class="dataComment" colspan="3"></td></tr><tr class="altrow"><td>Beta Phase Crystal Structure</td><td class="dataCell" colspan="2">bcc</td><td class="dataComment" colspan="3">&gt; 1760°C</td></tr><tr class=""><td>CAS Number</td><td class="dataCell" colspan="2">7440-58-6</td><td class="dataComment" colspan="3"></td></tr></table>
    table.prettify():  <table cellspacing="0" class="tabledataformat">
     <tr>
      <td colspan="4">
      </td>
     </tr>
     <tr>
      <th>
       Physical Properties
      </th>
      <th class="dataCell">
       Metric
      </th>
      <th class="dataCell">
       English
      </th>
      <th class="dataCell">
       Comments
      </th>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Density
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=43&amp;fromValue=13.31" title="Click to see this value in other UOMs">
        13.31
       </a>
       g/cc
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=87&amp;fromValue=0.4809" title="Click to see this value in other UOMs">
        0.4809
       </a>
       lb/in³
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr>
      <td colspan="4">
      </td>
     </tr>
     <tr>
      <th>
       Chemical Properties
      </th>
      <th class="dataCell">
       Metric
      </th>
      <th class="dataCell">
       English
      </th>
      <th class="dataCell">
       Comments
      </th>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Atomic Mass
      </td>
      <td class="dataCell" style="vertical-align:top;">
       178.49
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       178.49
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       1995
      </td>
     </tr>
     <tr class="datarowSeparator">
      <td style="vertical-align:top;">
       Atomic Number
      </td>
      <td class="dataCell" style="vertical-align:top;">
       72
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       72
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       X-ray Absorption Edge
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.18981" title="Click to see this value in other UOMs">
        0.18981
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.18981" title="Click to see this value in other UOMs">
        0.18981
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       K
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.09864" title="Click to see this value in other UOMs">
        1.09864
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.09864" title="Click to see this value in other UOMs">
        1.09864
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       L
       <sub>
        I
       </sub>
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.15311" title="Click to see this value in other UOMs">
        1.15311
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.15311" title="Click to see this value in other UOMs">
        1.15311
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       L
       <sub>
        II
       </sub>
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.2957" title="Click to see this value in other UOMs">
        1.2957
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.2957" title="Click to see this value in other UOMs">
        1.2957
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       L
       <sub>
        III
       </sub>
      </td>
     </tr>
     <tr class="datarowSeparator">
      <td style="vertical-align:top;">
       Electronegativity
      </td>
      <td class="dataCell" style="vertical-align:top;">
       1.3
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       1.3
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       Pauling
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Ionic Radius
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.780" title="Click to see this value in other UOMs">
        0.780
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.780" title="Click to see this value in other UOMs">
        0.780
       </a>
       Å
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       Crystal Ionic Radius for Valence +4
      </td>
     </tr>
     <tr>
      <td colspan="4">
      </td>
     </tr>
     <tr>
      <th>
       Mechanical Properties
      </th>
      <th class="dataCell">
       Metric
      </th>
      <th class="dataCell">
       English
      </th>
      <th class="dataCell">
       Comments
      </th>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Tensile Strength, Ultimate
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=485" title="Click to see this value in other UOMs">
        485
       </a>
       MPa
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=70300" title="Click to see this value in other UOMs">
        70300
       </a>
       psi
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       <a href="/search/GraphConditionalData.aspx?matguid=a438ce15c0964513aa34c7973fce82fd&amp;propid=743&amp;sigid=1" title="Graph this set of conditional data">
        <img alt="" src="/images/smallchart.gif"/>
       </a>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=310" title="Click to see this value in other UOMs">
        310
       </a>
       MPa
       <span class="dataCondition">
        <br/>
        @Temperature 315 °C
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=45000" title="Click to see this value in other UOMs">
        45000
       </a>
       psi
       <span class="dataCondition">
        <br/>
        @Temperature 599 °F
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       Longitudinal
      </td>
     </tr>
     <tr class="datarowSeparator">
      <td style="vertical-align:top;">
       Tensile Strength, Yield
       <a href="/search/GraphConditionalData.aspx?matguid=a438ce15c0964513aa34c7973fce82fd&amp;propid=745&amp;sigid=31" title="Graph this set of conditional data">
        <img alt="" src="/images/smallchart.gif"/>
       </a>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=125" title="Click to see this value in other UOMs">
        125
       </a>
       MPa
       <span class="dataCondition">
        <br/>
        @Strain 0.0200 %,
        <br/>
        Temperature 315 °C
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=18100" title="Click to see this value in other UOMs">
        18100
       </a>
       psi
       <span class="dataCondition">
        <br/>
        @Strain 0.0200 %,
        <br/>
        Temperature 599 °F
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       Longitudinal
      </td>
     </tr>
     <tr class="datarowSeparator">
      <td style="vertical-align:top;">
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=240" title="Click to see this value in other UOMs">
        240
       </a>
       MPa
       <span class="dataCondition">
        <br/>
        @Strain 0.0200 %,
        <br/>
        Temperature 23.0 °C
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=34800" title="Click to see this value in other UOMs">
        34800
       </a>
       psi
       <span class="dataCondition">
        <br/>
        @Strain 0.0200 %,
        <br/>
        Temperature 73.4 °F
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       Longitudinal
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Elongation at Break
      </td>
      <td class="dataCell" style="vertical-align:top;">
       25 %
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       25 %
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr>
      <td colspan="4">
      </td>
     </tr>
     <tr>
      <th>
       Electrical Properties
      </th>
      <th class="dataCell">
       Metric
      </th>
      <th class="dataCell">
       English
      </th>
      <th class="dataCell">
       Comments
      </th>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Electrical Resistivity
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.0000355" title="Click to see this value in other UOMs">
        0.0000355
       </a>
       ohm-cm
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.0000355" title="Click to see this value in other UOMs">
        0.0000355
       </a>
       ohm-cm
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr class="datarowSeparator">
      <td style="vertical-align:top;">
       Magnetic Susceptibility
      </td>
      <td class="dataCell" style="vertical-align:top;">
       4.2e-7
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       4.2e-7
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
       cgs/g
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Critical Magnetic Field Strength, Oersted
      </td>
      <td class="dataCell" style="vertical-align:top;">
       12.7
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       12.7
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr class="datarowSeparator">
      <td style="vertical-align:top;">
       Critical Superconducting Temperature
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=69&amp;fromValue=0.128" title="Click to see this value in other UOMs">
        0.128
       </a>
       K
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=69&amp;fromValue=0.128" title="Click to see this value in other UOMs">
        0.128
       </a>
       K
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr>
      <td colspan="4">
      </td>
     </tr>
     <tr>
      <th>
       Thermal Properties
      </th>
      <th class="dataCell">
       Metric
      </th>
      <th class="dataCell">
       English
      </th>
      <th class="dataCell">
       Comments
      </th>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       CTE, linear
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=5.90" title="Click to see this value in other UOMs">
        5.90
       </a>
       µm/m-°C
       <span class="dataCondition">
        <br/>
        @Temperature 20.0 °C
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=3.28" title="Click to see this value in other UOMs">
        3.28
       </a>
       µin/in-°F
       <span class="dataCondition">
        <br/>
        @Temperature 68.0 °F
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr class="datarowSeparator">
      <td style="vertical-align:top;">
       Specific Heat Capacity
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.144" title="Click to see this value in other UOMs">
        0.144
       </a>
       J/g-°C
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0344" title="Click to see this value in other UOMs">
        0.0344
       </a>
       BTU/lb-°F
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Thermal Conductivity
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=136&amp;fromValue=22.0" title="Click to see this value in other UOMs">
        22.0
       </a>
       W/m-K
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=10&amp;fromValue=153" title="Click to see this value in other UOMs">
        153
       </a>
       BTU-in/hr-ft²-°F
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr class="datarowSeparator">
      <td style="vertical-align:top;">
       Melting Point
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=2207" title="Click to see this value in other UOMs">
        2207
       </a>
       -
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=2247" title="Click to see this value in other UOMs">
        2247
       </a>
       °C
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=4005" title="Click to see this value in other UOMs">
        4005
       </a>
       -
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=4077" title="Click to see this value in other UOMs">
        4077
       </a>
       °F
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Boiling Point
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=3100" title="Click to see this value in other UOMs">
        3100
       </a>
       °C
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       <a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=5610" title="Click to see this value in other UOMs">
        5610
       </a>
       °F
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr>
      <td colspan="4">
      </td>
     </tr>
     <tr>
      <th>
       Component Elements Properties
      </th>
      <th class="dataCell">
       Metric
      </th>
      <th class="dataCell">
       English
      </th>
      <th class="dataCell">
       Comments
      </th>
     </tr>
     <tr class="altrow datarowSeparator">
      <td style="vertical-align:top;">
       Hafnium, Hf
      </td>
      <td class="dataCell" style="vertical-align:top;">
       100 %
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataCell" style="vertical-align:top;">
       100 %
       <span class="dataCondition">
       </span>
      </td>
      <td class="dataComment" style="vertical-align:top;">
      </td>
     </tr>
     <tr>
      <td colspan="4">
      </td>
     </tr>
     <tr>
      <th align="left" class="" colspan="6">
       Descriptive Properties
      </th>
     </tr>
     <tr class="altrow">
      <td>
       Alpha phase Crystal Structure
      </td>
      <td class="dataCell" colspan="2">
       hcp
      </td>
      <td class="dataComment" colspan="3">
       &lt; 1760°C
      </td>
     </tr>
     <tr class="">
      <td>
       Atomic weight
      </td>
      <td class="dataCell" colspan="2">
       178.5
      </td>
      <td class="dataComment" colspan="3">
      </td>
     </tr>
     <tr class="altrow">
      <td>
       Beta Phase Crystal Structure
      </td>
      <td class="dataCell" colspan="2">
       bcc
      </td>
      <td class="dataComment" colspan="3">
       &gt; 1760°C
      </td>
     </tr>
     <tr class="">
      <td>
       CAS Number
      </td>
      <td class="dataCell" colspan="2">
       7440-58-6
      </td>
      <td class="dataComment" colspan="3">
      </td>
     </tr>
    </table>
    
    df.head(20):                               0  \
    1          Physical Properties   
    2                      Density   
    3                          NaN   
    4          Chemical Properties   
    5                  Atomic Mass   
    6                Atomic Number   
    7        X-ray Absorption Edge   
    8                          NaN   
    9                          NaN   
    10                         NaN   
    11           Electronegativity   
    12                Ionic Radius   
    13                         NaN   
    14       Mechanical Properties   
    15  Tensile Strength, Ultimate   
    16                         NaN   
    17     Tensile Strength, Yield   
    18                         NaN   
    19         Elongation at Break   
    20                         NaN   
    
                                                       1  \
    1                                             Metric   
    2                                        13.31  g/cc   
    3                                                NaN   
    4                                             Metric   
    5                                             178.49   
    6                                                 72   
    7                                         0.18981  Å   
    8                                         1.09864  Å   
    9                                         1.15311  Å   
    10                                         1.2957  Å   
    11                                               1.3   
    12                                          0.780  Å   
    13                                               NaN   
    14                                            Metric   
    15                                          485  MPa   
    16                     310  MPa  @Temperature 315 °C   
    17   125  MPa  @Strain 0.0200 %,  Temperature 315 °C   
    18  240  MPa  @Strain 0.0200 %,  Temperature 23.0 °C   
    19                                              25 %   
    20                                               NaN   
    
                                                        2  \
    1                                             English   
    2                                      0.4809  lb/in³   
    3                                                 NaN   
    4                                             English   
    5                                              178.49   
    6                                                  72   
    7                                          0.18981  Å   
    8                                          1.09864  Å   
    9                                          1.15311  Å   
    10                                          1.2957  Å   
    11                                                1.3   
    12                                           0.780  Å   
    13                                                NaN   
    14                                            English   
    15                                         70300  psi   
    16                    45000  psi  @Temperature 599 °F   
    17  18100  psi  @Strain 0.0200 %,  Temperature 599 °F   
    18  34800  psi  @Strain 0.0200 %,  Temperature 73....   
    19                                               25 %   
    20                                                NaN   
    
                                          3  
    1                              Comments  
    2                                   NaN  
    3                                   NaN  
    4                              Comments  
    5                                  1995  
    6                                   NaN  
    7                                     K  
    8                                  L  I  
    9                                 L  II  
    10                               L  III  
    11                              Pauling  
    12  Crystal Ionic Radius for Valence +4  
    13                                  NaN  
    14                             Comments  
    15                                  NaN  
    16                         Longitudinal  
    17                         Longitudinal  
    18                         Longitudinal  
    19                                  NaN  
    20                                  NaN  



```python
def scrape_matweb(url):
    delay = 10
    time_delay = random.randrange(0, delay)
    time.sleep(time_delay)
    
    #Pick a random user agent
    user_agent = random.choice(user_agent_list)
    #Set the headers 
    headers = {'User-Agent': user_agent}
    
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e'
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8a6a0df6122349b7bdc92662658d4a4f'
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=15135dd30619457f902229b03619841a'
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=25ce9b7f40364cf79d54ed0db5c8e41f'
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=9aebe83845c04c1db5126fada6f76f7e'
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1980eb23287a4408adc404dd39293942'
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8cdce331dd324e80ab4a9fd801466a92'
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1fc9adc121df4b33a4263065d602a4e1'
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=750a3dd8d69b44b79468fbaf72a2beef'
    #matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd'

    #Make the request
    response = requests.get(url, headers=headers)

    # If the response was successful, no Exception will be raised
    response.raise_for_status()

    print('response.url: ', response.url)

    soup = BeautifulSoup(response.text, 'lxml')

    title = soup.find('title')
    print('title.text: ', title.text)
    table = soup.find('table', attrs={'cellspacing': '0'})
    #print('table: ', table)
    rows = table.find_all('tr')
    print('rows[2].text: ', rows[2].text)

    df1 = pd.DataFrame(data={'Material': [title.text.strip()]})
    print('df1.head(): ', df1.head())
    #df1['Material'] = title.text.strip()
    #df1.loc[:, 'Material'] = title.text.strip()
    #print('df1.head(): ', df1.head())

    for row in rows:
        cells = row.find_all('td')
        #print(cells[0].contents[-1])
        #if len(cells) > 1 and len(cells[0]) == 1:
        #    print('blank')
        if len(cells) > 1 and cells[0].text != '':
            if cells[0].text != cells[-3].text:
                #print('cells: ', cells)
                #cells[0] contains the property name
                #cells[1] contains the main property measurement
                #print('cells[0].text: ', cells[0].text)
                #print('cells[-3].text: ', cells[-3].text)
                #print('cells[-2].text: ', cells[-2].text)
                #print('cells[-1].text: ', cells[-1].text)
                df1[cells[0].text + '- Metric'] = cells[-3].text
                df1[cells[0].text + '- English'] = cells[-2].text
                df1[cells[0].text + '- Comments'] = cells[-1].text
            else:
                df1[cells[0].text] = cells[-2].text
                df1[cells[0].text + '- Comments'] = cells[-1].text

    # Extract DataFrame
    #df = pd.read_html(table.prettify())[0]
    #df = df.drop([4, 5], axis=1)
    #df = df.drop([0], axis=0)
    #df.head()
    
    print('df1.head(): ', df1.head())
    
    return df1
```


```python
def main():
    # Test urls
    urls = [
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8a6a0df6122349b7bdc92662658d4a4f',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=15135dd30619457f902229b03619841a',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=25ce9b7f40364cf79d54ed0db5c8e41f',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=9aebe83845c04c1db5126fada6f76f7e'
    ]
    '''urls = [
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1980eb23287a4408adc404dd39293942',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8cdce331dd324e80ab4a9fd801466a92',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1fc9adc121df4b33a4263065d602a4e1',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=750a3dd8d69b44b79468fbaf72a2beef',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd'
    ]'''
    
    idx = 0
    for url in urls:
        if idx == 0:
            df = scrape_matweb(url)
        else:
            df = df + scrape_matweb(url)
        idx += 1
    
    #df = df.reset_index()
    df.to_csv('scrape_matweb_results.csv')
    print('df.head(): ', df.head())
    
if __name__ == '__main__':
    main()
```

    response.url:  http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e&ckck=1
    title.text:  
    	Gold, Au
    
    rows[2].text:  Density 19.32 g/cc0.6980 lb/in³
    df1.head():     Material
    0  Gold, Au
    df1.head():     Material Density - Metric Density - English Density - Comments  \
    0  Gold, Au       19.32 g/cc     0.6980 lb/in³                      
    
               Vapor Pressure  - Metric        Vapor Pressure  - English  \
    0  0.001013 bar@Temperature 1770 °C  0.7598 torr@Temperature 3220 °F   
    
      Vapor Pressure  - Comments  - Metric  - English  - Comments  ...  \
    0                                 0.85       0.85      600 nm  ...   
    
      Emissivity (0-1) - English Emissivity (0-1) - Comments  \
    0   0.040@Temperature 212 °F              total spectrum   
    
      Reflection Coefficient, Visible (0-1) - Metric  \
    0                                           0.27   
    
      Reflection Coefficient, Visible (0-1) - English  \
    0                                            0.27   
    
      Reflection Coefficient, Visible (0-1) - Comments Gold, Au - Metric  \
    0                                           400 nm             100 %   
    
      Gold, Au - English Gold, Au - Comments CAS Number CAS Number- Comments  
    0              100 %                      7440-57-5                       
    
    [1 rows x 90 columns]
    response.url:  http://www.matweb.com/search/DataSheet.aspx?MatGUID=8a6a0df6122349b7bdc92662658d4a4f&ckck=1
    title.text:  
    	Beryllium, Be
    
    rows[2].text:  Density 1.844 g/cc0.06662 lb/in³
    df1.head():          Material
    0  Beryllium, Be
    df1.head():          Material Density - Metric Density - English Density - Comments  \
    0  Beryllium, Be       1.844 g/cc    0.06662 lb/in³                      
    
      Atomic Mass - Metric Atomic Mass - English Atomic Mass - Comments  \
    0             9.012182              9.012182                   1995   
    
      Atomic Number - Metric Atomic Number - English Atomic Number - Comments  \
    0                      4                       4                            
    
       ... Emissivity (0-1) - English Emissivity (0-1) - Comments  \
    0  ...                       0.61                      650 nm   
    
      Reflection Coefficient, Visible (0-1) - Metric  \
    0                                           0.50   
    
      Reflection Coefficient, Visible (0-1) - English  \
    0                                            0.50   
    
      Reflection Coefficient, Visible (0-1) - Comments Beryllium, Be - Metric  \
    0            50% visible; 55% UV; 98% IR (10.6 µm)                  100 %   
    
      Beryllium, Be - English Beryllium, Be - Comments CAS Number  \
    0                   100 %                           7440-41-7   
    
      CAS Number- Comments  
    0                       
    
    [1 rows x 105 columns]
    response.url:  http://www.matweb.com/search/DataSheet.aspx?MatGUID=15135dd30619457f902229b03619841a&ckck=1
    title.text:  
    	Calcium, Ca; Rolled
    
    rows[2].text:  Density 1.54 g/cc0.0556 lb/in³
    df1.head():                Material
    0  Calcium, Ca; Rolled
    df1.head():                Material Density - Metric Density - English Density - Comments  \
    0  Calcium, Ca; Rolled        1.54 g/cc     0.0556 lb/in³                      
    
      Atomic Mass - Metric Atomic Mass - English Atomic Mass - Comments  \
    0               40.078                40.078                   1995   
    
      Atomic Number - Metric Atomic Number - English Atomic Number - Comments  \
    0                     20                      20                            
    
       ... Melting Point - English Melting Point - Comments  \
    0  ...          1540 - 1550 °F                            
    
      Boiling Point - Metric Boiling Point - English Boiling Point - Comments  \
    0                1484 °C                 2703 °F                            
    
      Calcium, Ca - Metric Calcium, Ca - English Calcium, Ca - Comments  \
    0                100 %                 100 %                          
    
      CAS Number CAS Number- Comments  
    0  7740-70-2                       
    
    [1 rows x 90 columns]
    response.url:  http://www.matweb.com/search/DataSheet.aspx?MatGUID=25ce9b7f40364cf79d54ed0db5c8e41f&ckck=1
    title.text:  
    	Chromium, Cr; As-Swaged
    
    rows[2].text:  Density 7.19 g/cc0.260 lb/in³
    df1.head():                    Material
    0  Chromium, Cr; As-Swaged
    df1.head():                    Material Density - Metric Density - English  \
    0  Chromium, Cr; As-Swaged        7.19 g/cc      0.260 lb/in³   
    
      Density - Comments Atomic Mass - Metric Atomic Mass - English  \
    0                                 51.9961               51.9961   
    
      Atomic Mass - Comments Atomic Number - Metric Atomic Number - English  \
    0                   1995                     24                      24   
    
      Atomic Number - Comments  ... Emissivity (0-1) - English  \
    0                           ...    0.10@Temperature 122 °F   
    
      Emissivity (0-1) - Comments Reflection Coefficient, Visible (0-1) - Metric  \
    0   polished, total radiation                                           0.70   
    
      Reflection Coefficient, Visible (0-1) - English  \
    0                                            0.70   
    
      Reflection Coefficient, Visible (0-1) - Comments Chromium, Cr - Metric  \
    0                                           500 nm                 100 %   
    
      Chromium, Cr - English Chromium, Cr - Comments CAS Number  \
    0                  100 %                          7440-47-3   
    
      CAS Number- Comments  
    0                       
    
    [1 rows x 93 columns]
    response.url:  http://www.matweb.com/search/DataSheet.aspx?MatGUID=9aebe83845c04c1db5126fada6f76f7e&ckck=1
    title.text:  
    	Copper, Cu; Annealed
    
    rows[2].text:  Density  7.764 g/cc@Temperature 1300 °C0.2805 lb/in³@Temperature 2370 °F
    df1.head():                 Material
    0  Copper, Cu; Annealed
    df1.head():                 Material               Density  - Metric  \
    0  Copper, Cu; Annealed  7.764 g/cc@Temperature 1300 °C   
    
                      Density  - English Density  - Comments  \
    0  0.2805 lb/in³@Temperature 2370 °F                       
    
                              - Metric  \
    0  19600 W/m-K@Temperature -263 °C   
    
                                         - English  - Comments  \
    0  136000 BTU-in/hr-ft²-°F@Temperature -441 °F               
    
      Atomic Mass - Metric Atomic Mass - English Atomic Mass - Comments  ...  \
    0               65.546                65.546                   1995  ...   
    
                          Emissivity (0-1) - English Emissivity (0-1) - Comments  \
    0  0.15@Wavelength >=655 nm, Temperature 1480 °F                    polished   
    
      Reflection Coefficient, Visible (0-1) - Metric  \
    0                                           0.63   
    
      Reflection Coefficient, Visible (0-1) - English  \
    0                                            0.63   
    
      Reflection Coefficient, Visible (0-1) - Comments Copper, Cu - Metric  \
    0                                                                100 %   
    
      Copper, Cu - English Copper, Cu - Comments CAS Number CAS Number- Comments  
    0                100 %                        7440-50-8                       
    
    [1 rows x 96 columns]
    df.head():      - Comments   - English   - Metric            Atomic Mass - Comments  \
    0          NaN         NaN        NaN  196.96655 - 19951995199519951995   
    
                     Atomic Mass - English                 Atomic Mass - Metric  \
    0  196.96669.01218240.07851.996165.546  196.96669.01218240.07851.996165.546   
    
      Atomic Number - Comments Atomic Number - English Atomic Number - Metric  \
    0                                        794202429              794202429   
    
      Beryllium, Be - Comments  ...  \
    0                      NaN  ...   
    
                  Thermal Neutron Cross Section - Metric  \
    0  99 barns/atom0.0090 barns/atom0.43 barns/atom2...   
    
      Vapor Pressure  - Comments Vapor Pressure  - English  \
    0                        NaN                       NaN   
    
      Vapor Pressure  - Metric X-ray Absorption Edge - Comments  \
    0                      NaN                            KKKKK   
    
              X-ray Absorption Edge - English  \
    0  0.15344 Å110.68 Å3.07016 Å2.07 Å1.38 Å   
    
               X-ray Absorption Edge - Metric  - Comments  \
    0  0.15344 Å110.68 Å3.07016 Å2.07 Å1.38 Å   600 nmbcc   
    
                                               - English  \
    0  0.8510.2 µin/in-°F@Temperature 77.0 - 1830 °F0...   
    
                                                - Metric  
    0  0.8518.4 µm/m-°C@Temperature 25.0 - 1000 °C1.1...  
    
    [1 rows x 156 columns]



```python
#Constants
urls = [
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e',
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8a6a0df6122349b7bdc92662658d4a4f',
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=15135dd30619457f902229b03619841a',
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=25ce9b7f40364cf79d54ed0db5c8e41f',
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=9aebe83845c04c1db5126fada6f76f7e',
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1980eb23287a4408adc404dd39293942',
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8cdce331dd324e80ab4a9fd801466a92',
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1fc9adc121df4b33a4263065d602a4e1',
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=750a3dd8d69b44b79468fbaf72a2beef',
    'http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd'
]

user_agent_list = [
   #Chrome
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.113 Safari/537.36',
    'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36',
    'Mozilla/5.0 (Windows NT 5.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36',
    'Mozilla/5.0 (Windows NT 6.2; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36',
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36',
    'Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.113 Safari/537.36',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36',
    'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36',
    'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36',
    #Firefox
    'Mozilla/4.0 (compatible; MSIE 9.0; Windows NT 6.1)',
    'Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko',
    'Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0)',
    'Mozilla/5.0 (Windows NT 6.1; Trident/7.0; rv:11.0) like Gecko',
    'Mozilla/5.0 (Windows NT 6.2; WOW64; Trident/7.0; rv:11.0) like Gecko',
    'Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko',
    'Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.0; Trident/5.0)',
    'Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; rv:11.0) like Gecko',
    'Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)',
    'Mozilla/5.0 (Windows NT 6.1; Win64; x64; Trident/7.0; rv:11.0) like Gecko',
    'Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)',
    'Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)',
    'Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)'
]

def get_proxies():
    link = 'https://free-proxy-list.net/'
    #link = 'https://www.sslproxies.org/'
    response = requests.get(link)
    parser = fromstring(response.text)
    proxies = set()
    for i in parser.xpath('//tbody/tr')[:10]:
        if i.xpath('.//td[7][contains(text(),"yes")]'):
            proxy = ":".join([i.xpath('.//td[1]/text()')[0], i.xpath('.//td[2]/text()')[0]])
            proxies.add(proxy)
    return proxies

def main():
    i = 1
    time_delay = random.randrange(0, 10)
    print('time_delay: ', time_delay)
    time.sleep(time_delay)
    ua = UserAgent()
    proxies = get_proxies()
    proxy_pool = cycle(proxies)
    
    print('proxies: ', proxies)

    #Get a proxy from the pool
    proxy = next(proxy_pool)
    print("Request #%d"%1)
    #Pick a random url
    url = random.choice(urls)
    #Pick a random user agent
    user_agent = ua.random
    print('url, user_agent: ', url, ' ', user_agent)
    #Set the headers
    headers = {'User-Agent': user_agent}
    user = 'rakeshbhatia.developer@gmail.com'
    pwd = 'carnival01'
    try:
        #Make the request
        response = requests.get(url, headers=headers, proxies={"http": proxy, "https": proxy}, auth=HTTPBasicAuth(user, pwd))
        #response = requests.get(url, headers=headers, proxies={"http": proxy, "https": proxy})
        #response = requests.get(url, auth=HTTPBasicAuth(user, pwd))
        print('current url: ', response.url)

        soup = BeautifulSoup(response.text, 'html.parser')
        table = soup.find_all('table', attrs={'class': 'tabledataformat'})[2]
        rows = table.find_all('tr')
        print('rows[1]: ', rows[1])
        for row in rows[1:]:
            data = row.find_all('td')
            print('data[1].contents: ', data[1].contents)
    except:
        #Most free proxies will often get connection errors. You will have retry the entire request using another proxy to work. 
        #We will just skip retries as its beyond the scope of this tutorial and we are only downloading a single url 
        print("Skipping. Connection error")
            
    '''for i in range(1,11):
        time_delay = random.randrange(0, 10)
        print('time_delay: ', time_delay)
        time.sleep(time_delay)
        #Get a proxy from the pool
        proxy = next(proxy_pool)
        print("Request #%d"%i)
        #Pick a random url
        url = random.choice(urls)
        #Pick a random user agent
        user_agent = ua.random
        print('url, user_agent: ', url, ' ', user_agent)
        #Set the headers
        headers = {'User-Agent': user_agent}
        user = 'rakeshbhatia.developer@gmail.com'
        pwd = 'carnival01'
        try:
            #Make the request
            response = requests.get(url, headers=headers, auth=HTTPBasicAuth(user, pwd))
            #response = requests.get(url, headers=headers, proxies={"http": proxy, "https": proxy})
            #response = requests.get(url)
            print('current url: ', response.url)

            soup = BeautifulSoup(response.text, 'html.parser')
            table = soup.find_all('table', attrs={'class': 'tabledataformat'})[2]
            rows = table.find_all('tr')
            print('rows[0]: ', rows[0])
            for row in rows[1:]:
                data = row.find_all('td')
                print('data[1].contents: ', data[1].contents)
        except:
            #Most free proxies will often get connection errors. You will have retry the entire request using another proxy to work. 
            #We will just skip retries as its beyond the scope of this tutorial and we are only downloading a single url 
            print("Skipping. Connection error")'''

    '''# Choose a random proxy
    proxy_index = random_proxy()
    proxy = proxies[proxy_index]

    for i in range(1, len(urls)):
        req = Request(urls[i])
        req.set_proxy(proxy['ip'] + ':' + proxy['port'], 'http')

        # Every 10 requests, generate a new proxy
        if n % 10 == 0:
            proxy_index = random_proxy()
            proxy = proxies[proxy_index]

        # Make the call
        try:
            my_ip = urlopen(req).read().decode('utf8')
            print('#' + str(n) + ': ' + my_ip)
        except: # If error, delete this proxy and find another one
            del proxies[proxy_index]
            print('Proxy ' + proxy['ip'] + ':' + proxy['port'] + ' deleted.')
            proxy_index = random_proxy()
            proxy = proxies[proxy_index]'''

    '''#Pick a random url
    url = random.choice(urls)
    #Pick a random user agent
    user_agent = random.choice(user_agent_list)
    print('url, user_agent: ', url, ' ', user_agent)
    #Set the headers
    headers = {'User-Agent': user_agent}
    #Make the request
    response = requests.get(url, headers=headers, proxies=random_proxy)

    print('current url: ', response.url)
    print('html: ', response.text)'''

if __name__ == '__main__':
    main()

```

    time_delay:  7
    proxies:  {'54.38.110.35:47640', '185.10.166.130:8080', '190.186.59.22:52335', '81.223.122.78:48982', '112.78.170.29:8080', '185.5.19.234:52975', '212.42.206.58:3128', '203.189.150.223:55700', '103.49.221.27:8080'}
    Request #1
    url, user_agent:  http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e   Mozilla/5.0 (Windows NT 6.2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/28.0.1467.0 Safari/537.36
    current url:  http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e&ckck=1
    rows[1]:  <tr><th>Physical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr>
    Skipping. Connection error



```python
#firefox_options.accept_untrusted_certs = True
#firefox_options.add_argument("--ignore-certificate-errors")
#firefox_options.add_argument("user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36")

headers = {
        'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36',
    }

#chrome_options = Options(headers=headers)
chrome_options = Options()
#chrome_options.add_argument("--headless")
chrome_options.accept_untrusted_certs = True
chrome_options.add_argument("--ignore-certificate-errors")

#chrome_options.add_argument("user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36")

options = webdriver.ChromeOptions()
#chrome_options.add_argument("--headless")
options.accept_untrusted_certs = True
options.add_argument("--ignore-certificate-errors")
options.add_argument("--test-type")
#options.binary_location = "/usr/bin/chromium"
```


```python
driver.quit()
```


```python
sleep(5)
#link = "http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e&ckck=1"
link = "http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e"
#driver = webdriver.PhantomJS(service_args=["--load-images=no", '--disk-cache=true'])
#driver = webdriver.Firefox()

driver.get(link)
print(driver.current_url)
soup = BeautifulSoup(driver.page_source, "lxml")
table = soup.find_all("table")[8]
rows = soup.find_all("table")[8].find("tbody").find_all("tr")
#print(table)
print(rows[0])
driver.quit()
```


```python
# Extract DataFrame
df = pd.read_html(table.prettify())[0]
df
```


```python
# Drop excess rows and columns
df = df.drop([4, 5], axis=1)
df = df.drop([0], axis=0)
df
```


```python
properties_table = display_html(table.prettify(), raw=True)
properties_table
```


```python
tag = 'td'

filter_meta = {'style':'vertical-align:top;','class':False}
all_labels = table.findAll(tag, filter_meta)
print("######################################### print_all_labels")
print(all_labels)
print("#########################################")
for label in all_labels:
    print(label)
    sibling = label.findNextSibling()
    while sibling is not None:
        print(sibling.getText)
        sibling = sibling.findNextSibling()
```
