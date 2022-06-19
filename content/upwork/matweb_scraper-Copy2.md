```python
import csv
import time
import string
import urllib
import requests
import json
import numpy
import pandas as pd
import bs4
from bs4 import BeautifulSoup
from requests_html import HTMLSession
from time import sleep
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import Select
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.firefox.options import Options
from selenium.common.exceptions import NoSuchElementException
from IPython.display import display_html

url = "http://www.matweb.com/search/PropertySearch.aspx"
```


```python
firefox_options = Options()
#firefox_options.accept_untrusted_certs = True
#firefox_options.add_argument("--ignore-certificate-errors")
#firefox_options.add_argument("user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36")

chrome_options = Options()
#chrome_options.add_argument("--headless")
chrome_options.accept_untrusted_certs = True
chrome_options.add_argument("--ignore-certificate-errors")
chrome_options.add_argument("user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36")

options = webdriver.ChromeOptions()
#chrome_options.add_argument("--headless")
options.accept_untrusted_certs = True
options.add_argument("--ignore-certificate-errors")
options.add_argument("--test-type")
#options.binary_location = "/usr/bin/chromium"
```


```python
browser_options.add_argument("--headless")
browser_options.add_argument('--no-sandbox')
browser = webdriver.Chrome(webdriver_path, chrome_options=browser_options)
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

    /Users/rakeshbhatia/anaconda/lib/python3.6/site-packages/selenium/webdriver/phantomjs/webdriver.py:49: UserWarning: Selenium support for PhantomJS has been deprecated, please use headless versions of Chrome or Firefox instead
      warnings.warn('Selenium support for PhantomJS has been deprecated, please use headless '



    ---------------------------------------------------------------------------

    WebDriverException                        Traceback (most recent call last)

    <ipython-input-15-2d5eecb196df> in <module>
          5 #link = "http://www.matweb.com/search/DataSheet.aspx?MatGUID=15135dd30619457f902229b03619841a"
          6 #link = "http://www.matweb.com/search/DataSheet.aspx?MatGUID=25ce9b7f40364cf79d54ed0db5c8e41f"
    ----> 7 driver = webdriver.PhantomJS(service_args=["--load-images=no", '--disk-cache=true'])
          8 #driver = webdriver.Firefox()
          9 driver.get(link)


    ~/anaconda/lib/python3.6/site-packages/selenium/webdriver/phantomjs/webdriver.py in __init__(self, executable_path, port, desired_capabilities, service_args, service_log_path)
         54             service_args=service_args,
         55             log_path=service_log_path)
    ---> 56         self.service.start()
         57 
         58         try:


    ~/anaconda/lib/python3.6/site-packages/selenium/webdriver/common/service.py in start(self)
         96         count = 0
         97         while True:
    ---> 98             self.assert_process_still_running()
         99             if self.is_connectable():
        100                 break


    ~/anaconda/lib/python3.6/site-packages/selenium/webdriver/common/service.py in assert_process_still_running(self)
        109             raise WebDriverException(
        110                 'Service %s unexpectedly exited. Status code was: %s'
    --> 111                 % (self.path, return_code)
        112             )
        113 


    WebDriverException: Message: Service phantomjs unexpectedly exited. Status code was: -6




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
        print("a.text: ", a.text)
        print("a.get_attribute('href'): ", a.get_attribute("href"))
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
    for k, v in links.items():
        print("k, v: ", k, " ", v)
        browser = webdriver.Chrome(chrome_options=options)
        browser.get(v)
        sleep(5)
        soup = BeautifulSoup(browser.page_source, "lxml")
        table = soup.find_all("table")[8]
        #rows = soup.find_all("table")[8].find("tbody").find_all("tr")
        
        #print(table)

        # Extract DataFrame
        df = pd.read_html(table.prettify())[0]
        df = df.drop([4, 5], axis=1)
        df = df.drop([0], axis=0)
        df.head()
        
        dfs[k] = df
        
        browser.quit()
```


