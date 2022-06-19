```python
import csv
import time
from time import sleep
import string
import random
import urllib
import requests
import json
import numpy
import pandas as pd
import bs4
from bs4 import BeautifulSoup
from lxml.html import fromstring
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

print(python_version())

my_user_agent = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36'

url = "http://www.matweb.com/search/PropertySearch.aspx"
urls = ['http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e&ckck=1',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8a6a0df6122349b7bdc92662658d4a4f',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=15135dd30619457f902229b03619841a',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=25ce9b7f40364cf79d54ed0db5c8e41f',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=9aebe83845c04c1db5126fada6f76f7e',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1980eb23287a4408adc404dd39293942',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=8cdce331dd324e80ab4a9fd801466a92',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=1fc9adc121df4b33a4263065d602a4e1',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=750a3dd8d69b44b79468fbaf72a2beef',
        'http://www.matweb.com/search/DataSheet.aspx?MatGUID=a438ce15c0964513aa34c7973fce82fd']
```

    3.6.7



```python
def get_proxies():
    url = 'https://www.socks-proxy.net/'
    response = requests.get(url)
    parser = fromstring(response.text)
    proxies = set()
    for i in parser.xpath('//*[@id="proxylisttable"]/tbody'):
        if i.xpath('[contains(text(),"yes")]'):
            #Grabbing IP and corresponding PORT
            proxy = ":".join([i.xpath('.//td[1]/text()')[0], i.xpath('.//td[2]/text()')[0]])
            proxies.add(proxy)
    return proxies
```


```python
proxies = get_proxies()
print(proxies)
```

    {'80.78.75.59:38253', '45.112.57.230:61222'}



```python
#time_delay = random.randrange(0, delay)
headers = {'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36'}

def scrape_matweb():
    matweb_url = 'http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e'
    response = requests.get(matweb_url, headers=headers)

    # If the response was successful, no Exception will be raised
    response.raise_for_status()

    print('response.url: ', response.url)

    soup = BeautifulSoup(response.text, 'lxml')
    table = soup.find('table', attrs={'cellspacing': '0'})
    print('table: ', table)
    #rows = soup.find_all("table")[8].find("tbody").find_all("tr")

    # Extract DataFrame
    #df = pd.read_html(table.prettify())[0]
    #df = df.drop([4, 5], axis=1)
    #df = df.drop([0], axis=0)
    #df.head()
```


