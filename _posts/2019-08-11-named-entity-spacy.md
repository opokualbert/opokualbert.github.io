---
published: true
title: "Named Entity Recognition With Spacy Python Package
Automated Information Extraction from Text - Natural Language Processing"
date: 2019-08-11
categories:
  - data science
  - nlp
---
![](https://images.unsplash.com/photo-1444910069701-01b71164c92d?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=60)

In my [previous](https://opokualbert.com/data%20science/nlp/stanford-ner/) post, I showed how you can get the entities in an article or text documents using natural language processing NER Package by Stanford NLP. 

In this post I will share how to do this in a few lines of code in Spacy and compare the results from the two packages. Named entity recognition is using natural language processing to pull out all entities like a person, organization, money, geo location, time and date from an article or documents.

<!--more-->

Spacy and Stanford NLP python packages both use part of speech tagging to identify which entity a word in the article should be assigned to. As I indicated in the previous post this is useful if you quickly need to gather the specific salient information about a very long document, example who contacted who at what time and at what place; and which organization do they work for or are they discussing? Was money involved in the dealings and how much? 
It turns out spacy is fast and also requires a few lines of code. In addition, spacy is more accurate in identifying entities compared to Stanford NLP NER at least for this particular dataset. Spacy also gives additional methods to describe or explain what the labels represent. And if you so desire, you can also visualize the entities in the text document.


![](https://github.com/opokualbert/Named_Entity_Recognition_With_Spacy/raw/master/Spacy_screensot.JPG)

For comparison purposes, I will use the same text I used in the earlier post. The text is from this article from techrunch. [African fintech dominates Catalyst Fund’s 2019 startup cohort](https://techcrunch.com/2019/06/21/african-fintech-dominates-catalyst-funds-2019-startup-cohort). 

The complete code is saved on [Github](https://github.com/opokualbert/Named_Entity_Recognition_With_Spacy). 


We import Spacy and other packages and set options.

```
import spacy
import pandas as pd
pd.set_option('max_colwidth', 2000)
pd.options.display.max_rows = 500
```


These few lines of code loads the spacy large english core language model, loads and parse the text file we are trying to analyse and assign to spacy object named doc. We also take a look at the content of the document loaded.


```
nlp = spacy.load('en_core_web_lg')
doc = nlp(open('African_Fintech.txt', encoding="utf8").read())
doc
```

We write a for loop to extract the words, their entities (level) as determined by spacy and the description of each of the entities and put them into a list called table. 

```
table = []
for ent in doc.ents:
    table.append([ent.text,ent.label_,spacy.explain(ent.label_)])
```

Let us convert the list into a pandas dataframe and take a look at the information we extracted. We also filter on the Lable column to be able to compare the results with the Stanford Nlp results. Please pull the Stanford Nlp results from the previous post and compare. 


```
df2 = pd.DataFrame(table, columns=['Entity', 'Label','Label_Description']).sort_values(by=['Label'])
print(df2)
df2.loc[df2['Label'].isin(['PERSON','ORG','PERCENT','MONEY','LOCATION','GPE','DATE'])]
```

Display the enties and their labels in the actual document.

```
spacy.displacy.render(doc, style='ent',jupyter=True)
```



I welcome feedback and discussion. I can be reached on Twitter [@opalbert](https://twitter.com/opalbert).