```python
def scrape_matweb(d):
    material_property_links = {}
    
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
    material_property_links = extract_links_alt(d)
    
    # Go to the next page
    #go_next_page(d)
    
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
    a.text:  Chromium, Cr; As-Swaged
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
    a.text:  Thulium, Tm
    a.text:  Uranium, U; Cast
    a.text:  Uranium, U; Wrought Alpha Phase
    a.text:  Vanadium, V; Cold Rolled
    a.text:  Vanadium, V; Vacuum Annealed Wire
    a.text:  Vanadium, V; Hot Rolled Bar
    a.text:  Vanadium, V; Vacuum Annealed Sheet
    a.text:  Vanadium, V; Cold Drawn Wire
    a.text:  Yttrium, Y; Annealed Rod
    a.text:  Zirconium, Zr
    a.text:  Morgan Advanced Ceramics Nioro™ Mac-Nioro™-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics Nicoro™ Mac-Nicoro™-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics 35 Aj-65Cu Mac-35 Au/65 Cu-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics Paniro™ Mac-Palniro™7-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics Nioro™ ABA Mac-Nioro™ABA-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics Incusil™ Mac-Incusil™ ABA-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics Cusil™ Mac-Cusil™-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics Mac-Copper OFHC-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics 50Au-50Cu Mac-560Au/50 Cu-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics Mac-Copper ABA™L-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics Nicusil™ Mac-Nicusil™ 3-WM Braze Alloy
    a.text:  Morgan Advanced Ceramics Palcusil™ Mac-Palcusil™10-WM Braze Alloy
    a.text:  Overview of materials for AISI 4000 Series Steel
    a.text:  Overview of materials for AISI 5000 Series Steel
    a.text:  Overview of materials for AISI 6000 Series Steel
    a.text:  Overview of materials for AISI 8000 Series Steel
    a.text:  Overview of materials for AISI 9000 Series Steel
    a.text:  Overview of materials for Low Alloy Steel
    a.text:  Overview of materials for Medium Alloy Steel
    a.text:  Overview of materials for Austenitic
    a.text:  Overview of materials for AISI 1000 Series Steel
    a.text:  Overview of materials for High Carbon Steel
    a.text:  Overview of materials for Low Carbon Steel
    a.text:  Overview of materials for Medium Carbon Steel
    a.text:  Overview of materials for Cast Iron
    a.text:  Overview of materials for Alloy Cast Iron
    a.text:  Overview of materials for Chrome-moly Steel
    a.text:  Overview of materials for Ductile Iron
    a.text:  Overview of materials for Duplex
    a.text:  Overview of materials for Gray Cast Iron
    a.text:  Overview of materials for Maraging Steel
    a.text:  Overview of materials for Martensitic
    a.text:  Overview of materials for Stainless Steel
    a.text:  Overview of materials for Cast Stainless Steel
    a.text:  Overview of materials for Precipitation Hardening Stainless
    a.text:  Overview of materials for T 300 Series Stainless Steel
    a.text:  Overview of materials for T 400 Series Stainless Steel
    a.text:  Overview of materials for T 600 Series Stainless Steel
    a.text:  Overview of materials for T S10000 Series Stainless Steel
    a.text:  Overview of materials for Air-Hardening Steel
    a.text:  Overview of materials for Cold Work Steel
    a.text:  Overview of materials for Hot Work Steel
    a.text:  Overview of materials for Mold Steel
    a.text:  Overview of materials for Oil-Hardening Steel
    a.text:  Overview of materials for Shock-Resisting Steel
    a.text:  Overview of materials for Water-Hardening Steel
    a.text:  Overview of materials for White Cast Iron
    a.text:  Overview of materials for Aluminum Alloy
    a.text:  Overview of materials for 1000 Series Aluminum
    a.text:  Overview of materials for 2000 Series Aluminum Alloy
    a.text:  Overview of materials for 3000 Series Aluminum Alloy
    a.text:  Overview of materials for 4000 Series Aluminum Alloy
    a.text:  Overview of materials for 5000 Series Aluminum Alloy
    a.text:  Overview of materials for 6000 Series Aluminum Alloy
    a.text:  Overview of materials for 7000 Series Aluminum Alloy
    a.text:  Overview of materials for Aluminum Casting Alloy
    a.text:  Overview of materials for Beryllium Alloy
    a.text:  Overview of materials for Bismuth Alloy
    a.text:  Overview of materials for Cobalt Alloy
    a.text:  Overview of materials for Copper Alloy
    a.text:  Overview of materials for Brass
    a.text:  Overview of materials for Bronze
    a.text:  Overview of materials for Copper Casting Alloy
    a.text:  Overview of materials for Wrought Copper
    a.text:  Overview of materials for Indium Alloy
    a.text:  Overview of materials for Lead Alloy
    a.text:  Overview of materials for Magnesium Alloy
    a.text:  Overview of materials for Molybdenum Alloy
    a.text:  Overview of materials for Nickel Alloy
    a.text:  Overview of materials for Niobium Alloy
    a.text:  Overview of materials for Gold Alloy
    a.text:  Overview of materials for Palladium Alloy
    a.text:  Overview of materials for Platinum Alloy
    a.text:  Overview of materials for Silver Alloy
    a.text:  Overview of materials for Refractory Metal
    a.text:  Overview of materials for Solder/Braze Alloy
    a.text:  Overview of materials for Tin Alloy
    a.text:  Overview of materials for Alpha/Beta Titanium Alloy
    a.text:  Overview of materials for Alpha/Near Alpha Titanium Alloy
    a.text:  Overview of materials for Beta Titanium Alloy
    a.text:  Overview of materials for Unalloyed/Modified Titanium
    a.text:  Overview of materials for Tungsten Alloy
    a.text:  Overview of materials for Zinc Alloy
    a.text:  Overview of materials for Zirconium Alloy
    a.text:  AISI 1006 Steel, cold drawn
    a.text:  AISI 1006 Steel, hot rolled bar, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1006 Steel, cold drawn bar, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1006 Steel, CQ, DQ, and DQSK sheet, 1.6-5.8 (0.06-0.23 in) mm thick
    a.text:  AISI 1008 Steel, hot rolled bar, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1008 Steel, cold drawn bar, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1008 Steel, CQ, DQ, and DQSK sheet, 1.6-5.8 mm thick
    a.text:  AISI 1010 Steel, cold drawn
    a.text:  AISI 1010 Steel, hot rolled bar, 19-32 mm (0.75-1.25 in) round or thickness
    a.text:  AISI 1010 Steel, cold drawn bar, 19-32 mm (0.75-1.25 in) round or thickness
    a.text:  AISI 1010 Steel, CQ sheet,1.6-5.8 mm round or thickness
    a.text:  AISI 1012 Steel, cold drawn
    a.text:  AISI 1012 Steel, hot rolled bar, 19-32 mm (0.75-1.25 in) round or thickness
    a.text:  AISI 1012 Steel, cold drawn bar, 19-32 mm (0.75-1.25 in) round or thickness
    a.text:  AISI 1012 Steel, CQ sheet,1.6-5.8 mm round or thickness
    a.text:  AISI 1015 Steel, cold drawn
    a.text:  AISI 1015 Steel, cold drawn, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1015 Steel, hot rolled, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1015 Steel, as rolled
    a.text:  AISI 1015 Steel, normalized at 925°C (1700°F)
    a.text:  AISI 1015 Steel, annealed at 870°C (1600°F)
    a.text:  AISI 1015 Steel, annealed at 870°C (1600°F), furnace cooled 17°C (31°F) per hour to 725°C (1340°F), air cooled, 25 mm (1 in.) round
    a.text:  AISI 1015 Steel, normalized at 925°C (1700°F), 13 mm (0.5 in.) round
    a.text:  AISI 1015 Steel, normalized at 925°C (1700°F), 25 mm (1 in.) round
    a.text:  AISI 1015 Steel, normalized at 925°C (1700°F), 50 mm (2 in.) round
    a.text:  AISI 1015 Steel, normalized at 925°C (1700°F), 100 mm (4 in.) round
    a.text:  AISI 1015 Steel, mock carburized at 915°C (1680°F) for 8 hours, 775°C (1430°F) reheat, water quenched, 175°C (350°F) temper, 13 mm (0.5 in.) round
    a.text:  AISI 1015 Steel, mock carburized at 915°C (1680°F) for 8 hours, 775°C (1430°F) reheat, water quenched, 175°C (350°F) temper, 25 mm (1 in.) round
    a.text:  AISI 1015 Steel, mock carburized at 915°C (1680°F) for 8 hours, 775°C (1430°F) reheat, water quenched, 175°C (350°F) temper, 50 mm (2 in.) round
    a.text:  AISI 1015 Steel, mock carburized at 915°C (1680°F) for 8 hours, 775°C (1430°F) reheat, water quenched, 175°C (350°F) temper, 100 mm (4 in.) round
    a.text:  AISI 1016 Steel, cold drawn, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1016 Steel, hot rolled, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1017 Steel, cold drawn
    a.text:  AISI 1017 Steel, hot rolled, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1018 Steel, cold drawn
    a.text:  AISI 1018 Steel, hot rolled, quenched, and tempered
    a.text:  AISI 1018 Steel, hot rolled, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1018 Steel, cold drawn, quenched, and tempered, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1018 Steel, as cold drawn, 16-22 mm (0.625-0.875 in) round
    a.text:  AISI 1018 Steel, as cold drawn, 22-32 mm (0.875-1.25 in) round
    a.text:  AISI 1018 Steel, as cold drawn, 32-50 mm (1.25-2 in) round
    a.text:  AISI 1018 Steel, as cold drawn, 50-76 mm round
    a.text:  AISI 1018 Steel, cold drawn, high temperature, stress relieved, 16-22 mm (0.625-0.875 in) round
    a.text:  AISI 1018 Steel, cold drawn, high temperature, stress relieved, 22-32 mm (0.875-1.25 in) round
    a.text:  AISI 1018 Steel, cold drawn, high temperature, stress relieved, 32-50 mm (1.25-2 in) round
    a.text:  AISI 1018 Steel, cold drawn, high temperature, stress relieved, 50-76 mm round
    a.text:  AISI 1018 Steel, carburized at 925°C (1700°F), box cooled, 775°C (1430°F) reheat, water quenched, 175°C (350°F) temper. 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1019 Steel, cold drawn
    a.text:  AISI 1019 Steel, hot rolled, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1020 Steel, cold rolled
    a.text:  AISI 1020 Steel, hot rolled, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1020 Steel, hot rolled, quenched and tempered, 0.2% offset, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1020 Steel, as rolled
    a.text:  AISI 1020 Steel, normalized at 870°C (1600°F)
    a.text:  AISI 1020 Steel, annealed at 870°C (1600°F)
    a.text:  AISI 1020 Steel, as rolled, 25 mm (1 in.) round
    a.text:  AISI 1020 Steel, annealed at 870°C (1600°F), furnace cooled 17°C (31°F) per hour to 700°C, air cooled, 25 mm (1 in.) round
    a.text:  AISI 1020 Steel, normalized at 925°C (1700°F), air cooled, 13 mm (0.5 in.) round
    a.text:  AISI 1020 Steel, normalized at 925°C (1700°F), air cooled, 25 mm (1 in.) round
    a.text:  AISI 1020 Steel, normalized at 925°C (1700°F), air cooled, 50 mm (2 in.) round
    a.text:  AISI 1020 Steel, normalized at 925°C (1700°F), air cooled, 100 mm (4 in.) round
    a.text:  AISI 1020 Steel, mock carburized at 915°C (1680°F) for 8 hours, 775°C (1430°F) reheat, water quenched, 175°C (350°F) temper, 13 mm (0.5 in.) round
    a.text:  AISI 1020 Steel, mock carburized at 915°C (1680°F) for 8 hours, 775°C (1430°F) reheat, water quenched, 175°C (350°F) temper, 25 mm (1 in.) round
    a.text:  AISI 1020 Steel, mock carburized at 915°C (1680°F) for 8 hours, 775°C (1430°F) reheat, water quenched, 175°C (350°F) temper, 50 mm (2 in.) round
    a.text:  AISI 1020 Steel, mock carburized at 915°C (1680°F) for 8 hours, 775°C (1430°F) reheat, water quenched, 175°C (350°F) temper, 100 mm (4 in.) round
    a.text:  AISI 1021 Steel, cold drawn
    a.text:  AISI 1021 Steel, hot rolled, 19-32 mm (0.75-1.25 in) round
    a.text:  AISI 1021 Steel, cold rolled, 25 mm (1 in.) round
    a.text:  AISI 1021 Steel, annealed at 870°C (1600°F), furnace cooled 17°C (31°F) per hour to 675°C, air cooled, 25 mm (1 in.) round
    a.text:  AISI 1021 Steel, normalized at 925°C (1700°F), air cooled, 13 mm (0.5 in.) round
    k, v:  Gold, Au   http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e
    k, v:  Beryllium, Be   http://www.matweb.com/search/DataSheet.aspx?MatGUID=8a6a0df6122349b7bdc92662658d4a4f
    k, v:  Calcium, Ca; Rolled   http://www.matweb.com/search/DataSheet.aspx?MatGUID=15135dd30619457f902229b03619841a



    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    <ipython-input-1001-2181b83aa89e> in <module>
          2     print("Matweb Scraper v1")
          3     driver = init()
    ----> 4     data = scrape_matweb(driver)
          5     driver.quit()


    <ipython-input-1000-38f8861123c9> in scrape_matweb(d)
         23 
         24     # Get properties of each material
    ---> 25     get_material_properties(material_property_links)
         26 
         27     #sleep(10)


    <ipython-input-999-0a2dbc07b443> in get_material_properties(links)
         14         # Extract DataFrame
         15         df = pd.read_html(table.prettify())[0]
    ---> 16         df = df.drop([4, 5], axis=1)
         17         df = df.drop([0], axis=0)
         18         df.head()


    ~/anaconda/lib/python3.6/site-packages/pandas/core/frame.py in drop(self, labels, axis, index, columns, level, inplace, errors)
       4100             level=level,
       4101             inplace=inplace,
    -> 4102             errors=errors,
       4103         )
       4104 


    ~/anaconda/lib/python3.6/site-packages/pandas/core/generic.py in drop(self, labels, axis, index, columns, level, inplace, errors)
       3912         for axis, labels in axes.items():
       3913             if labels is not None:
    -> 3914                 obj = obj._drop_axis(labels, axis, level=level, errors=errors)
       3915 
       3916         if inplace:


    ~/anaconda/lib/python3.6/site-packages/pandas/core/generic.py in _drop_axis(self, labels, axis, level, errors)
       3944                 new_axis = axis.drop(labels, level=level, errors=errors)
       3945             else:
    -> 3946                 new_axis = axis.drop(labels, errors=errors)
       3947             result = self.reindex(**{axis_name: new_axis})
       3948 


    ~/anaconda/lib/python3.6/site-packages/pandas/core/indexes/base.py in drop(self, labels, errors)
       5338         if mask.any():
       5339             if errors != "ignore":
    -> 5340                 raise KeyError("{} not found in axis".format(labels[mask]))
       5341             indexer = indexer[~mask]
       5342         return self.delete(indexer)


    KeyError: '[4 5] not found in axis'



```python
# html-requests