```python
scrape_matweb()
```

    response.url:  http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e&ckck=1
    table:  <table cellspacing="0" class="tabledataformat"><tr><td colspan="4"> </td></tr><tr><th>Physical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Density </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=43&amp;fromValue=19.32" title="Click to see this value in other UOMs">19.32</a> g/cc<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=87&amp;fromValue=0.6980" title="Click to see this value in other UOMs">0.6980</a> lb/in³<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Vapor Pressure <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=788&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=0.001013" title="Click to see this value in other UOMs">0.001013</a> bar<span class="dataCondition"><br/>@Temperature 1770 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=0.7598" title="Click to see this value in other UOMs">0.7598</a> torr<span class="dataCondition"><br/>@Temperature 3220 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=0.01013" title="Click to see this value in other UOMs">0.01013</a> bar<span class="dataCondition"><br/>@Temperature 2036 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=7.598" title="Click to see this value in other UOMs">7.598</a> torr<span class="dataCondition"><br/>@Temperature 3697 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=0.1013" title="Click to see this value in other UOMs">0.1013</a> bar<span class="dataCondition"><br/>@Temperature 2383 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=75.98" title="Click to see this value in other UOMs">75.98</a> torr<span class="dataCondition"><br/>@Temperature 4321 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=1.013" title="Click to see this value in other UOMs">1.013</a> bar<span class="dataCondition"><br/>@Temperature 2857 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=759.8" title="Click to see this value in other UOMs">759.8</a> torr<span class="dataCondition"><br/>@Temperature 5175 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Chemical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Atomic Mass </td><td class="dataCell" style="vertical-align:top;">196.9666<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">196.9666<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">196.96655 - 1995</td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Atomic Number </td><td class="dataCell" style="vertical-align:top;">79<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">79<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Thermal Neutron Cross Section </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=300&amp;fromValue=99" title="Click to see this value in other UOMs">99</a> barns/atom<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=300&amp;fromValue=99" title="Click to see this value in other UOMs">99</a> barns/atom<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">X-ray Absorption Edge </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.15344" title="Click to see this value in other UOMs">0.15344</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.15344" title="Click to see this value in other UOMs">0.15344</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">K</td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.86378" title="Click to see this value in other UOMs">0.86378</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.86378" title="Click to see this value in other UOMs">0.86378</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">L<sub>I</sub></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.90277" title="Click to see this value in other UOMs">0.90277</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.90277" title="Click to see this value in other UOMs">0.90277</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">L<sub>II</sub></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.04028" title="Click to see this value in other UOMs">1.04028</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.04028" title="Click to see this value in other UOMs">1.04028</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">L<sub>III</sub></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Electrode Potential </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=146&amp;fromValue=1.68" title="Click to see this value in other UOMs">1.68</a> V<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=146&amp;fromValue=1.68" title="Click to see this value in other UOMs">1.68</a> V<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Electronegativity </td><td class="dataCell" style="vertical-align:top;">2.4<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">2.4<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">Pauling</td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Ionic Radius </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.850" title="Click to see this value in other UOMs">0.850</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.850" title="Click to see this value in other UOMs">0.850</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">Crystal Ionic Radius for Valence +3</td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.37" title="Click to see this value in other UOMs">1.37</a> Å<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.37" title="Click to see this value in other UOMs">1.37</a> Å<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">Crystal Ionic Radius for Valence +1</td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Electrochemical Equivalent </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=301&amp;fromValue=2.45" title="Click to see this value in other UOMs">2.45</a> g/A/h<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=301&amp;fromValue=2.45" title="Click to see this value in other UOMs">2.45</a> g/A/h<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Mechanical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Hardness, Vickers </td><td class="dataCell" style="vertical-align:top;">25<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">25<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Tensile Strength, Ultimate </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=120" title="Click to see this value in other UOMs">120</a> MPa<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=17400" title="Click to see this value in other UOMs">17400</a> psi<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">Annealed</td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Elongation at Break </td><td class="dataCell" style="vertical-align:top;">30 %<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">30 %<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Modulus of Elasticity </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=45&amp;fromValue=77.2" title="Click to see this value in other UOMs">77.2</a> GPa<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=78&amp;fromValue=11200" title="Click to see this value in other UOMs">11200</a> ksi<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">60% Cold Worked</td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Poissons Ratio </td><td class="dataCell" style="vertical-align:top;">0.42<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">0.42<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Shear Modulus </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=45&amp;fromValue=27.2" title="Click to see this value in other UOMs">27.2</a> GPa<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=78&amp;fromValue=3950" title="Click to see this value in other UOMs">3950</a> ksi<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">60% Cold Worked; Calculated Value</td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Electrical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Electrical Resistivity </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.00000220" title="Click to see this value in other UOMs">0.00000220</a> ohm-cm<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.00000220" title="Click to see this value in other UOMs">0.00000220</a> ohm-cm<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Magnetic Susceptibility </td><td class="dataCell" style="vertical-align:top;">-1.42e-7<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">-1.42e-7<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">cgs/g</td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Thermal Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Heat of Fusion </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=64&amp;fromValue=66.2" title="Click to see this value in other UOMs">66.2</a> J/g<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=11&amp;fromValue=28.5" title="Click to see this value in other UOMs">28.5</a> BTU/lb<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Heat of Vaporization </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=64&amp;fromValue=1738" title="Click to see this value in other UOMs">1738</a> J/g<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=11&amp;fromValue=747.7" title="Click to see this value in other UOMs">747.7</a> BTU/lb<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">CTE, linear <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=182&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=14.4" title="Click to see this value in other UOMs">14.4</a> µm/m-°C<span class="dataCondition"><br/>@Temperature 20.0 - 100 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=8.00" title="Click to see this value in other UOMs">8.00</a> µin/in-°F<span class="dataCondition"><br/>@Temperature 68.0 - 212 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=14.6" title="Click to see this value in other UOMs">14.6</a> µm/m-°C<span class="dataCondition"><br/>@Temperature 250 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=8.11" title="Click to see this value in other UOMs">8.11</a> µin/in-°F<span class="dataCondition"><br/>@Temperature 482 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=15.2" title="Click to see this value in other UOMs">15.2</a> µm/m-°C<span class="dataCondition"><br/>@Temperature 500 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=8.44" title="Click to see this value in other UOMs">8.44</a> µin/in-°F<span class="dataCondition"><br/>@Temperature 932 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=16.7" title="Click to see this value in other UOMs">16.7</a> µm/m-°C<span class="dataCondition"><br/>@Temperature 950 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=9.28" title="Click to see this value in other UOMs">9.28</a> µin/in-°F<span class="dataCondition"><br/>@Temperature 1740 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Specific Heat Capacity <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=695&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.128" title="Click to see this value in other UOMs">0.128</a> J/g-°C<span class="dataCondition"><br/>@Temperature 25.0 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0306" title="Click to see this value in other UOMs">0.0306</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 77.0 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.133" title="Click to see this value in other UOMs">0.133</a> J/g-°C<span class="dataCondition"><br/>@Temperature 227 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0318" title="Click to see this value in other UOMs">0.0318</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 441 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.142" title="Click to see this value in other UOMs">0.142</a> J/g-°C<span class="dataCondition"><br/>@Temperature 627 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0339" title="Click to see this value in other UOMs">0.0339</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 1160 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.159" title="Click to see this value in other UOMs">0.159</a> J/g-°C<span class="dataCondition"><br/>@Temperature 1227 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0380" title="Click to see this value in other UOMs">0.0380</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 2241 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.163" title="Click to see this value in other UOMs">0.163</a> J/g-°C<span class="dataCondition"><br/>@Temperature 1027 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0390" title="Click to see this value in other UOMs">0.0390</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 1881 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.166" title="Click to see this value in other UOMs">0.166</a> J/g-°C<span class="dataCondition"><br/>@Temperature 1127 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0397" title="Click to see this value in other UOMs">0.0397</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 2061 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.170" title="Click to see this value in other UOMs">0.170</a> J/g-°C<span class="dataCondition"><br/>@Temperature 1063 °C</span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0406" title="Click to see this value in other UOMs">0.0406</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 1945 °F</span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Thermal Conductivity </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=136&amp;fromValue=301" title="Click to see this value in other UOMs">301</a> W/m-K<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=10&amp;fromValue=2090" title="Click to see this value in other UOMs">2090</a> BTU-in/hr-ft²-°F<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Melting Point </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=1064.43" title="Click to see this value in other UOMs">1064.43</a> °C<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=1947.97" title="Click to see this value in other UOMs">1947.97</a> °F<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr class="altrow datarowSeparator"><td style="vertical-align:top;">Boiling Point </td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=2856" title="Click to see this value in other UOMs">2856</a> °C<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=5173" title="Click to see this value in other UOMs">5173</a> °F<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Optical Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Emissivity (0-1) </td><td class="dataCell" style="vertical-align:top;">0.040<span class="dataCondition"><br/>@Temperature 100 °C</span></td><td class="dataCell" style="vertical-align:top;">0.040<span class="dataCondition"><br/>@Temperature 212 °F</span></td><td class="dataComment" style="vertical-align:top;">total spectrum</td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;">Reflection Coefficient, Visible (0-1) </td><td class="dataCell" style="vertical-align:top;">0.27<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">0.27<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">400 nm</td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;">0.50<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">0.50<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">500 nm</td></tr>
    <tr class="datarowSeparator"><td style="vertical-align:top;"> </td><td class="dataCell" style="vertical-align:top;">0.85<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">0.85<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;">600 nm</td></tr>
    <tr><td colspan="4"> </td></tr><tr><th>Component Elements Properties</th><th class="dataCell">Metric</th><th class="dataCell">English</th><th class="dataCell">Comments</th></tr><tr class="altrow datarowSeparator"><td style="vertical-align:top;">Gold, Au </td><td class="dataCell" style="vertical-align:top;">100 %<span class="dataCondition"></span></td><td class="dataCell" style="vertical-align:top;">100 %<span class="dataCondition"></span></td><td class="dataComment" style="vertical-align:top;"></td></tr>
    <tr><td colspan="4"> </td></tr><tr><th align="left" class="" colspan="6">Descriptive Properties</th></tr><tr class="altrow"><td>CAS Number</td><td class="dataCell" colspan="2">7440-57-5</td><td class="dataComment" colspan="3"></td></tr></table>



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
def init():
    driver = webdriver.Chrome(chrome_options=options)
    #driver = webdriver.Firefox(capabilities={"acceptInsecureCerts": True})
    #driver = webdriver.Chrome(capabilities={"acceptInsecureCerts": True})
    driver.get(url)
    return driver
