---
published: true
title: "Implementing ROE and EV_EBITDA in Python to Discover Bargain Stocks"
date: 2020-01-05
categories:
  - Data Science
  - Finance
  - Automation
  - Selenium
  - Scraping
---
![](https://images.unsplash.com/photo-1559589689-577aabd1db4f?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=500&q=60)

Implementing Eddy Elfenbein's Two Articles on Crossing Wall Street Website. In these articles Eddy show why [return-on-equity (ROE)](http://www.crossingwallstreet.com/archives/2012/09/why-return-on-equity-is-so-important.html) is so important and why [EV/EBITDA](http://www.crossingwallstreet.com/archives/2014/02/the-single-best-metric-evebitda.html) is the single most important metric.

I used selenium and chromedriver to scrape data from yahoo finance to implement these concepts shared in the articles. To try to run this, you just need to add one or more stock tickers in the fuction.


<!--more-->

If you want to try your hands on the process copy the code from my [GitHub](https://github.com/opokualbert/Implementing-ROE-and-EV_EBITDA-to-Discover-Bargain-Stocks).

You need to download chromdriver from here [chromdriver](https://chromedriver.chromium.org/downloads) and save it in your directory Change the tickers list to as many stock tickers you want to download the stock price for and that is it.

```
def ticks(symbol):
    import pandas as pd
    tickers = list(symbol.split(" ")) 
    from selenium import webdriver
    from selenium.webdriver.common.by import By
    from selenium.webdriver.support.ui import WebDriverWait
    from selenium.webdriver.support import expected_conditions as EC
    from selenium.common.exceptions import TimeoutException
    l=[]
    browser = webdriver.Chrome()
    browser.set_page_load_timeout(120)
    browser_buy = webdriver.Chrome()
    browser_buy.set_page_load_timeout(120)
    y=[]
    for ticker in tickers:
        site= 'https://finance.yahoo.com/quote/' +ticker+ '/analysis/'
        browser.get(site)
        site_buy= 'https://finance.yahoo.com/quote/' +ticker+ '/key-statistics/'
        browser_buy.get(site_buy)
        # get all the financial values
        e={}
        e['name']=ticker
        try:
            e['price'] = browser_buy.find_element_by_xpath('//*[@id="quote-header-info"]/div[3]/div[1]/div/span[1]').text
        except:
            pass
        try:
            e['eps_y1'] = browser.find_element_by_xpath('//*[@id="Col1-0-AnalystLeafPage-Proxy"]/section/table[1]/tbody/tr[2]/td[5]/span').text
        except:
            pass
        try:
            e['eps_growth_5y'] = browser.find_element_by_xpath('//*[@id="Col1-0-AnalystLeafPage-Proxy"]/section/table[6]/tbody/tr[5]/td[2]').text
        except:
            pass
        try:
            e['ev_ebitda'] = browser_buy.find_element_by_xpath('//*[@id="Col1-0-KeyStatistics-Proxy"]/section/div[2]/div[1]/div/div/div[1]/table/tbody/tr[9]/td[2]').text
        except:
            pass
        try:
            e['roe'] = browser_buy.find_element_by_xpath('//*[@id="Col1-0-KeyStatistics-Proxy"]/section/div[2]/div[3]/div/div[3]/div/table/tbody/tr[2]/td[2]').text
        except:
            pass
        try:
            e['pe'] = browser_buy.find_element_by_xpath('//*[@id="Col1-0-KeyStatistics-Proxy"]/section/div[2]/div[1]/div/div/div[1]/table/tbody/tr[3]/td[2]').text
        except:
            pass
        try:
            e['peg'] = browser_buy.find_element_by_xpath('//*[@id="Col1-0-KeyStatistics-Proxy"]/section/div[2]/div[1]/div/div/div[1]/table/tbody/tr[5]/td[2]').text
        except:
            pass
        try:
            e['ps'] = browser_buy.find_element_by_xpath('//*[@id="Col1-0-KeyStatistics-Proxy"]/section/div[2]/div[1]/div/div/div[1]/table/tbody/tr[6]/td[2]').text
        except:
            pass
        y.append(e)
    #save to dataframe
    import pandas as pd
    buy_yahoo=pd.DataFrame(y)
    buy_yahoo[buy_yahoo.columns.drop('name')] = buy_yahoo[buy_yahoo.columns.drop('name')].astype(str)
    buy_yahoo=buy_yahoo.loc[buy_yahoo['roe'] !='N/A']
    buy_yahoo=buy_yahoo.loc[buy_yahoo['eps_growth_5y'] !='nan']
    buy_yahoo['roe']=(buy_yahoo['roe'].str[:-1].astype(float))
    buy_yahoo=buy_yahoo.loc[buy_yahoo['eps_growth_5y'] !='N/A']
    buy_yahoo['eps_growth_5y']=(buy_yahoo['eps_growth_5y'].str[:-1].astype(float))
    buy_yahoo[buy_yahoo.columns.drop('name')] = buy_yahoo[buy_yahoo.columns.drop('name')].apply(pd.to_numeric, errors='coerce')
    buy_yahoo['fair_pe']=(buy_yahoo.eps_growth_5y/2)+8
    buy_yahoo['fair_price']=buy_yahoo.fair_pe*buy_yahoo.eps_y1
    buy_yahoo['discount']=((buy_yahoo.fair_price - buy_yahoo.price)/buy_yahoo.fair_price)
    return buy_yahoo
    browser.quit()
    browser_buy.quit()
ticks('AAPL MESA UAL aal dal BHF MLHR') 
```
![discount stocks](https://github.com/opokualbert/Implementing-ROE-and-EV_EBITDA-to-Discover-Bargain-Stocks/blob/master/discount%20stocks.JPG?raw=true)


The information in this blog is for learning purpose and does not contain a recommendation for any particular security or investment. The information provided is obtained from sources which are believed to be reliable. However, I have not independently verified or otherwise investigated all such information. I do not guarantee the accuracy or completeness of any such information.


I welcome feedback and discussion. I can be reached on Twitter [@opalbert](https://twitter.com/opalbert).
