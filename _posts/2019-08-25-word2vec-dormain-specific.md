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

![](https://github.com/opokualbert/Domain_Specific_Word2Vec_Word_Embedding_Model_with_Gensim/raw/master/a_word_by.JPG)

Word2vec utilizes two model architectures, continuous bag-of-words (CBOW) or continuous skip-gram. For models that use continuous bag-of-words, a current word is predicted from a window of surrounding context words. In other words, you hide the target word and use the surrounding words to predict the hidden word. The order of surrounding words does not influence prediction. 

In the case of continuous skip-gram architecture, the current word in the model is used to predict the surrounding window of the context words. The skip-gram architecture gives more weight to nearby words than distant words. CBOW is faster than skip-gram but skip-gram performs better when it comes to infrequent words. 


Before Word2Vec became popular, one hot encoding which has sparsity problem, where most of the rows will be zeros for a very wide dimension. The dimension is the same as the number of words in the corpus. Each word will represent a column. This is very expensive for computation. Word2Vec solved this problem by representing each word as mostly 300 dimension space.


![](https://github.com/opokualbert/Domain_Specific_Word2Vec_Word_Embedding_Model_with_Gensim/raw/master/vectors.JPG)


After converting words to vector representations, math can be done on the vectors. You could find answers to analogy questions like:

Columbus is to Ohio as what is to Illinois?

word vector(Columbus) - word vector(Ohio) + word vector(Illinois) = Chicago

Or the popular one:

word vector(King) - word vector(Man) + word vector(woman) is Queen.


From the code example for our domain specific model, we are able to find the most similar words to foreclosure. If you are familiar with the mortgage industry, you will notice most of the words on the list are real estate related. This is not the best results because we trained the model on only 66k documents instead of over a million documents required. This is a good thing for our future comparison because we will use the same 66k documents in our next project to proof that when your data is small, fine-tuning an existing embedding is the way to go.


![](https://github.com/opokualbert/Domain_Specific_Word2Vec_Word_Embedding_Model_with_Gensim/raw/master/foreclosure.JPG)

If you need to read more about Word2vec and embeddings, this is a good article on [ kdnuggets](https://www.kdnuggets.com/2019/02/word-embeddings-nlp-applications.html) or this fantastic podcast by [Data Skeptic](http://dataskeptic.com/blog/episodes/2019/word2vec).


The complete code is saved on [Github](https://github.com/opokualbert/Domain_Specific_Word2Vec_Word_Embedding_Model_with_Gensim).


### We import all the needed python packages .

```
import re  # For preprocessing
import pandas as pd  # For data handling
from time import time  # To time our operations
from collections import defaultdict  # For word frequency
from nltk.tokenize import RegexpTokenizer
import multiprocessing

from gensim.models import Word2Vec

import logging  # Setting up the loggings to monitor gensim
logging.basicConfig(format="%(levelname)s - %(asctime)s: %(message)s", datefmt= '%H:%M:%S', level=logging.INFO)
pd.set_option('max_colwidth', 1000)
pd.options.display.max_rows = 500
```
Load the pickle file that contains the text documents. This file is a subset of the CFPB data from kaggle, which was used in some of my previous posts. I am also reseting the index, showing the numer of columns and taking a look at the top 5 documents. 

```
 df = pd.read_pickle('df_orig.pkl')
  df=df.reset_index(drop=True)
  df.shape
  df.head() 
```


The lines of code below are for cleaning the text and to tokenize the sentences into words.

```
df['clean'] = df['consumer_complaint_narrative'].str.replace('X', '')
df['clean'] = df['clean'].str.replace('\n', '')

df = df.dropna().drop_duplicates()
df_clean= df[['clean']]
df_clean.head()

sentences = df_clean.clean.astype('str').tolist()
sentences[0]

tokenizer = RegexpTokenizer(r'\w+')
sentences_tokenized = [w.lower() for w in sentences]
sentences_tokenized = [tokenizer.tokenize(i) for i in sentences_tokenized]
sentences_tokenized[0]
```

Set the parameters and build the model for 5 epochs. 

```
w2v_model = Word2Vec(min_count=10,
                     window=10,
                     size=300,
                     sample=6e-5, 
                     alpha=0.03, 
                     min_alpha=0.0007, 
                     negative=20,
                     workers=cores-1)
t = time()

w2v_model.build_vocab(sentences_tokenized, progress_per=10000)

print('Time to build vocab: {} mins'.format(round((time() - t) / 60, 2)))                     
t = time()

w2v_model.train(sentences_tokenized, total_examples=w2v_model.corpus_count, epochs=5, report_delay=1)

print('Time to train the model: {} mins'.format(round((time() - t) / 60, 2)))
```


I welcome feedback and discussion. I can be reached on Twitter [@opalbert](https://twitter.com/opalbert).
