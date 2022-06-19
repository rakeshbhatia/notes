# Scraping Stock Market News and Updates with lxml


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
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
```


```python
#XPath: //*[@id="HP-MarketsModule"]/section[2]/section[2]/section/div[1]

url = 'https://www.cnbc.com/'

page = requests.get(url)

doc = lh.fromstring(page.content)

#print(doc.text_content())

market_news = doc.xpath('//*[@id="HP-MarketsModule"]/section[2]/section[2]/section//div')

print(market_news[0].text_content())
```

    The last time the S&P did this, it rallied 25% to an all-time highNia Warfield6 hours ago



```python
keywords = ['Trump', 'China', 'tariff', 'Tariff', 'Mnuchin', 'Treasury', 'commerce', 'Commerce']
k = set(keywords)

headlines = []

for headline in market_news:
    if headline.text_content() not in headlines:
        headlines.append(headline.text_content())
    # Check if keyword in headline
    #matching = k.intersection(headline.text_content())
    #if matching:
    #    headlines.append(headline.text_content())

headlines = headlines[1:]
#headlines = headlines.remove(headlines[1])
print('headlines: \n', headlines)
```

    headlines: 
     ['The last time the S&P did this, it rallied 25% to an all-time high', 'Nia Warfield6 hours ago', 'The market week ahead: Mexico tariffs and more data that may clear the way for a rate cut', 'Dow jumps 260 points, posts best week since November after jobs report spurs rate-cut hopes', "That big May jobs letdown may just be part of the 'new normal' for the labor market"]



```python
#import smtplib
#from email.mime.text import MIMEText
#from email.mime.multipart import MIMEMultipart

email = "rakeshbhatia87@gmail.com"
pas = ""

sms_gateway = '5106480300@txt.att.net'
# The server we use to send emails in our case it will be gmail but every email provider has a different smtp 
# and port is also provided by the email provider.
smtp = "smtp.gmail.com"
port = 587
# This will start our email server
server = smtplib.SMTP(smtp,port)
# Starting the server
server.starttls()
# Now we need to login
server.login(email,pas)

# Now we use the MIME module to structure our message.
msg = MIMEMultipart()
msg['From'] = email
msg['To'] = sms_gateway
# Make sure you add a new line in the subject
msg['Subject'] = "Testing sms\n"
# Make sure you also add new lines to your body
body = "Headlines: \n" + '\n'.join(headlines)
print(body)
# and then attach that body furthermore you can also send html content.
msg.attach(MIMEText(body, 'plain'))

sms = msg.as_string()

server.sendmail(email,sms_gateway,sms)

# lastly quit the server
server.quit()
```

    Headlines: 
    The last time the S&P did this, it rallied 25% to an all-time high
    Nia Warfield6 hours ago
    The market week ahead: Mexico tariffs and more data that may clear the way for a rate cut
    Dow jumps 260 points, posts best week since November after jobs report spurs rate-cut hopes
    That big May jobs letdown may just be part of the 'new normal' for the labor market





    (221, b'2.0.0 closing connection p188sm3519277oia.14 - gsmtp')




```python

```