```


```python
def select_material_category(d):
    sleep(3)
    toggle_metal_xpath = '//*[@id="ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewn3"]'
    #toggle_metal_button = d.find_element_by_xpath(toggle_metal_xpath)
    #toggle_metal_button = d.find_element_by_id("ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewn3")
    toggle_metal_button = d.find_element_by_id("ctl00_ContentMain_ucMatGroupTree_LODCS1_msTreeViewt3")
    toggle_metal_button.click()
```


```python
'//*[@id="ctl00_ContentMain_ucPropertyDropdown1_drpPropertyList"]/option[21]'
'/html/body/form[2]/div[4]/table[2]/tbody/tr/td[2]/table/tbody/tr[2]/td/div/select/option[21]'
```




    '/html/body/form[2]/div[4]/table[2]/tbody/tr/td[2]/table/tbody/tr[2]/td/div/select/option[21]'




```python
def select_material_property_1(d):
    sleep(3)
    #material_property = d.find_element_by_name("ctl00$ContentMain$ucPropertyDropdown1$drpPropertyList")
    material_property_select = Select(d.find_element_by_name("ctl00$ContentMain$ucPropertyDropdown1$drpPropertyList"))
    material_property_select.select_by_value("215")

    #material_property_select.select_by_index(20)
    
    sleep(3)
    
    material_property_min = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit1$txtpMin")
    material_property_min.clear()
    material_property_min.send_keys("1")
        
    material_property_max = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit1$txtpMax")
    material_property_max.send_keys("20")
