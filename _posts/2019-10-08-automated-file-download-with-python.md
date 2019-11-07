---
published: true
title: "Automated File Downloader With Python, Chromedriver and Selenium"
date: 2019-10-08
categories:
  - data science
  - Selenium
  - Scraping
---
![](https://images.unsplash.com/photo-1531492898132-a3dfbc4dbac1?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=500&q=60)

Downloading a single file from a website is simply easy and does not require a tutorial whatsoever. Where it becomes a daunting task is when you need to download multiple files of say hundreds or thousands. For most employees, a big percentage of their daily tasks involve downloading files and then the files are stitched together for a report or analysis.

In this short tutorial, I will show how you can download one year stock price data from yahoo finance for multiple companies using their ticker symbols. Python, chromedriver and selenium package were used to automate this process. You could use the same code to download any other file type.

<!--more-->

If you want to try your hands on the process copy the code from my [GitHub](https://github.com/opokualbert/Python-Automated-File-Downloader). Credit to Artur Spirin, this post was inspired by his tutorial.

You need to download chromdriver from here [chromdriver](https://chromedriver.chromium.org/downloads) and save it in your directory Change the tickers list to as many stock tickers you want to download the stock price for and that is it.

```
from selenium import webdriver
from selenium.webdriver.common.by import By

options = webdriver.ChromeOptions()

#Change this to your default downloading folder r"C:\Users\alber\selenium\download_files"
preferences = {"download.default_directory": r"C:\Users\alber\selenium\download_files", "safebrowsing.enabled": "false"}

options.add_experimental_option("prefs", preferences)

driver = webdriver.Chrome(options=options)

tickers= ['AAN','AMP','BAC','C','CAT','CINF','CMI','DFS','HUM',
'MCK','TECD','TSN','URI','PYPL','AYX']

for ticker in tickers:
    driver.get("https://finance.yahoo.com/quote/" +ticker+ "/history?p="+ticker)

    driver.find_element(By.XPATH, '//*[@id="Col1-1-HistoricalDataTable-Proxy"]/section/div[1]/div[2]/span[2]/a').click() 
```
In case duplicate files are downloaded, use the code below to remove the duplicate files from the folder. 

```
import os

file_path = r"C:\Users\alber\selenium\download_files"
file_list = os.listdir(file_path)
# print (file_list)
for file_name in file_list:
    if " (1)" not in file_name:
        continue
    original_file_name = file_name.replace(' (1)', '')
    if not os.path.exists(os.path.join(file_path, original_file_name)):
        continue  # do not remove files which have no original
    os.remove(os.path.join(file_path, file_name))
```

I welcome feedback and discussion. I can be reached on Twitter [@opalbert](https://twitter.com/opalbert).
