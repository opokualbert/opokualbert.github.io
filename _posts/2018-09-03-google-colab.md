---
published: true
title: "Downloading Kaggle Datasets into Google Colab:
Easy Access to Kaggle Datasets in Colab"
date: 2018-09-03
categories:
  - data science
  - tools
---
![](https://miro.medium.com/max/543/1*Lad06lrjlU9UZgSTHUoyfA.png)

In this tutorial, I show how to download kaggle datasets into google colab. Kaggle has been and remains the de factor platform to try your hands on data science projects. The platform has huge rich free datasets for machine learning projects.


Another product from google, the company behind kaggle is colab, a platform suitable for training machine learning models and deep neural network free of charge without any installation requirement. One key thing that makes colab a game changer, especially for people who do not own GPU laptop is that users have the option to train their models with free GPU. Colab does not have the trove of datasets kaggle host on its platform therefore, it will be nice if you could access the datasets on kaggle from colab. There is in fact a kaggle API which we can use in colab but setting it up to work is not so easy. I would want to show how to use the API in a few simple steps. 

<!--more-->

### Step 1

Create a kaggle account if you do not have one already. Click on your user name, click on account.

![](https://miro.medium.com/max/560/1*XWyvY6DETwE1G7us2Acvgw.png)

Scroll down to click on create new API token. This will download a file unto your PC. Note the location of the downloaded file.

![](https://miro.medium.com/max/897/1*jRvlte2MylmEnriavqYnMA.png)

### Step 2

Go to colab via this link: Colab and under file, click on new python 3 notebook. In the first cell, type this code to install kaggle API and make a directory called kaggle.
 
```
!pip install -U -q kaggle
!mkdir -p ~/.kaggle
```

### Step 3

Type this code into the next cell and run to import the API key into colab.


```
from google.colab import files
files.upload()
```

In the next cell, run this code to copy the API key to the kaggle directory we created.


```
!cp kaggle.json ~/.kaggle/
```

The datasets should be available for us to use. Let us list the datasets with this code.

```
!kaggle datasets list
```

### Step 4

We can download files now by using this sample code. In this case the US consumer finance complaints was downloaded.

```
!kaggle datasets download -d cfpb/us-consumer-finance-complaints
!ls
```

### Step 5

We use pandas to read the data we have downloaded by unzipping the file first. This line of code works in most situations.

```
import pandas as pd
data2 = pd.read_csv('/content/us-consumer-finance-complaints.zip', compression='zip', header=0, sep=',', quotechar='"')
```

It did not work here because the zipped file also contains a sqlite database. I will use a different method below to extract only the CSV.

```
from zipfile import ZipFile
zip_file = ZipFile('/content/us-consumer-finance-complaints.zip')
fields= ['product','consumer_complaint_narrative'] 
data=pd.read_csv(zip_file.open('consumer_complaints.csv'), usecols=fields)
data.head()
```

You can get the entire code at [GitHub](https://github.com/opokualbert/Downloading-Kaggle-Datasets-into-Google-Colab).



I welcome feedback and discussion. I can be reached on Twitter [@opalbert](https://twitter.com/opalbert).