session = HTMLSession()
r = session.get("http://www.matweb.com/search/DataSheet.aspx?MatGUID=d2a2119a08904a0fa706e9408cddb88e")
r.html.links
```




    {'/Search/MaterialGroupSearch.aspx?GroupID=177',
     '/Search/MaterialGroupSearch.aspx?GroupID=180',
     '/Search/MaterialGroupSearch.aspx?GroupID=184',
     '/Search/MaterialGroupSearch.aspx?GroupID=214',
     '/Search/MaterialGroupSearch.aspx?GroupID=9',
     '/clickthrough.aspx?addataid=1239',
     '/clickthrough.aspx?addataid=277',
     '/clickthrough.aspx?addataid=3111',
     '/folders/ListFolders.aspx',
     '/help/fea_export.aspx',
     '/help/help.aspx',
     '/index.aspx',
     '/membership/login.aspx',
     '/membership/regstart.aspx',
     '/membership/regupgrade.aspx',
     '/reference/faq.aspx',
     '/reference/link.aspx',
     '/reference/privacy.aspx',
     '/reference/suppliers.aspx',
     '/reference/terms.aspx',
     '/search/AdvancedSearch.aspx',
     '/search/CompositionSearch.aspx',
     '/search/GetReference.aspx?matid=7',
     '/search/GetVendors.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e',
     '/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&propid=182&sigid=1',
     '/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&propid=695&sigid=1',
     '/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&propid=788&sigid=1',
     '/search/MaterialGroupSearch.aspx',
     '/search/PropertySearch.aspx',
     '/search/SearchManufacturerName.aspx',
     '/search/SearchTradeName.aspx',
     '/search/SearchUNS.aspx',
     '/search/datasheet.aspx?MatGUID=0cd1edf33ac145ee93a0aa6fc666c0e0',
     '/search/datasheet.aspx?MatGUID=63cbd043a31f4f739ddb7632c1443d33',
     '/search/datasheet.aspx?MatGUID=654ca9c358264b5392d43315d8535b7d',
     '/search/datasheet.aspx?MatGUID=9aebe83845c04c1db5126fada6f76f7e',
     '/search/datasheet.aspx?MatGUID=e6eb83327e534850a062dbca3bc758dc',
     '/search/datasheet.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&n=1',
     '/search/search.aspx',
     '/services/advertising.aspx',
     '/services/contact.aspx',
     '/services/databaselicense.aspx',
     '/services/services.aspx',
     '/services/submitdata.aspx',
     '/services/webhosting.aspx',
     '/tools/contents.aspx',
     '/tools/contents.aspx#reference',
     '/tools/tools.aspx',
     '/tools/unitconverter.aspx',
     '/tools/unitconverter.aspx?fromID=10&fromValue=2090',
     '/tools/unitconverter.aspx?fromID=108&fromValue=120',
     '/tools/unitconverter.aspx?fromID=11&fromValue=28.5',
     '/tools/unitconverter.aspx?fromID=11&fromValue=747.7',
     '/tools/unitconverter.aspx?fromID=115&fromValue=0.00000220',
     '/tools/unitconverter.aspx?fromID=12&fromValue=0.0306',
     '/tools/unitconverter.aspx?fromID=12&fromValue=0.0318',
     '/tools/unitconverter.aspx?fromID=12&fromValue=0.0339',
     '/tools/unitconverter.aspx?fromID=12&fromValue=0.0380',
     '/tools/unitconverter.aspx?fromID=12&fromValue=0.0390',
     '/tools/unitconverter.aspx?fromID=12&fromValue=0.0397',
     '/tools/unitconverter.aspx?fromID=12&fromValue=0.0406',
     '/tools/unitconverter.aspx?fromID=123&fromValue=17400',
     '/tools/unitconverter.aspx?fromID=136&fromValue=301',
     '/tools/unitconverter.aspx?fromID=144&fromValue=0.15344',
     '/tools/unitconverter.aspx?fromID=144&fromValue=0.850',
     '/tools/unitconverter.aspx?fromID=144&fromValue=0.86378',
     '/tools/unitconverter.aspx?fromID=144&fromValue=0.90277',
     '/tools/unitconverter.aspx?fromID=144&fromValue=1.04028',
     '/tools/unitconverter.aspx?fromID=144&fromValue=1.37',
     '/tools/unitconverter.aspx?fromID=146&fromValue=1.68',
     '/tools/unitconverter.aspx?fromID=2&fromValue=1064.43',
     '/tools/unitconverter.aspx?fromID=2&fromValue=2856',
     '/tools/unitconverter.aspx?fromID=251&fromValue=0.7598',
     '/tools/unitconverter.aspx?fromID=251&fromValue=7.598',
     '/tools/unitconverter.aspx?fromID=251&fromValue=75.98',
     '/tools/unitconverter.aspx?fromID=251&fromValue=759.8',
     '/tools/unitconverter.aspx?fromID=3&fromValue=1947.97',
     '/tools/unitconverter.aspx?fromID=3&fromValue=5173',
     '/tools/unitconverter.aspx?fromID=300&fromValue=99',
     '/tools/unitconverter.aspx?fromID=301&fromValue=2.45',
     '/tools/unitconverter.aspx?fromID=4&fromValue=8.00',
     '/tools/unitconverter.aspx?fromID=4&fromValue=8.11',
     '/tools/unitconverter.aspx?fromID=4&fromValue=8.44',
     '/tools/unitconverter.aspx?fromID=4&fromValue=9.28',
     '/tools/unitconverter.aspx?fromID=43&fromValue=19.32',
     '/tools/unitconverter.aspx?fromID=45&fromValue=27.2',
     '/tools/unitconverter.aspx?fromID=45&fromValue=77.2',
     '/tools/unitconverter.aspx?fromID=5&fromValue=14.4',
     '/tools/unitconverter.aspx?fromID=5&fromValue=14.6',
     '/tools/unitconverter.aspx?fromID=5&fromValue=15.2',
     '/tools/unitconverter.aspx?fromID=5&fromValue=16.7',
     '/tools/unitconverter.aspx?fromID=64&fromValue=1738',
     '/tools/unitconverter.aspx?fromID=64&fromValue=66.2',
     '/tools/unitconverter.aspx?fromID=65&fromValue=0.128',
     '/tools/unitconverter.aspx?fromID=65&fromValue=0.133',
     '/tools/unitconverter.aspx?fromID=65&fromValue=0.142',
     '/tools/unitconverter.aspx?fromID=65&fromValue=0.159',
     '/tools/unitconverter.aspx?fromID=65&fromValue=0.163',
     '/tools/unitconverter.aspx?fromID=65&fromValue=0.166',
     '/tools/unitconverter.aspx?fromID=65&fromValue=0.170',
     '/tools/unitconverter.aspx?fromID=78&fromValue=11200',
     '/tools/unitconverter.aspx?fromID=78&fromValue=3950',
     '/tools/unitconverter.aspx?fromID=8&fromValue=0.001013',
     '/tools/unitconverter.aspx?fromID=8&fromValue=0.01013',
     '/tools/unitconverter.aspx?fromID=8&fromValue=0.1013',
     '/tools/unitconverter.aspx?fromID=8&fromValue=1.013',
     '/tools/unitconverter.aspx?fromID=87&fromValue=0.6980',
     'https://twitter.com/MatWeb'}




```python
browser.quit()
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-968-710928f8954f> in <module>
    ----> 1 browser.quit()
    

    NameError: name 'browser' is not defined



