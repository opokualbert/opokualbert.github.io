---
published: true
title: "Classification of Customer Complaints using Tensorflow, Transfer Learning:
Text Classification with Word Embeddings"
date: 2018-07-20
categories:
  - data science
  - nlp
  - Machine Learning
  - Transfer Learning
---
![](https://miro.medium.com/max/378/0*kNshKdwHupVaUcEL)

In this post, I show how to classify consumer complaints text into these categories: Debt collection, Consumer Loan, Mortgage, Credit card, Credit reporting, Student loan, Bank account or service, Payday loan, Money transfers, Other financial service, Prepaid card.


This kind of model will be very useful for a customer service department that wants to classify the complaints they receive from their customers. The classification of the issues they have received into buckets will help the department to provide customized solutions to the customers in each group. 

<!--more-->

This model can also be expanded into a system, that can recommend automatic solutions to future complaints as they come in. In the past, performing these kinds of tasks were done manually by multiple employees and of course, take a long time to accomplish, delaying swift response to the complaints received.
 


Machine learning and AI are here to solve this caliber of problems. Imagine you can classify new complaints with 95% accuracy and route them to the right team to resolve the issue. That will be a win and time saving to any business. Your customers will be happy because the right expert from your business will talk to your customers in trying to resolving their complaints. This will translate into lowering churning rate which means more revenue.

I trained a text classifier with 66,806 of data on customers that have made a complaint to consumer financial protection bureau - CFPB about US financial institutions on the services they have rendered to these consumers. The dataset is on kaggle.com at this link .


I used the universal-sentence-encoder-large/3 module on the new tensorflowhub platform to leverage the power of transfer learning which according to Wikipedia, is a research problem in machine learning that focuses on storing knowledge gained while solving one problem and applying it to a different but related problem. Google and other teams have made available on tensorflowhub, models that took them about 62,000 GPU hours to train for our free use.

This dataset is relatively not large but this kind of machine learning process requires more compute power so I chose to use Google’s colab, which gives the option to train a model with free GPU. I have a previous blog post on downloading Kaggle datasets into Google Colab on my [website](https://opokualbert.com/), you may want to check it out if you are interested in downloading this dataset to follow along with this demo.


I will walk through the steps and in the end, we will classify new complaints and see how the model performed.

We import the needed packages for this work.
```
import os
import numpy as np
import pandas as pd
import tensorflow as tf
import tensorflow_hub as hub
import json
import pickle
import urllib

from sklearn.preprocessing import LabelBinarizer

print(tf.__version__)
```

### Data Preparation and Cleaning

Take a look at the data with pandas.


```
data = data[pd.notnull(data['consumer_complaint_narrative'])]
data.head()
```
We convert the consumer_complaint_narrative column to lower case, perfect for NLP. Also remove characters that do not have predictive power and can misinform the model.


```
pd.set_option('max_colwidth', 1000)
data['consumer_complaint_narrative'] = data['consumer_complaint_narrative'].str.lower()
data.head()
import re
data['consumer_complaint_narrative'] = data['consumer_complaint_narrative'].str.replace('x', '')
data['consumer_complaint_narrative'] = data['consumer_complaint_narrative'].str.replace('{', '')
data['consumer_complaint_narrative'] = data['consumer_complaint_narrative'].str.replace('}', '')
data['consumer_complaint_narrative'] = data['consumer_complaint_narrative'].str.replace('/', '')
data.head() 
```

We assign the the complaint text and the product type to variables to be able to preprocess. We also set aside 0.0001 for testing after we train the model. The remainder of the data is split into train and validation sets.

```
data_comp=data[['consumer_complaint_narrative']]
data_prod=data[['product']]

train_size = int(len(data_comp) * .999)
train_descriptions = data_comp[:train_size].astype('str')
train_prod = data_prod[:train_size]
test_descriptions = data_comp[train_size:].astype('str')
test_prod =data_prod[train_size:]

train_size = int(len(train_descriptions) * .8)
train_desc = train_descriptions[:train_size]
train_pr = train_prod[:train_size]
val_desc = train_descriptions[train_size:]
val_pr =train_prod[train_size:]
```

We will use scikit learn to encode the labels into one-hot vector. We also print encoder.classes_ to show the list of all the classes the model will be predicting.



```
from sklearn import preprocessing
encoder = preprocessing.LabelBinarizer()
encoder.fit_transform(train_pr)
train_encoded = encoder.transform(train_pr)
val_encoded = encoder.transform(val_pr)
num_classes = len(encoder.classes_)

# Print all possible products and the label for the first complaint in our training dataset
print(encoder.classes_)
print(train_encoded[0])
```

### Model training using TensorFlow DNNEstimator

We will download the tfhub pre-trained text embeddings universal-sentence-encoder-large/3 to encode the complaint text into high dimensional text vectors. Be aware that this can take a few minutes to download.

```
description_embeddings = hub.text_embedding_column("descriptions", module_spec="https://tfhub.dev/google/universal-sentence-encoder-large/3", trainable=False)
```

We are going to use a DNNEstimator to train a deep neural net.The model will have 2 hidden layers of 64 neurons in the first layer and 10 neurons in the second layer. We also use a batch_size of 100 and 10 epochs. These are parameters you can tune to find the perfect combination. We also pass the inputs, in this case, the features and label as well as the description_embeddings.

Batch_size represent the number of examples that will be passed to our model during one iteration, and num_epochs is the number of times our model will go through the entire training set.

```
multi_label_head  = tf.contrib.estimator.multi_label_head(
    num_classes,
    loss_reduction=tf.losses.Reduction.SUM_OVER_BATCH_SIZE
)
features = {
  "descriptions": np.array(train_desc).astype(np.str)
}
labels = np.array(train_encoded).astype(np.int32)
train_input_fn = tf.estimator.inputs.numpy_input_fn(features, labels, shuffle=True, batch_size=100, num_epochs=10)
estimator = tf.contrib.estimator.DNNEstimator(
    head=multi_label_head,
    hidden_units=[64,10],
    feature_columns=[description_embeddings])    
 ```

 We are ready to train the model. We will time it to know how long it took to finish training.

```
 %%timeit
estimator.train(input_fn=train_input_fn)

#This is optional to run to get the training accuracy so that we can compare with the validation accuracy to check for overfitting
%%timeit
train_input_fn_1 = tf.estimator.inputs.numpy_input_fn({"descriptions": np.array(train_desc).astype(np.str)}, train_encoded.astype(np.int32), shuffle=False)
estimator.evaluate(input_fn=train_input_fn_1)
```

It is time to evaluate our model with the evaluation dataset we set aside. The results is pretty good at 95%. You can tune the hyperparameters to acheive a better results.

```
eval_input_fn = tf.estimator.inputs.numpy_input_fn({"descriptions": np.array(val_desc).astype(np.str)}, val_encoded.astype(np.int32), shuffle=False)
estimator.evaluate(input_fn=eval_input_fn)
```

Finally, let us test our model and see how it will do on classifying new data. We will test the model with 67 records. It got a few incorrect. You will see that those the model could not predict correctly also had low confidence values.

```
predict_input_fn = tf.estimator.inputs.numpy_input_fn({"descriptions": np.array(test_descriptions).astype(np.str)}, shuffle=False)
results = estimator.predict(predict_input_fn)
for product in results:
  top = product['probabilities'].argsort()[-1:]
  for prod in top:
    text_prod = encoder.classes_[prod]
    print(text_prod + ': ' + str(round(product['probabilities'][prod] * 100, 2)) + '%')
  print('')
```

The complete code is saved on [Github](https://github.com/opokualbert/Classification-of-customer-complaints-using-tensorflow---Text-Classification-with-Word-Embeddings).




I welcome feedback and discussion. I can be reached on Twitter [@opalbert](https://twitter.com/opalbert).
