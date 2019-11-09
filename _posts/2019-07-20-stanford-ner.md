---
published: true
title: "Named Entity Recognition With Stanford NLP NER Package:
Automated Information Extraction from Text - Natural Language Processing"
date: 2019-07-20
categories:
  - data science
  - nlp
---
![](https://github.com/opokualbert/Named_Entity_Recognition_With_Stanford_NLP/blob/master/cristian-newman-ZjYIfZ8wf2I-unsplash.jpg?raw=true)

In this short post, I will show how to get the entities in an article or any text documents using Natural language processing technique. We will use the powerful NER Package by Stanford NLP in this tutorial.
 

An entity could be a person, organization, location, date/time, money, percentage, the list goes on. This is useful if you quickly need to gather the specific salient information about a very long document, example who contacted who at what time and at what place; and which organization do they work for or are they discussing? Was money involved in the dealings and how much? 

<!--more-->

If your job involve answering these kinds of questions using text documents, then this tutorial may improve your productivity. 


We are using the text from this article from techrunch. [African fintech dominates Catalyst Fund’s 2019 startup cohort](https://techcrunch.com/2019/06/21/african-fintech-dominates-catalyst-funds-2019-startup-cohort/)

Make sure you have java.exe in this location. Sometimes, it is not required, depending on your machine setup

```
import os
java_path = "C:/Program Files/Java/jre1.8.0_211/bin/java.exe"
os.environ['JAVAHOME'] = java_path
```

We import packages and create a function to capture the entities, credit to Omar Bahareth.


```
from nltk.tag.stanford import StanfordNERTagger
from nltk.tokenize import word_tokenize

def formatted_entities(classified_paragraphs_list):
    entities = {'persons': list(), 'organizations': list(), 'locations': list(), 'dates': list(), 'money': list(), 'percent': list()}

    for classified_paragraph in classified_paragraphs_list:
        for entry in classified_paragraph:
            entry_value = entry[0]
            entry_type = entry[1]

            if entry_type == 'PERSON':
                entities['persons'].append(entry_value)

            elif entry_type == 'ORGANIZATION':
                entities['organizations'].append(entry_value)

            elif entry_type == 'LOCATION':
                entities['locations'].append(entry_value)
            elif entry_type == 'DATE':
                entities['dates'].append(entry_value)
            elif entry_type == 'MONEY':
                entities['money'].append(entry_value)
            elif entry_type == 'PERCENT':
                entities['percent'].append(entry_value)
    return entities
```
Download stanford-ner-2018-10-16 and unzip to get english.muc.7class.distsim.crf.ser.gz and stanford-ner.jar files. Save them in the appropraite folder from stanfor nlp site [Stanford parser](https://nlp.stanford.edu/software/stanford-parser-full-2018-10-17.zip) and run the code below.


```
tagger = StanfordNERTagger('/Users/Shared/stanford-ner/classifiers/english.muc.7class.distsim.crf.ser.gz',
               '/Users/Shared/stanford-ner/stanford-ner.jar',encoding='utf-8')
```

Import spacy to extract the data from the text file

```
import spacy,en_core_web_sm
import pandas as pd

nlp = en_core_web_sm.load()
doc = nlp(open('African_Fintech.txt', encoding="utf8").read())
```

We create a dataframe that has the tokenized sentences in a column.


```
d = []
for idno, sentence in enumerate(doc.sents):
    d.append({"id": idno, "sentence":str(sentence)})

df = pd.DataFrame(d)
df.set_index('id', inplace=True)

print('There are {}'.format(len(d)) ,'Sentences in this article')
df.head()
```

Convert the sentences to a list and create the result: a dictionary of all the entities you specified above. 


```
df1=df.sentence.tolist()
tokenized_paragraphs = list()

for text in df1:
    tokenized_paragraphs.append(word_tokenize(text))

classified_paragraphs_list = tagger.tag_sents(tokenized_paragraphs)


formatted_result = formatted_entities(classified_paragraphs_list)
print(formatted_result)
```


![](https://github.com/opokualbert/Named_Entity_Recognition_With_Stanford_NLP/blob/master/Capture.JPG?raw=true)


I welcome feedback and discussion. I can be reached on Twitter [@opalbert](https://twitter.com/opalbert).