```python
driver.quit()
```


```python
'''def scrape_matweb(d):
    material_property_links = []

    # Select material category
    select_material_category(d)
    
    # Select material properties
    select_material_property_1(d)
    select_material_property_2(d)
    select_material_property_3(d)
    
    # Submit query
    submit_query(d)
    
    select_view(d, "200")
    
    # Extract material links
    while go_next_page:
            material_property_links = material_property_links + extract_links(d)
        
    sleep(10)'''
```




    'def scrape_matweb(d):\n    # Select material category\n    select_material_category(d)\n    \n    # Select material properties\n    select_material_property_1(d)\n    select_material_property_2(d)\n    select_material_property_3(d)\n    \n    # Submit query\n    submit_query(d)\n    \n    select_view(d, "200")\n    \n    # Extract material links\n    while go_next_page:\n        extract_links(d)\n    sleep(10)'




```python
# Extract DataFrame
df = pd.read_html(table.prettify())[0]
df
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
      <th>0</th>
      <th>1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Printer friendly version  Download as PDF  Dow...</td>
      <td>Add to Folder:  My Folder</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Drop excess rows and columns
df = df.drop([4, 5], axis=1)
df = df.drop([0], axis=0)
df
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    <ipython-input-859-3e5b2e506f1e> in <module>
          1 # Drop excess rows and columns
    ----> 2 df = df.drop([4, 5], axis=1)
          3 df = df.drop([0], axis=0)
          4 df


    ~/anaconda/lib/python3.6/site-packages/pandas/core/frame.py in drop(self, labels, axis, index, columns, level, inplace, errors)
       4100             level=level,
       4101             inplace=inplace,
    -> 4102             errors=errors,
       4103         )
       4104 


    ~/anaconda/lib/python3.6/site-packages/pandas/core/generic.py in drop(self, labels, axis, index, columns, level, inplace, errors)
       3912         for axis, labels in axes.items():
       3913             if labels is not None:
    -> 3914                 obj = obj._drop_axis(labels, axis, level=level, errors=errors)
       3915 
       3916         if inplace:


    ~/anaconda/lib/python3.6/site-packages/pandas/core/generic.py in _drop_axis(self, labels, axis, level, errors)
       3944                 new_axis = axis.drop(labels, level=level, errors=errors)
       3945             else:
    -> 3946                 new_axis = axis.drop(labels, errors=errors)
       3947             result = self.reindex(**{axis_name: new_axis})
       3948 


    ~/anaconda/lib/python3.6/site-packages/pandas/core/indexes/base.py in drop(self, labels, errors)
       5338         if mask.any():
       5339             if errors != "ignore":
    -> 5340                 raise KeyError("{} not found in axis".format(labels[mask]))
       5341             indexer = indexer[~mask]
       5342         return self.delete(indexer)


    KeyError: '[4 5] not found in axis'



```python
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>Physical Properties</td>
      <td>Metric</td>
      <td>English</td>
      <td>Comments</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Density</td>
      <td>19.32  g/cc</td>
      <td>0.6980  lb/in³</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Vapor Pressure</td>
      <td>0.001013  bar  @Temperature 1770 °C</td>
      <td>0.7598  torr  @Temperature 3220 °F</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>NaN</td>
      <td>0.01013  bar  @Temperature 2036 °C</td>
      <td>7.598  torr  @Temperature 3697 °F</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>5</td>
      <td>NaN</td>
      <td>0.1013  bar  @Temperature 2383 °C</td>
      <td>75.98  torr  @Temperature 4321 °F</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>58</td>
      <td>Component Elements Properties</td>
      <td>Metric</td>
      <td>English</td>
      <td>Comments</td>
    </tr>
    <tr>
      <td>59</td>
      <td>Gold, Au</td>
      <td>100 %</td>
      <td>100 %</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>60</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>61</td>
      <td>Descriptive Properties</td>
      <td>Descriptive Properties</td>
      <td>Descriptive Properties</td>
      <td>Descriptive Properties</td>
    </tr>
    <tr>
      <td>62</td>
      <td>CAS Number</td>
      <td>7440-57-5</td>
      <td>7440-57-5</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>62 rows × 4 columns</p>
