# Scrape Proxy Table with Node.js
May 21, 2022

In this guide I will demonstrate how to scrape a table of free proxies using Node.js. First, in order to use Node.js in this Jupyter notebook, we need to run the following command in its own cell to install <b>pixiedust</b> and <b>pixiedust_node</b> with the package manager pip.


```python
!pip install pixiedust
!pip install pixiedust_node
```

    Requirement already satisfied: pixiedust in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (1.1.19)
    Requirement already satisfied: pandas in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust) (1.3.4)
    Requirement already satisfied: colour in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust) (0.1.5)
    Requirement already satisfied: geojson in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust) (2.5.0)
    Requirement already satisfied: requests in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust) (2.26.0)
    Requirement already satisfied: matplotlib in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust) (3.4.3)
    Requirement already satisfied: markdown in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust) (3.3.7)
    Requirement already satisfied: astunparse in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust) (1.6.3)
    Requirement already satisfied: six<2.0,>=1.6.1 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from astunparse->pixiedust) (1.16.0)
    Requirement already satisfied: wheel<1.0,>=0.23.0 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from astunparse->pixiedust) (0.37.0)
    Requirement already satisfied: importlib-metadata>=4.4 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from markdown->pixiedust) (4.8.1)
    Requirement already satisfied: pillow>=6.2.0 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust) (9.1.0)
    Requirement already satisfied: kiwisolver>=1.0.1 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust) (1.3.1)
    Requirement already satisfied: pyparsing>=2.2.1 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust) (3.0.4)
    Requirement already satisfied: numpy>=1.16 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust) (1.21.1)
    Requirement already satisfied: cycler>=0.10 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust) (0.10.0)
    Requirement already satisfied: python-dateutil>=2.7 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust) (2.8.2)
    Requirement already satisfied: pytz>=2017.3 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pandas->pixiedust) (2021.3)
    Requirement already satisfied: certifi>=2017.4.17 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from requests->pixiedust) (2020.12.5)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from requests->pixiedust) (1.26.7)
    Requirement already satisfied: idna<4,>=2.5 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from requests->pixiedust) (3.2)
    Requirement already satisfied: charset-normalizer~=2.0.0 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from requests->pixiedust) (2.0.4)
    Requirement already satisfied: zipp>=0.5 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from importlib-metadata>=4.4->markdown->pixiedust) (3.6.0)
    [33mWARNING: There was an error checking the latest version of pip.[0m[33m
    [0mRequirement already satisfied: pixiedust_node in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (0.2.5)
    Requirement already satisfied: pixiedust in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust_node) (1.1.19)
    Requirement already satisfied: pandas in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust_node) (1.3.4)
    Requirement already satisfied: ipython in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust_node) (7.29.0)
    Requirement already satisfied: traitlets>=4.2 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (5.1.0)
    Requirement already satisfied: pexpect>4.3 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (4.8.0)
    Requirement already satisfied: setuptools>=18.5 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (58.0.4)
    Requirement already satisfied: decorator in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (5.1.0)
    Requirement already satisfied: jedi>=0.16 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (0.18.0)
    Requirement already satisfied: pickleshare in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (0.7.5)
    Requirement already satisfied: pygments in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (2.10.0)
    Requirement already satisfied: backcall in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (0.2.0)
    Requirement already satisfied: appnope in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (0.1.2)
    Requirement already satisfied: prompt-toolkit!=3.0.0,!=3.0.1,<3.1.0,>=2.0.0 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (3.0.20)
    Requirement already satisfied: matplotlib-inline in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from ipython->pixiedust_node) (0.1.2)
    Requirement already satisfied: python-dateutil>=2.7.3 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pandas->pixiedust_node) (2.8.2)
    Requirement already satisfied: pytz>=2017.3 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pandas->pixiedust_node) (2021.3)
    Requirement already satisfied: numpy>=1.17.3 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pandas->pixiedust_node) (1.21.1)
    Requirement already satisfied: astunparse in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust->pixiedust_node) (1.6.3)
    Requirement already satisfied: requests in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust->pixiedust_node) (2.26.0)
    Requirement already satisfied: geojson in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust->pixiedust_node) (2.5.0)
    Requirement already satisfied: markdown in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust->pixiedust_node) (3.3.7)
    Requirement already satisfied: matplotlib in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust->pixiedust_node) (3.4.3)
    Requirement already satisfied: colour in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pixiedust->pixiedust_node) (0.1.5)
    Requirement already satisfied: parso<0.9.0,>=0.8.0 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from jedi>=0.16->ipython->pixiedust_node) (0.8.2)
    Requirement already satisfied: ptyprocess>=0.5 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from pexpect>4.3->ipython->pixiedust_node) (0.7.0)
    Requirement already satisfied: wcwidth in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from prompt-toolkit!=3.0.0,!=3.0.1,<3.1.0,>=2.0.0->ipython->pixiedust_node) (0.2.5)
    Requirement already satisfied: six>=1.5 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from python-dateutil>=2.7.3->pandas->pixiedust_node) (1.16.0)
    Requirement already satisfied: wheel<1.0,>=0.23.0 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from astunparse->pixiedust->pixiedust_node) (0.37.0)
    Requirement already satisfied: importlib-metadata>=4.4 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from markdown->pixiedust->pixiedust_node) (4.8.1)
    Requirement already satisfied: kiwisolver>=1.0.1 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust->pixiedust_node) (1.3.1)
    Requirement already satisfied: pyparsing>=2.2.1 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust->pixiedust_node) (3.0.4)
    Requirement already satisfied: cycler>=0.10 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust->pixiedust_node) (0.10.0)
    Requirement already satisfied: pillow>=6.2.0 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from matplotlib->pixiedust->pixiedust_node) (9.1.0)
    Requirement already satisfied: certifi>=2017.4.17 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from requests->pixiedust->pixiedust_node) (2020.12.5)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from requests->pixiedust->pixiedust_node) (1.26.7)
    Requirement already satisfied: charset-normalizer~=2.0.0 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from requests->pixiedust->pixiedust_node) (2.0.4)
    Requirement already satisfied: idna<4,>=2.5 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from requests->pixiedust->pixiedust_node) (3.2)
    Requirement already satisfied: zipp>=0.5 in /Users/rakeshbhatia/opt/anaconda3/lib/python3.9/site-packages (from importlib-metadata>=4.4->markdown->pixiedust->pixiedust_node) (3.6.0)
    [33mWARNING: There was an error checking the latest version of pip.[0m[33m
    [0m

Next, let's start up pixiedust_node by importing it in a new cell.


```python
import pixiedust_node
npm.install(("table-scraper"))
```

    Pixiedust database opened successfully




<div style="margin:10px">
    <a href="https://github.com/ibm-watson-data-lab/pixiedust" target="_new">
        <img src="https://github.com/ibm-watson-data-lab/pixiedust/raw/master/docs/_static/pd_icon32.png" style="float:left;margin-right:10px"/>
    </a>
    <span>Pixiedust version 1.1.19</span>
</div>





<div style="margin:10px"> 
<a href="https://github.com/ibm-cds-labs/pixiedust_node" target="_new"> 
<img src="https://github.com/ibm-cds-labs/pixiedust_node/raw/master/docs/_images/pdn_icon32.png" style="float:left;margin-right:10px"/> 
</a> 
<span>Pixiedust Node.js</span> 
</div> 



    /usr/local/bin/npm install -s table-scraper
    pixiedust_node 0.2.5 started. Cells starting '%%node' may contain Node.js code.



```python
import pixiedust_node
```

Now we can install our required modules.


```python
npm.install(("table-scraper"))
```

    /usr/local/bin/npm install -s table-scraper


The `table-scraper` module enables us to extract all tables from the HTML of a webpage. The tables are stored as an array of JSON objects. From here we can extract the individual proxy IP addresses and port numbers to use for scraping.


```python
%%node
var scraper = require("table-scraper");
var data;
scraper.get("https://www.proxynova.com/proxy-server-list/elite-proxies/").then(function(tableData) {
        data = tableData[0];
        console.log(data);
    });
```

    ... ... ...
    [
    {
    '0': 'document.write("164" + ".15" + "5.1" + "50." + "31");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '695 ms',
    Uptime: '50%\n                \n                 (413)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("104" + ".16" + "0.1" + "89." + "3");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '572 ms',
    Uptime: '90%\n                \n                 (486)',
    'Proxy Country': 'United States\n' +
    '\n' +
    '                                             - Los Angeles',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("164" + ".15" + "5.1" + "45." + "0");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '1038 ms',
    Uptime: '58%\n                \n                 (86)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("164" + ".15" + "5.1" + "47." + "31");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '639 ms',
    Uptime: '49%\n                \n                 (521)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("164" + ".15" + "5.1" + "51." + "1");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '485 ms',
    Uptime: '48%\n                \n                 (633)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("47." + "74." + "226" + ".8");',
    'Proxy Port': '5001',
    'Last Check': '',
    'Proxy Speed': '847 ms',
    Uptime: '7%\n                \n                 (494)',
    'Proxy Country': 'Singapore\n\n                                             - Singapore',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("202" + ".16" + "2.1" + "94." + "70");',
    'Proxy Port': '41766',
    'Last Check': '',
    'Proxy Speed': '3380 ms',
    Uptime: '3%\n                \n                 (127)',
    'Proxy Country': 'Indonesia\n\n                                             - Medan',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("120" + ".19" + "4.1" + "50." + "70");',
    'Proxy Port': '9091',
    'Last Check': '',
    'Proxy Speed': '1104 ms',
    Uptime: '100%\n                \n                 (2)',
    'Proxy Country': 'China\n\n                                             - Nanyang',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("43." + "255" + ".11" + "3.2" + "32");',
    'Proxy Port': '8084',
    'Last Check': '',
    'Proxy Speed': '3128 ms',
    Uptime: '21%\n                \n                 (213)',
    'Proxy Country': 'Cambodia\n\n                                             - Phnom Penh',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("216" + ".13" + "7.1" + "84." + "253");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '4037 ms',
    Uptime: '28%\n                \n                 (175)',
    'Proxy Country': 'United States',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("66." + "94." + "120" + ".16" + "1");',
    'Proxy Port': '443',
    'Last Check': '',
    'Proxy Speed': '4578 ms',
    Uptime: '1%\n                \n                 (105)',
    'Proxy Country': 'United States\n\n                                             - Seattle',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("80." + "48." + "119" + ".28");',
    'Proxy Port': '8080',
    'Last Check': '',
    'Proxy Speed': '822 ms',
    Uptime: '99%\n                \n                 (183)',
    'Proxy Country': 'Poland\n\n                                             - Makow Mazowiecki',
    Anonymity: 'Elite'
    },
    { '0': '(adsbygoogle = window.adsbygoogle || []).push({});' },
    {
    '0': 'document.write("8.1" + "42." + "142" + ".25" + "0");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '2816 ms',
    Uptime: '25%\n                \n                 (499)',
    'Proxy Country': 'China\n\n                                             - Beijing',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("182" + ".61" + ".20" + "1.2" + "01");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '1159 ms',
    Uptime: '59%\n                \n                 (172)',
    'Proxy Country': 'China',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("61." + "79." + "139" + ".30");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '713 ms',
    Uptime: '16%\n                \n                 (139)',
    'Proxy Country': 'South Korea\n\n                                             - Seoul',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("213" + ".23" + "0.9" + "7.1" + "0");',
    'Proxy Port': '3128',
    'Last Check': '',
    'Proxy Speed': '3372 ms',
    Uptime: '37%\n                \n                 (195)',
    'Proxy Country': 'Uzbekistan\n\n                                             - Tashkent',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("59." + "11." + "52." + "237");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '723 ms',
    Uptime: '16%\n                \n                 (171)',
    'Proxy Country': 'South Korea\n\n                                             - Hwaseong-si',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("121" + ".19" + "9.7" + "8.2" + "28");',
    'Proxy Port': '8888',
    'Last Check': '',
    'Proxy Speed': '1050 ms',
    Uptime: '45%\n                \n                 (154)',
    'Proxy Country': 'China\n\n                                             - Hangzhou',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("39." + "175" + ".75" + ".24");',
    'Proxy Port': '30001',
    'Last Check': '',
    'Proxy Speed': '1368 ms',
    Uptime: '44%\n                \n                 (139)',
    'Proxy Country': 'China',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("47." + "104" + ".23" + "7.3" + "5");',
    'Proxy Port': '81',
    'Last Check': '',
    'Proxy Speed': '1168 ms',
    Uptime: '12%\n                \n                 (94)',
    'Proxy Country': 'China\n\n                                             - Qingdao',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("112" + ".6." + "117" + ".13" + "5");',
    'Proxy Port': '8085',
    'Last Check': '',
    'Proxy Speed': '1323 ms',
    Uptime: '94%\n                \n                 (269)',
    'Proxy Country': 'China\n\n                                             - Qingdao',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("67." + "212" + ".18" + "6.1" + "01");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '2237 ms',
    Uptime: '20%\n                \n                 (516)',
    'Proxy Country': 'United States',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("58." + "20." + "184" + ".18" + "7");',
    'Proxy Port': '9091',
    'Last Check': '',
    'Proxy Speed': '2283 ms',
    Uptime: '67%\n                \n                 (152)',
    'Proxy Country': 'China\n\n                                             - Jingzhou',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("195" + ".15" + "8.1" + "8.2" + "36");',
    'Proxy Port': '3128',
    'Last Check': '',
    'Proxy Speed': '827 ms',
    Uptime: '13%\n                \n                 (45)',
    'Proxy Country': 'Uzbekistan\n\n                                             - Andijan',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("43." + "255" + ".11" + "3.2" + "32");',
    'Proxy Port': '8081',
    'Last Check': '',
    'Proxy Speed': '3051 ms',
    Uptime: '19%\n                \n                 (282)',
    'Proxy Country': 'Cambodia\n\n                                             - Phnom Penh',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("54." + "175" + ".19" + "7.2" + "35");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '1743 ms',
    Uptime: '11%\n                \n                 (134)',
    'Proxy Country': 'United States\n\n                                             - Ashburn',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("183" + ".24" + "7.2" + "11." + "50");',
    'Proxy Port': '30001',
    'Last Check': '',
    'Proxy Speed': '1457 ms',
    Uptime: '56%\n                \n                 (125)',
    'Proxy Country': 'China',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("151" + ".10" + "6.1" + "8.1" + "24");',
    'Proxy Port': '1080',
    'Last Check': '',
    'Proxy Speed': '3608 ms',
    Uptime: '16%\n                \n                 (156)',
    'Proxy Country': 'France\n\n                                             - Strasbourg',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("222" + ".65" + ".22" + "8.9" + "6");',
    'Proxy Port': '8085',
    'Last Check': '',
    'Proxy Speed': '2392 ms',
    Uptime: '50%\n                \n                 (122)',
    'Proxy Country': 'China\n\n                                             - Shanghai',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("106" + ".14" + ".25" + "5.1" + "24");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '1185 ms',
    Uptime: '72%\n                \n                 (166)',
    'Proxy Country': 'China\n\n                                             - Shanghai',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("169" + ".57" + ".1." + "85");',
    'Proxy Port': '8123',
    'Last Check': '',
    'Proxy Speed': '1119 ms',
    Uptime: '91%\n                \n                 (220)',
    'Proxy Country': 'Mexico\n\n                                             - Mexico City',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("106" + ".15" + "8.1" + "56." + "213");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '651 ms',
    Uptime: '9%\n                \n                 (141)',
    'Proxy Country': 'Japan\n\n                                             - Toride',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("64." + "227" + ".62" + ".12" + "3");',
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '155 ms',
    Uptime: '33%\n                \n                 (357)',
    'Proxy Country': 'United States\n' +
    '\n' +
    '                                             - Santa Clara',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("20." + "47." + "108" + ".20" + "4");',
    'Proxy Port': '8888',
    'Last Check': '',
    'Proxy Speed': '161 ms',
    Uptime: '25%\n                \n                 (208)',
    'Proxy Country': 'United States\n\n                                             - Washington',
    Anonymity: 'Elite'
    },
    {
    '0': 'document.write("77." + "50." + "104" + ".11" + "0");',
    'Proxy Port': '3128',
    'Last Check': '',
    'Proxy Speed': '2936 ms',
    Uptime: '5%\n                \n                 (813)',
    'Proxy Country': 'Russia\n\n                                             - Moscow',
    Anonymity: 'Elite'
    }
    ]


We will clean up the IP address with regular expressions, as it has been rendered as a JavaScript operation string. Then we will remove the 13th element from the JSON array, since it points to an ad that is embedded in the table.


```python
%%node
for (var i = 0; i < data.length; i++) {
    ip = data[i]['0'].replace(/[^0-9\.]+/g,"");
    console.log(ip);
    ip = ip.slice(1);
    data[i]['0'] = ip;
    //console.log(ip);
}
var removed = data.splice(12,1);
data = JSON.stringify(data);
console.log(data);
```

    ... ... ... ... ... ... 164.155.150.31
    104.160.189.3
    164.155.145.0
    164.155.147.31
    164.155.151.1
    47.74.226.8
    202.162.194.70
    120.194.150.70
    43.255.113.232
    216.137.184.253
    66.94.120.161
    80.48.119.28
    8.142.142.250
    182.61.201.201
    61.79.139.30
    213.230.97.10
    59.11.52.237
    121.199.78.228
    39.175.75.24
    47.104.237.35
    112.6.117.135
    67.212.186.101
    58.20.184.187
    195.158.18.236
    43.255.113.232
    54.175.197.235
    183.247.211.50
    151.106.18.124
    222.65.228.96
    106.14.255.124
    169.57.1.85
    106.158.156.213
    64.227.62.123
    20.47.108.204
    77.50.104.110
    [{"0":"64.155.150.31","Proxy Port":"80","Last Check":"","Proxy Speed":"695 ms","Uptime":"50%\n                \n                 (413)","Proxy Country":"United States\n\n                                             - Chicago","Anonymity":"Elite"},{"0":"04.160.189.3","Proxy Port":"80","Last Check":"","Proxy Speed":"572 ms","Uptime":"90%\n                \n                 (486)","Proxy Country":"United States\n\n                                             - Los Angeles","Anonymity":"Elite"},{"0":"64.155.145.0","Proxy Port":"80","Last Check":"","Proxy Speed":"1038 ms","Uptime":"58%\n                \n                 (86)","Proxy Country":"United States\n\n                                             - Chicago","Anonymity":"Elite"},{"0":"64.155.147.31","Proxy Port":"80","Last Check":"","Proxy Speed":"639 ms","Uptime":"49%\n                \n                 (521)","Proxy Country":"United States\n\n                                             - Chicago","Anonymity":"Elite"},{"0":"64.155.151.1","Proxy Port":"80","Last Check":"","Proxy Speed":"485 ms","Uptime":"48%\n                \n                 (633)","Proxy Country":"United States\n\n                                             - Chicago","Anonymity":"Elite"},{"0":"7.74.226.8","Proxy Port":"5001","Last Check":"","Proxy Speed":"847 ms","Uptime":"7%\n                \n                 (494)","Proxy Country":"Singapore\n\n                                             - Singapore","Anonymity":"Elite"},{"0":"02.162.194.70","Proxy Port":"41766","Last Check":"","Proxy Speed":"3380 ms","Uptime":"3%\n                \n                 (127)","Proxy Country":"Indonesia\n\n                                             - Medan","Anonymity":"Elite"},{"0":"20.194.150.70","Proxy Port":"9091","Last Check":"","Proxy Speed":"1104 ms","Uptime":"100%\n                \n                 (2)","Proxy Country":"China\n\n                                             - Nanyang","Anonymity":"Elite"},{"0":"3.255.113.232","Proxy Port":"8084","Last Check":"","Proxy Speed":"3128 ms","Uptime":"21%\n                \n                 (213)","Proxy Country":"Cambodia\n\n                                             - Phnom Penh","Anonymity":"Elite"},{"0":"16.137.184.253","Proxy Port":"80","Last Check":"","Proxy Speed":"4037 ms","Uptime":"28%\n                \n                 (175)","Proxy Country":"United States","Anonymity":"Elite"},{"0":"6.94.120.161","Proxy Port":"443","Last Check":"","Proxy Speed":"4578 ms","Uptime":"1%\n                \n                 (105)","Proxy Country":"United States\n\n                                             - Seattle","Anonymity":"Elite"},{"0":"0.48.119.28","Proxy Port":"8080","Last Check":"","Proxy Speed":"822 ms","Uptime":"99%\n                \n                 (183)","Proxy Country":"Poland\n\n                                             - Makow Mazowiecki","Anonymity":"Elite"},{"0":"82.61.201.201","Proxy Port":"80","Last Check":"","Proxy Speed":"1159 ms","Uptime":"59%\n                \n                 (172)","Proxy Country":"China","Anonymity":"Elite"},{"0":"1.79.139.30","Proxy Port":"80","Last Check":"","Proxy Speed":"713 ms","Uptime":"16%\n                \n                 (139)","Proxy Country":"South Korea\n\n                                             - Seoul","Anonymity":"Elite"},{"0":"13.230.97.10","Proxy Port":"3128","Last Check":"","Proxy Speed":"3372 ms","Uptime":"37%\n                \n                 (195)","Proxy Country":"Uzbekistan\n\n                                             - Tashkent","Anonymity":"Elite"},{"0":"9.11.52.237","Proxy Port":"80","Last Check":"","Proxy Speed":"723 ms","Uptime":"16%\n                \n                 (171)","Proxy Country":"South Korea\n\n                                             - Hwaseong-si","Anonymity":"Elite"},{"0":"21.199.78.228","Proxy Port":"8888","Last Check":"","Proxy Speed":"1050 ms","Uptime":"45%\n                \n                 (154)","Proxy Country":"China\n\n                                             - Hangzhou","Anonymity":"Elite"},{"0":"9.175.75.24","Proxy Port":"30001","Last Check":"","Proxy Speed":"1368 ms","Uptime":"44%\n                \n                 (139)","Proxy Country":"China","Anonymity":"Elite"},{"0":"7.104.237.35","Proxy Port":"81","Last Check":"","Proxy Speed":"1168 ms","Uptime":"12%\n                \n                 (94)","Proxy Country":"China\n\n                                             - Qingdao","Anonymity":"Elite"},{"0":"12.6.117.135","Proxy Port":"8085","Last Check":"","Proxy Speed":"1323 ms","Uptime":"94%\n                \n                 (269)","Proxy Country":"China\n\n                                             - Qingdao","Anonymity":"Elite"},{"0":"7.212.186.101","Proxy Port":"80","Last Check":"","Proxy Speed":"2237 ms","Uptime":"20%\n                \n                 (516)","Proxy Country":"United States","Anonymity":"Elite"},{"0":"8.20.184.187","Proxy Port":"9091","Last Check":"","Proxy Speed":"2283 ms","Uptime":"67%\n                \n                 (152)","Proxy Country":"China\n\n                                             - Jingzhou","Anonymity":"Elite"},{"0":"95.158.18.236","Proxy Port":"3128","Last Check":"","Proxy Speed":"827 ms","Uptime":"13%\n                \n                 (45)","Proxy Country":"Uzbekistan\n\n                                             - Andijan","Anonymity":"Elite"},{"0":"3.255.113.232","Proxy Port":"8081","Last Check":"","Proxy Speed":"3051 ms","Uptime":"19%\n                \n                 (282)","Proxy Country":"Cambodia\n\n                                             - Phnom Penh","Anonymity":"Elite"},{"0":"4.175.197.235","Proxy Port":"80","Last Check":"","Proxy Speed":"1743 ms","Uptime":"11%\n                \n                 (134)","Proxy Country":"United States\n\n                                             - Ashburn","Anonymity":"Elite"},{"0":"83.247.211.50","Proxy Port":"30001","Last Check":"","Proxy Speed":"1457 ms","Uptime":"56%\n                \n                 (125)","Proxy Country":"China","Anonymity":"Elite"},{"0":"51.106.18.124","Proxy Port":"1080","Last Check":"","Proxy Speed":"3608 ms","Uptime":"16%\n                \n                 (156)","Proxy Country":"France\n\n                                             - Strasbourg","Anonymity":"Elite"},{"0":"22.65.228.96","Proxy Port":"8085","Last Check":"","Proxy Speed":"2392 ms","Uptime":"50%\n                \n                 (122)","Proxy Country":"China\n\n                                             - Shanghai","Anonymity":"Elite"},{"0":"06.14.255.124","Proxy Port":"80","Last Check":"","Proxy Speed":"1185 ms","Uptime":"72%\n                \n                 (166)","Proxy Country":"China\n\n                                             - Shanghai","Anonymity":"Elite"},{"0":"69.57.1.85","Proxy Port":"8123","Last Check":"","Proxy Speed":"1119 ms","Uptime":"91%\n                \n                 (220)","Proxy Country":"Mexico\n\n                                             - Mexico City","Anonymity":"Elite"},{"0":"06.158.156.213","Proxy Port":"80","Last Check":"","Proxy Speed":"651 ms","Uptime":"9%\n                \n                 (141)","Proxy Country":"Japan\n\n                                             - Toride","Anonymity":"Elite"},{"0":"4.227.62.123","Proxy Port":"80","Last Check":"","Proxy Speed":"155 ms","Uptime":"33%\n                \n                 (357)","Proxy Country":"United States\n\n                                             - Santa Clara","Anonymity":"Elite"},{"0":"0.47.108.204","Proxy Port":"8888","Last Check":"","Proxy Speed":"161 ms","Uptime":"25%\n                \n                 (208)","Proxy Country":"United States\n\n                                             - Washington","Anonymity":"Elite"},{"0":"7.50.104.110","Proxy Port":"3128","Last Check":"","Proxy Speed":"2936 ms","Uptime":"5%\n                \n                 (813)","Proxy Country":"Russia\n\n                                             - Moscow","Anonymity":"Elite"}]
    



```python
%%node
var proxies = [];
function addProxies(obj) {
    proxies.push(obj["0"] + ":" + obj["Proxy Port"]);
}
data.forEach(obj => addProxies(obj));
console.log(proxies);
```

    ... ...
    [
    '64.155.150.31:80',    '04.160.189.3:80',
    '64.155.145.0:80',     '64.155.147.31:80',
    '64.155.151.1:80',     '7.74.226.8:5001',
    '02.162.194.70:41766', '20.194.150.70:9091',
    '3.255.113.232:8084',  '16.137.184.253:80',
    '6.94.120.161:443',    '0.48.119.28:8080',
    '82.61.201.201:80',    '1.79.139.30:80',
    '13.230.97.10:3128',   '9.11.52.237:80',
    '21.199.78.228:8888',  '9.175.75.24:30001',
    '7.104.237.35:81',     '12.6.117.135:8085',
    '7.212.186.101:80',    '8.20.184.187:9091',
    '95.158.18.236:3128',  '3.255.113.232:8081',
    '4.175.197.235:80',    '83.247.211.50:30001',
    '51.106.18.124:1080',  '22.65.228.96:8085',
    '06.14.255.124:80',    '69.57.1.85:8123',
    '06.158.156.213:80',   '4.227.62.123:80',
    '0.47.108.204:8888',   '7.50.104.110:3128'
    ]



```python
%%node
function renameKey(obj, oldKey, newKey) {
    console.log("Renaming key");
    obj[newKey] = obj[oldKey];
    delete obj[oldKey];
}

data.forEach(obj => renameKey(obj, "0", "Proxy IP"));
console.log(data);
//const newData = JSON.stringify(data);
//console.log(newData);
```

    ... ... ... ...
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    Renaming key
    [
    {
    'Proxy Port': '8000',
    'Last Check': '',
    'Proxy Speed': '3891 ms',
    Uptime: '4%\n                \n                 (68)',
    'Proxy Country': 'India\n\n                                             - Mumbai',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '1948 ms',
    Uptime: '56%\n                \n                 (513)',
    'Proxy Country': 'United States',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '2344 ms',
    Uptime: '41%\n                \n                 (365)',
    'Proxy Country': 'Japan\n\n                                             - Tokyo',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '8080',
    'Last Check': '',
    'Proxy Speed': '598 ms',
    Uptime: '29%\n                \n                 (342)',
    'Proxy Country': 'Russia\n\n                                             - Korolyov',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '1883 ms',
    Uptime: '40%\n                \n                 (277)',
    'Proxy Country': 'Kazakhstan\n\n                                             - Oral',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '8080',
    'Last Check': '',
    'Proxy Speed': '3727 ms',
    Uptime: '3%\n                \n                 (365)',
    'Proxy Country': 'Russia\n\n                                             - Moscow',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '535 ms',
    Uptime: '43%\n                \n                 (544)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '2678 ms',
    Uptime: '37%\n                \n                 (464)',
    'Proxy Country': 'Japan\n\n                                             - Osaka',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '8080',
    'Last Check': '',
    'Proxy Speed': '924 ms',
    Uptime: '19%\n                \n                 (496)',
    'Proxy Country': 'India\n\n                                             - Mumbai',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '3128',
    'Last Check': '',
    'Proxy Speed': '2084 ms',
    Uptime: '28%\n                \n                 (662)',
    'Proxy Country': 'Uzbekistan\n\n                                             - Tashkent',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '2780 ms',
    Uptime: '19%\n                \n                 (436)',
    'Proxy Country': 'United States',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '443',
    'Last Check': '',
    'Proxy Speed': '3221 ms',
    Uptime: '3%\n                \n                 (159)',
    'Proxy Country': 'Singapore\n\n                                             - Singapore',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    { 'Proxy IP': undefined },
    {
    'Proxy Port': '9812',
    'Last Check': '',
    'Proxy Speed': '3628 ms',
    Uptime: '10%\n                \n                 (79)',
    'Proxy Country': 'India\n\n                                             - Patan',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '2351 ms',
    Uptime: '20%\n                \n                 (461)',
    'Proxy Country': 'United States',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '1583',
    'Last Check': '',
    'Proxy Speed': '3378 ms',
    Uptime: '25%\n                \n                 (163)',
    'Proxy Country': 'Vietnam',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '459 ms',
    Uptime: '97%\n                \n                 (409)',
    'Proxy Country': 'United States\n' +
    '\n' +
    '                                             - Los Angeles',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '5566',
    'Last Check': '',
    'Proxy Speed': '4667 ms',
    Uptime: '5%\n                \n                 (442)',
    'Proxy Country': 'Canada\n\n                                             - Montreal',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '606 ms',
    Uptime: '44%\n                \n                 (373)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '631 ms',
    Uptime: '47%\n                \n                 (467)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '51696',
    'Last Check': '',
    'Proxy Speed': '3289 ms',
    Uptime: '4%\n                \n                 (163)',
    'Proxy Country': 'United States\n\n                                             - Evanston',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '53281',
    'Last Check': '',
    'Proxy Speed': '3496 ms',
    Uptime: '6%\n                \n                 (198)',
    'Proxy Country': 'Russia\n\n                                             - Lyubertsy',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '9812',
    'Last Check': '',
    'Proxy Speed': '3468 ms',
    Uptime: '3%\n                \n                 (335)',
    'Proxy Country': 'South Africa\n\n                                             - Cape Town',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '624 ms',
    Uptime: '54%\n                \n                 (373)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '1317 ms',
    Uptime: '72%\n                \n                 (280)',
    'Proxy Country': 'United States',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '3585 ms',
    Uptime: '17%\n                \n                 (508)',
    'Proxy Country': 'France',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '974 ms',
    Uptime: '48%\n                \n                 (124)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '585 ms',
    Uptime: '48%\n                \n                 (439)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '25',
    'Last Check': '',
    'Proxy Speed': '3869 ms',
    Uptime: '6%\n                \n                 (171)',
    'Proxy Country': 'Cambodia\n\n                                             - Phnom Penh',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '724 ms',
    Uptime: '50%\n                \n                 (390)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '2931 ms',
    Uptime: '39%\n                \n                 (616)',
    'Proxy Country': 'Japan\n\n                                             - Tokyo',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '688 ms',
    Uptime: '52%\n                \n                 (460)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '698 ms',
    Uptime: '48%\n                \n                 (443)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '8000',
    'Last Check': '',
    'Proxy Speed': '2002 ms',
    Uptime: '4%\n                \n                 (149)',
    'Proxy Country': 'Hong Kong\n\n                                             - Central',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '80',
    'Last Check': '',
    'Proxy Speed': '720 ms',
    Uptime: '46%\n                \n                 (459)',
    'Proxy Country': 'United States\n\n                                             - Chicago',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    },
    {
    'Proxy Port': '20000',
    'Last Check': '',
    'Proxy Speed': '923 ms',
    Uptime: '59%\n                \n                 (409)',
    'Proxy Country': 'Singapore\n' +
    '\n' +
    '                                             - Ang Mo Kio New Town',
    Anonymity: 'Elite',
    'Proxy IP': undefined
    }
    ]



```python

```