```


```python
def select_material_property_2(d):
    sleep(3)
    #material_property = d.find_element_by_id("ctl00$ContentMain$ucPropertyDropdown1$drpPropertyList")
    material_property_select = Select(d.find_element_by_name("ctl00$ContentMain$ucPropertyDropdown2$drpPropertyList"))
    material_property_select.select_by_value("263")

    sleep(3)

    material_property_min = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit2$txtpMin")
    material_property_min.clear()
    material_property_min.send_keys("0.1")
    
    material_property_max = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit2$txtpMax")
    material_property_max.send_keys("2000")
```


```python
def select_material_property_3(d):
    sleep(3)
    #material_property = d.find_element_by_id("ctl00$ContentMain$ucPropertyDropdown1$drpPropertyList")
    material_property_select = Select(d.find_element_by_name("ctl00$ContentMain$ucPropertyDropdown3$drpPropertyList"))
    material_property_select.select_by_value("743")
    
    sleep(3)
    
    material_property_min = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit3$txtpMin")
    material_property_min.clear()
    material_property_min.send_keys("100")
    
    material_property_max = d.find_element_by_name("ctl00$ContentMain$ucPropertyEdit3$txtpMax")
    material_property_max.send_keys("6000")
```


```python
def submit_query(d):
    sleep(3)
    submit_button = d.find_element_by_name("ctl00$ContentMain$btnSubmit")
    submit_button.click()
```


```python
def select_view(d, desired_view):
    sleep(3)
    material_property_select = Select(d.find_element_by_id("ctl00_ContentMain_UcSearchResults1_drpPageSize2"))
    material_property_select.select_by_value(desired_view)