</div>




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

    ######################################### print_all_labels
    [<td style="vertical-align:top;">Density </td>, <td style="vertical-align:top;">Vapor Pressure <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=788&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;">Atomic Mass </td>, <td style="vertical-align:top;">Atomic Number </td>, <td style="vertical-align:top;">Thermal Neutron Cross Section </td>, <td style="vertical-align:top;">X-ray Absorption Edge </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;">Electrode Potential </td>, <td style="vertical-align:top;">Electronegativity </td>, <td style="vertical-align:top;">Ionic Radius </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;">Electrochemical Equivalent </td>, <td style="vertical-align:top;">Hardness, Vickers </td>, <td style="vertical-align:top;">Tensile Strength, Ultimate </td>, <td style="vertical-align:top;">Elongation at Break </td>, <td style="vertical-align:top;">Modulus of Elasticity </td>, <td style="vertical-align:top;">Poissons Ratio </td>, <td style="vertical-align:top;">Shear Modulus </td>, <td style="vertical-align:top;">Electrical Resistivity </td>, <td style="vertical-align:top;">Magnetic Susceptibility </td>, <td style="vertical-align:top;">Heat of Fusion </td>, <td style="vertical-align:top;">Heat of Vaporization </td>, <td style="vertical-align:top;">CTE, linear <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=182&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;">Specific Heat Capacity <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=695&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;">Thermal Conductivity </td>, <td style="vertical-align:top;">Melting Point </td>, <td style="vertical-align:top;">Boiling Point </td>, <td style="vertical-align:top;">Emissivity (0-1) </td>, <td style="vertical-align:top;">Reflection Coefficient, Visible (0-1) </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;"> </td>, <td style="vertical-align:top;">Gold, Au </td>]
    #########################################
    <td style="vertical-align:top;">Density </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=43&amp;fromValue=19.32" title="Click to see this value in other UOMs">19.32</a> g/cc<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=87&amp;fromValue=0.6980" title="Click to see this value in other UOMs">0.6980</a> lb/in³<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Vapor Pressure <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=788&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=0.001013" title="Click to see this value in other UOMs">0.001013</a> bar<span class="dataCondition"><br/>@Temperature 1770 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=0.7598" title="Click to see this value in other UOMs">0.7598</a> torr<span class="dataCondition"><br/>@Temperature 3220 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=0.01013" title="Click to see this value in other UOMs">0.01013</a> bar<span class="dataCondition"><br/>@Temperature 2036 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=7.598" title="Click to see this value in other UOMs">7.598</a> torr<span class="dataCondition"><br/>@Temperature 3697 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=0.1013" title="Click to see this value in other UOMs">0.1013</a> bar<span class="dataCondition"><br/>@Temperature 2383 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=75.98" title="Click to see this value in other UOMs">75.98</a> torr<span class="dataCondition"><br/>@Temperature 4321 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=1.013" title="Click to see this value in other UOMs">1.013</a> bar<span class="dataCondition"><br/>@Temperature 2857 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=759.8" title="Click to see this value in other UOMs">759.8</a> torr<span class="dataCondition"><br/>@Temperature 5175 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Atomic Mass </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">196.9666<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">196.9666<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">196.96655 - 1995</td>>
    <td style="vertical-align:top;">Atomic Number </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">79<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">79<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Thermal Neutron Cross Section </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=300&amp;fromValue=99" title="Click to see this value in other UOMs">99</a> barns/atom<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=300&amp;fromValue=99" title="Click to see this value in other UOMs">99</a> barns/atom<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">X-ray Absorption Edge </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.15344" title="Click to see this value in other UOMs">0.15344</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.15344" title="Click to see this value in other UOMs">0.15344</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">K</td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.86378" title="Click to see this value in other UOMs">0.86378</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.86378" title="Click to see this value in other UOMs">0.86378</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">L<sub>I</sub></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.90277" title="Click to see this value in other UOMs">0.90277</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.90277" title="Click to see this value in other UOMs">0.90277</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">L<sub>II</sub></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.04028" title="Click to see this value in other UOMs">1.04028</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.04028" title="Click to see this value in other UOMs">1.04028</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">L<sub>III</sub></td>>
    <td style="vertical-align:top;">Electrode Potential </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=146&amp;fromValue=1.68" title="Click to see this value in other UOMs">1.68</a> V<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=146&amp;fromValue=1.68" title="Click to see this value in other UOMs">1.68</a> V<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Electronegativity </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">2.4<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">2.4<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">Pauling</td>>
    <td style="vertical-align:top;">Ionic Radius </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.850" title="Click to see this value in other UOMs">0.850</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.850" title="Click to see this value in other UOMs">0.850</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">Crystal Ionic Radius for Valence +3</td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.37" title="Click to see this value in other UOMs">1.37</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.37" title="Click to see this value in other UOMs">1.37</a> Å<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">Crystal Ionic Radius for Valence +1</td>>
    <td style="vertical-align:top;">Electrochemical Equivalent </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=301&amp;fromValue=2.45" title="Click to see this value in other UOMs">2.45</a> g/A/h<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=301&amp;fromValue=2.45" title="Click to see this value in other UOMs">2.45</a> g/A/h<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Hardness, Vickers </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">25<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">25<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Tensile Strength, Ultimate </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=120" title="Click to see this value in other UOMs">120</a> MPa<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=17400" title="Click to see this value in other UOMs">17400</a> psi<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">Annealed</td>>
    <td style="vertical-align:top;">Elongation at Break </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">30 %<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">30 %<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Modulus of Elasticity </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=45&amp;fromValue=77.2" title="Click to see this value in other UOMs">77.2</a> GPa<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=78&amp;fromValue=11200" title="Click to see this value in other UOMs">11200</a> ksi<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">60% Cold Worked</td>>
    <td style="vertical-align:top;">Poissons Ratio </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.42<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.42<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Shear Modulus </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=45&amp;fromValue=27.2" title="Click to see this value in other UOMs">27.2</a> GPa<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=78&amp;fromValue=3950" title="Click to see this value in other UOMs">3950</a> ksi<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">60% Cold Worked; Calculated Value</td>>
    <td style="vertical-align:top;">Electrical Resistivity </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.00000220" title="Click to see this value in other UOMs">0.00000220</a> ohm-cm<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.00000220" title="Click to see this value in other UOMs">0.00000220</a> ohm-cm<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Magnetic Susceptibility </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">-1.42e-7<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">-1.42e-7<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">cgs/g</td>>
    <td style="vertical-align:top;">Heat of Fusion </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=64&amp;fromValue=66.2" title="Click to see this value in other UOMs">66.2</a> J/g<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=11&amp;fromValue=28.5" title="Click to see this value in other UOMs">28.5</a> BTU/lb<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Heat of Vaporization </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=64&amp;fromValue=1738" title="Click to see this value in other UOMs">1738</a> J/g<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=11&amp;fromValue=747.7" title="Click to see this value in other UOMs">747.7</a> BTU/lb<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">CTE, linear <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=182&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=14.4" title="Click to see this value in other UOMs">14.4</a> µm/m-°C<span class="dataCondition"><br/>@Temperature 20.0 - 100 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=8.00" title="Click to see this value in other UOMs">8.00</a> µin/in-°F<span class="dataCondition"><br/>@Temperature 68.0 - 212 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=14.6" title="Click to see this value in other UOMs">14.6</a> µm/m-°C<span class="dataCondition"><br/>@Temperature 250 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=8.11" title="Click to see this value in other UOMs">8.11</a> µin/in-°F<span class="dataCondition"><br/>@Temperature 482 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=15.2" title="Click to see this value in other UOMs">15.2</a> µm/m-°C<span class="dataCondition"><br/>@Temperature 500 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=8.44" title="Click to see this value in other UOMs">8.44</a> µin/in-°F<span class="dataCondition"><br/>@Temperature 932 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=16.7" title="Click to see this value in other UOMs">16.7</a> µm/m-°C<span class="dataCondition"><br/>@Temperature 950 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=9.28" title="Click to see this value in other UOMs">9.28</a> µin/in-°F<span class="dataCondition"><br/>@Temperature 1740 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Specific Heat Capacity <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=695&amp;sigid=1" title="Graph this set of conditional data"><img alt="" src="/images/smallchart.gif"/></a> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.128" title="Click to see this value in other UOMs">0.128</a> J/g-°C<span class="dataCondition"><br/>@Temperature 25.0 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0306" title="Click to see this value in other UOMs">0.0306</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 77.0 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.133" title="Click to see this value in other UOMs">0.133</a> J/g-°C<span class="dataCondition"><br/>@Temperature 227 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0318" title="Click to see this value in other UOMs">0.0318</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 441 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.142" title="Click to see this value in other UOMs">0.142</a> J/g-°C<span class="dataCondition"><br/>@Temperature 627 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0339" title="Click to see this value in other UOMs">0.0339</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 1160 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.159" title="Click to see this value in other UOMs">0.159</a> J/g-°C<span class="dataCondition"><br/>@Temperature 1227 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0380" title="Click to see this value in other UOMs">0.0380</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 2241 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.163" title="Click to see this value in other UOMs">0.163</a> J/g-°C<span class="dataCondition"><br/>@Temperature 1027 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0390" title="Click to see this value in other UOMs">0.0390</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 1881 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.166" title="Click to see this value in other UOMs">0.166</a> J/g-°C<span class="dataCondition"><br/>@Temperature 1127 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0397" title="Click to see this value in other UOMs">0.0397</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 2061 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.170" title="Click to see this value in other UOMs">0.170</a> J/g-°C<span class="dataCondition"><br/>@Temperature 1063 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0406" title="Click to see this value in other UOMs">0.0406</a> BTU/lb-°F<span class="dataCondition"><br/>@Temperature 1945 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Thermal Conductivity </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=136&amp;fromValue=301" title="Click to see this value in other UOMs">301</a> W/m-K<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=10&amp;fromValue=2090" title="Click to see this value in other UOMs">2090</a> BTU-in/hr-ft²-°F<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Melting Point </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=1064.43" title="Click to see this value in other UOMs">1064.43</a> °C<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=1947.97" title="Click to see this value in other UOMs">1947.97</a> °F<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Boiling Point </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=2856" title="Click to see this value in other UOMs">2856</a> °C<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;"><a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=5173" title="Click to see this value in other UOMs">5173</a> °F<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>
    <td style="vertical-align:top;">Emissivity (0-1) </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.040<span class="dataCondition"><br/>@Temperature 100 °C</span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.040<span class="dataCondition"><br/>@Temperature 212 °F</span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">total spectrum</td>>
    <td style="vertical-align:top;">Reflection Coefficient, Visible (0-1) </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.27<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.27<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">400 nm</td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.50<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.50<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">500 nm</td>>
    <td style="vertical-align:top;"> </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.85<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">0.85<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;">600 nm</td>>
    <td style="vertical-align:top;">Gold, Au </td>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">100 %<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataCell" style="vertical-align:top;">100 %<span class="dataCondition"></span></td>>
    <bound method Tag.get_text of <td class="dataComment" style="vertical-align:top;"></td>>



