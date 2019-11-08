---
published: true
title: "Training a domain specific Word2Vec word embedding model with Gensim,
improve your text search and classification results"
date: 2019-08-25
categories:
  - data science
  - nlp
---
![](https://images.unsplash.com/photo-1551524559-8af4e6624178?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=500&q=60)

In this post, I will show how to train your own domain specific Word2Vec model using your own data. There are powerful, off the shelf embedding models built by the likes of Google (Word2Vec), Facebook (FastText) and Stanford (Glove) because they have the resources to do it and as a result of years research. These models that were trained on huge corpus generally perform well but there are times they fail on specific tasks in industries like health, finance and legal. There are two approaches to solve this problem. First, train your own embeddings if you have enough data of over a million text documents and the compute power. Two, fine-tune one of the listed models above with your data, especially, when your data is small (I will post a follow up blog to show how to fine-tune word2vec models). 

Word2Vec assumes two words that have the same context will also share the same meaning and therefore, both words will have similar vector representation. The vector of a word is a semantic representation of how that word is used in context. Being able to represent words as dense vectors is the core of the successes registered in the application of deep learning to NLP in recent times.

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