```


```python
def extract_links(d):
    # id: lnkMatl_7
    # xpath: //*[@id="lnkMatl_7"]
    # full xpath: /html/body/form[2]/div[4]/div[2]/div/table[3]/tbody/tr[2]/td[3]/a
    # /html/body/form[2]/div[4]/div[2]/div/table[3]/tbody/tr[2]/td[3]
    sleep(5)
    rows = d.find_elements_by_xpath("/html/body/form[2]/div[4]/div[2]/div/table[3]/tbody/tr")
    links = {}
    for row in rows[1:]:
        a = row.find_element_by_tag_name("a")
        links[a.text] = a.get_attribute("href")
        #print("a.text: ", a.text)
        #print("a.get_attribute('href'): ", a.get_attribute("href"))
    return links
```


```python
def extract_links_2(d):
    sleep(5)
    
    links = {}
```


```python
def extract_links_3(d):
    sleep(5)

    session = HTMLSession()
    r = session.get("http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e")
    r.html.links
    
    soup = BeautifulSoup(d.page_source, "lxml")
    table = soup.find("table", attrs={"id": "tblResults"})
    #print("table: ", table)
    rows = table.find_all("tr")
    print("rows[0]: ", rows[0])
    
    links = {}
    for row in rows[1:]:
        a = row.find("a")
        print("a.text: ", a.text)
        links[a.text] = "http://www.matweb.com" + a["href"]
        #print(links[a.text])
        #links.append(link)
    return links
```


```python
def go_next_page(d):
    sleep(3)
    #"ctl00_ContentMain_UcSearchResults1_lnkNextPage2"
    try:
        next_page_button = d.find_element_by_name("ctl00_ContentMain_UcSearchResults1_lnkNextPage2").click()
        #next_page_button.click()
    except NoSuchElementException:
        next_page_button = None
    return next_page_button
```


```python
def get_material_properties(links):
    dfs = {}
   
    #for k, v in links.items():
    #print("k, v: ", k, " ", v)
    #try:
    response = requests.get("http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e")

    # If the response was successful, no Exception will be raised
    response.raise_for_status()

    sleep(3)

    print("response.url: ", response.url)

    soup = BeautifulSoup(response.text, "lxml")
    table = soup.find("table", attrs={"class": "tabledataformat"})
    print("table: ", table)
    #rows = soup.find_all("table")[8].find("tbody").find_all("tr")

    # Extract DataFrame
    df = pd.read_html(table.prettify())[0]
    #df = df.drop([4, 5], axis=1)
    #df = df.drop([0], axis=0)
    df.head()

    #dfs[k] = df
    #break
    '''except urllib.HTTPError as http_err:
        print(f'HTTP error occurred: {http_err}')  # Python 3.6
    except Exception as err:
        print(f'Other error occurred: {err}')  # Python 3.6
    else:
        print('Success!')'''
    
```


```python
def scrape_matweb(d):
    '''material_property_links = {}
    
    # Select material category
    select_material_category(d)
    
    # Select material properties
    select_material_property_1(d)
    select_material_property_2(d)
    select_material_property_3(d)
    
    # Submit query
    submit_query(d)
    
    # Select max view of 200 per page
    select_view(d, "200")
    
    # Extract material property links
    material_property_links = extract_links(d)'''
    
    # Go to the next page
    #go_next_page(d)
    
    material_property_links = []
    
    # Get properties of each material
    get_material_properties(material_property_links)
    
    #sleep(10)
```


```python
if __name__ == '__main__':
    print("Matweb Scraper v1")
    driver = init()
    data = scrape_matweb(driver)
    driver.quit()
```

    Matweb Scraper v1


    /Users/rakeshbhatia/anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:2: DeprecationWarning: use options instead of chrome_options
      


    response.url:  http://www.matweb.com/errorUser.aspx?msgid=11
    table:  <table class="tabledataformat tableloose" id="tblRecentMatls" style="background-color:White;">
    <tr><td>
    <p></p>
    <a href="/membership/login.aspx">Login</a> to see your most recently viewed materials here.<p></p>
                    Or if you don't have an account with us yet, then <a href="/membership/regstart.aspx">click here to register.</a>
    </td></tr>
    </table>



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
df.head(100)
```


```python
df.to_csv("matweb_sample_data_gold_au.csv")
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