```python
properties_table = display_html(table.prettify(), raw=True)
properties_table
```


<table cellspacing="0" class="tabledataformat">
 <tbody>
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
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=43&amp;fromValue=19.32" title="Click to see this value in other UOMs">
     19.32
    </a>
    g/cc
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=87&amp;fromValue=0.6980" title="Click to see this value in other UOMs">
     0.6980
    </a>
    lb/in³
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Vapor Pressure
    <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=788&amp;sigid=1" title="Graph this set of conditional data">
     <img alt="" src="/images/smallchart.gif"/>
    </a>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=0.001013" title="Click to see this value in other UOMs">
     0.001013
    </a>
    bar
    <span class="dataCondition">
     <br/>
     @Temperature 1770 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=0.7598" title="Click to see this value in other UOMs">
     0.7598
    </a>
    torr
    <span class="dataCondition">
     <br/>
     @Temperature 3220 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=0.01013" title="Click to see this value in other UOMs">
     0.01013
    </a>
    bar
    <span class="dataCondition">
     <br/>
     @Temperature 2036 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=7.598" title="Click to see this value in other UOMs">
     7.598
    </a>
    torr
    <span class="dataCondition">
     <br/>
     @Temperature 3697 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=0.1013" title="Click to see this value in other UOMs">
     0.1013
    </a>
    bar
    <span class="dataCondition">
     <br/>
     @Temperature 2383 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=75.98" title="Click to see this value in other UOMs">
     75.98
    </a>
    torr
    <span class="dataCondition">
     <br/>
     @Temperature 4321 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=8&amp;fromValue=1.013" title="Click to see this value in other UOMs">
     1.013
    </a>
    bar
    <span class="dataCondition">
     <br/>
     @Temperature 2857 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=251&amp;fromValue=759.8" title="Click to see this value in other UOMs">
     759.8
    </a>
    torr
    <span class="dataCondition">
     <br/>
     @Temperature 5175 °F
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
    196.9666
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    196.9666
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    196.96655 - 1995
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Atomic Number
   </td>
   <td class="dataCell" style="vertical-align:top;">
    79
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    79
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="altrow datarowSeparator">
   <td style="vertical-align:top;">
    Thermal Neutron Cross Section
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=300&amp;fromValue=99" title="Click to see this value in other UOMs">
     99
    </a>
    barns/atom
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=300&amp;fromValue=99" title="Click to see this value in other UOMs">
     99
    </a>
    barns/atom
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    X-ray Absorption Edge
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.15344" title="Click to see this value in other UOMs">
     0.15344
    </a>
    Å
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.15344" title="Click to see this value in other UOMs">
     0.15344
    </a>
    Å
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    K
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.86378" title="Click to see this value in other UOMs">
     0.86378
    </a>
    Å
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.86378" title="Click to see this value in other UOMs">
     0.86378
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
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.90277" title="Click to see this value in other UOMs">
     0.90277
    </a>
    Å
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.90277" title="Click to see this value in other UOMs">
     0.90277
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
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.04028" title="Click to see this value in other UOMs">
     1.04028
    </a>
    Å
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.04028" title="Click to see this value in other UOMs">
     1.04028
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
  <tr class="altrow datarowSeparator">
   <td style="vertical-align:top;">
    Electrode Potential
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=146&amp;fromValue=1.68" title="Click to see this value in other UOMs">
     1.68
    </a>
    V
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=146&amp;fromValue=1.68" title="Click to see this value in other UOMs">
     1.68
    </a>
    V
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Electronegativity
   </td>
   <td class="dataCell" style="vertical-align:top;">
    2.4
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    2.4
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
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.850" title="Click to see this value in other UOMs">
     0.850
    </a>
    Å
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=0.850" title="Click to see this value in other UOMs">
     0.850
    </a>
    Å
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    Crystal Ionic Radius for Valence +3
   </td>
  </tr>
  <tr class="altrow datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.37" title="Click to see this value in other UOMs">
     1.37
    </a>
    Å
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=144&amp;fromValue=1.37" title="Click to see this value in other UOMs">
     1.37
    </a>
    Å
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    Crystal Ionic Radius for Valence +1
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Electrochemical Equivalent
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=301&amp;fromValue=2.45" title="Click to see this value in other UOMs">
     2.45
    </a>
    g/A/h
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=301&amp;fromValue=2.45" title="Click to see this value in other UOMs">
     2.45
    </a>
    g/A/h
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
    Hardness, Vickers
   </td>
   <td class="dataCell" style="vertical-align:top;">
    25
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    25
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Tensile Strength, Ultimate
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=108&amp;fromValue=120" title="Click to see this value in other UOMs">
     120
    </a>
    MPa
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=123&amp;fromValue=17400" title="Click to see this value in other UOMs">
     17400
    </a>
    psi
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    Annealed
   </td>
  </tr>
  <tr class="altrow datarowSeparator">
   <td style="vertical-align:top;">
    Elongation at Break
   </td>
   <td class="dataCell" style="vertical-align:top;">
    30 %
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    30 %
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Modulus of Elasticity
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=45&amp;fromValue=77.2" title="Click to see this value in other UOMs">
     77.2
    </a>
    GPa
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=78&amp;fromValue=11200" title="Click to see this value in other UOMs">
     11200
    </a>
    ksi
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    60% Cold Worked
   </td>
  </tr>
  <tr class="altrow datarowSeparator">
   <td style="vertical-align:top;">
    Poissons Ratio
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.42
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.42
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Shear Modulus
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=45&amp;fromValue=27.2" title="Click to see this value in other UOMs">
     27.2
    </a>
    GPa
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=78&amp;fromValue=3950" title="Click to see this value in other UOMs">
     3950
    </a>
    ksi
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    60% Cold Worked; Calculated Value
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
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.00000220" title="Click to see this value in other UOMs">
     0.00000220
    </a>
    ohm-cm
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=115&amp;fromValue=0.00000220" title="Click to see this value in other UOMs">
     0.00000220
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
    -1.42e-7
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    -1.42e-7
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    cgs/g
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
    Heat of Fusion
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=64&amp;fromValue=66.2" title="Click to see this value in other UOMs">
     66.2
    </a>
    J/g
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=11&amp;fromValue=28.5" title="Click to see this value in other UOMs">
     28.5
    </a>
    BTU/lb
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Heat of Vaporization
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=64&amp;fromValue=1738" title="Click to see this value in other UOMs">
     1738
    </a>
    J/g
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=11&amp;fromValue=747.7" title="Click to see this value in other UOMs">
     747.7
    </a>
    BTU/lb
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="altrow datarowSeparator">
   <td style="vertical-align:top;">
    CTE, linear
    <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=182&amp;sigid=1" title="Graph this set of conditional data">
     <img alt="" src="/images/smallchart.gif"/>
    </a>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=14.4" title="Click to see this value in other UOMs">
     14.4
    </a>
    µm/m-°C
    <span class="dataCondition">
     <br/>
     @Temperature 20.0 - 100 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=8.00" title="Click to see this value in other UOMs">
     8.00
    </a>
    µin/in-°F
    <span class="dataCondition">
     <br/>
     @Temperature 68.0 - 212 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="altrow datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=14.6" title="Click to see this value in other UOMs">
     14.6
    </a>
    µm/m-°C
    <span class="dataCondition">
     <br/>
     @Temperature 250 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=8.11" title="Click to see this value in other UOMs">
     8.11
    </a>
    µin/in-°F
    <span class="dataCondition">
     <br/>
     @Temperature 482 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="altrow datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=15.2" title="Click to see this value in other UOMs">
     15.2
    </a>
    µm/m-°C
    <span class="dataCondition">
     <br/>
     @Temperature 500 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=8.44" title="Click to see this value in other UOMs">
     8.44
    </a>
    µin/in-°F
    <span class="dataCondition">
     <br/>
     @Temperature 932 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="altrow datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=5&amp;fromValue=16.7" title="Click to see this value in other UOMs">
     16.7
    </a>
    µm/m-°C
    <span class="dataCondition">
     <br/>
     @Temperature 950 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=4&amp;fromValue=9.28" title="Click to see this value in other UOMs">
     9.28
    </a>
    µin/in-°F
    <span class="dataCondition">
     <br/>
     @Temperature 1740 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Specific Heat Capacity
    <a href="/search/GraphConditionalData.aspx?matguid=d2a2119a08904a0fa706e9408cddb88e&amp;propid=695&amp;sigid=1" title="Graph this set of conditional data">
     <img alt="" src="/images/smallchart.gif"/>
    </a>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.128" title="Click to see this value in other UOMs">
     0.128
    </a>
    J/g-°C
    <span class="dataCondition">
     <br/>
     @Temperature 25.0 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0306" title="Click to see this value in other UOMs">
     0.0306
    </a>
    BTU/lb-°F
    <span class="dataCondition">
     <br/>
     @Temperature 77.0 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.133" title="Click to see this value in other UOMs">
     0.133
    </a>
    J/g-°C
    <span class="dataCondition">
     <br/>
     @Temperature 227 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0318" title="Click to see this value in other UOMs">
     0.0318
    </a>
    BTU/lb-°F
    <span class="dataCondition">
     <br/>
     @Temperature 441 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.142" title="Click to see this value in other UOMs">
     0.142
    </a>
    J/g-°C
    <span class="dataCondition">
     <br/>
     @Temperature 627 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0339" title="Click to see this value in other UOMs">
     0.0339
    </a>
    BTU/lb-°F
    <span class="dataCondition">
     <br/>
     @Temperature 1160 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.159" title="Click to see this value in other UOMs">
     0.159
    </a>
    J/g-°C
    <span class="dataCondition">
     <br/>
     @Temperature 1227 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0380" title="Click to see this value in other UOMs">
     0.0380
    </a>
    BTU/lb-°F
    <span class="dataCondition">
     <br/>
     @Temperature 2241 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.163" title="Click to see this value in other UOMs">
     0.163
    </a>
    J/g-°C
    <span class="dataCondition">
     <br/>
     @Temperature 1027 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0390" title="Click to see this value in other UOMs">
     0.0390
    </a>
    BTU/lb-°F
    <span class="dataCondition">
     <br/>
     @Temperature 1881 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.166" title="Click to see this value in other UOMs">
     0.166
    </a>
    J/g-°C
    <span class="dataCondition">
     <br/>
     @Temperature 1127 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0397" title="Click to see this value in other UOMs">
     0.0397
    </a>
    BTU/lb-°F
    <span class="dataCondition">
     <br/>
     @Temperature 2061 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=65&amp;fromValue=0.170" title="Click to see this value in other UOMs">
     0.170
    </a>
    J/g-°C
    <span class="dataCondition">
     <br/>
     @Temperature 1063 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=12&amp;fromValue=0.0406" title="Click to see this value in other UOMs">
     0.0406
    </a>
    BTU/lb-°F
    <span class="dataCondition">
     <br/>
     @Temperature 1945 °F
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
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=136&amp;fromValue=301" title="Click to see this value in other UOMs">
     301
    </a>
    W/m-K
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=10&amp;fromValue=2090" title="Click to see this value in other UOMs">
     2090
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
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=1064.43" title="Click to see this value in other UOMs">
     1064.43
    </a>
    °C
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=1947.97" title="Click to see this value in other UOMs">
     1947.97
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
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=2&amp;fromValue=2856" title="Click to see this value in other UOMs">
     2856
    </a>
    °C
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    <a class="unitlink" href="/tools/unitconverter.aspx?fromID=3&amp;fromValue=5173" title="Click to see this value in other UOMs">
     5173
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
    Optical Properties
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
    Emissivity (0-1)
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.040
    <span class="dataCondition">
     <br/>
     @Temperature 100 °C
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.040
    <span class="dataCondition">
     <br/>
     @Temperature 212 °F
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    total spectrum
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
    Reflection Coefficient, Visible (0-1)
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.27
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.27
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    400 nm
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.50
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.50
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    500 nm
   </td>
  </tr>
  <tr class="datarowSeparator">
   <td style="vertical-align:top;">
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.85
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataCell" style="vertical-align:top;">
    0.85
    <span class="dataCondition">
    </span>
   </td>
   <td class="dataComment" style="vertical-align:top;">
    600 nm
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
    Gold, Au
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
    CAS Number
   </td>
   <td class="dataCell" colspan="2">
    7440-57-5
   </td>
   <td class="dataComment" colspan="3">
   </td>
  </tr>
 </tbody>
</table>




```python
# Iterate through DataFrame
dfs = {} # Dictionary of DataFrames
for row in df.itertuples():
    df_tmp = pd.DataFrame()
    while():
```
