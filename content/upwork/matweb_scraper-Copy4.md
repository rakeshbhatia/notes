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

print(python_version())

#url = "http://www.matweb.com/search/PropertySearch.aspx"
```

    3.6.7



```python
# Open database connection
db = pymysql.connect("localhost","root","carnival01","decisive_mfg" )

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
connection = mysql.connector.connect(host='localhost',
                                     database=,
                                     user=,
                                     password=)

try:
    if connection.is_connected():
        db_Info = connection.get_server_info()
        print("Connected to MySQL Server version ", db_Info)
        cursor = connection.cursor()
        cursor.execute("select database();")
        record = cursor.fetchone()
        print("You're connected to database: ", record)

except Error as e:
    print("Error while connecting to MySQL", e)
finally:
    if (connection.is_connected()):
        cursor.close()
        connection.close()
        print("MySQL connection is closed")
```


    ---------------------------------------------------------------------------

    MySQLInterfaceError                       Traceback (most recent call last)

    ~/anaconda/lib/python3.6/site-packages/mysql/connector/connection_cext.py in _open_connection(self)
        199         try:
    --> 200             self._cmysql.connect(**cnx_kwargs)
        201         except MySQLInterfaceError as exc:


    MySQLInterfaceError: SSL connection error: SSL_CTX_set_tmp_dh failed

    
    During handling of the above exception, another exception occurred:


    InterfaceError                            Traceback (most recent call last)

    <ipython-input-198-19adc6266e38> in <module>
          2                                      database='Electronics',
          3                                      user='pynative',
    ----> 4                                      password='pynative@#29')
          5 
          6 try:


    ~/anaconda/lib/python3.6/site-packages/mysql/connector/__init__.py in connect(*args, **kwargs)
        174 
        175     if HAVE_CEXT and not use_pure:
    --> 176         return CMySQLConnection(*args, **kwargs)
        177     return MySQLConnection(*args, **kwargs)
        178 Connect = connect  # pylint: disable=C0103


    ~/anaconda/lib/python3.6/site-packages/mysql/connector/connection_cext.py in __init__(self, **kwargs)
         78 
         79         if kwargs:
    ---> 80             self.connect(**kwargs)
         81 
         82     def _add_default_conn_attrs(self):


    ~/anaconda/lib/python3.6/site-packages/mysql/connector/abstracts.py in connect(self, **kwargs)
        779 
        780         self.disconnect()
    --> 781         self._open_connection()
        782         # Server does not allow to run any other statement different from ALTER
        783         # when user's password has been expired.


    ~/anaconda/lib/python3.6/site-packages/mysql/connector/connection_cext.py in _open_connection(self)
        201         except MySQLInterfaceError as exc:
        202             raise errors.get_mysql_exception(msg=exc.msg, errno=exc.errno,
    --> 203                                              sqlstate=exc.sqlstate)
        204         self._do_handshake()
        205 


    InterfaceError: 2026 (HY000): SSL connection error: SSL_CTX_set_tmp_dh failed



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
