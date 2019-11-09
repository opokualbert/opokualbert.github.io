---
published: true
title: "Creating a Searchable Database with Text Extracted from Scanned Pdfs or Images
Pdf Text OCR - Searchable Pdf text in a Database"
date: 2019-08-05
categories:
  - data science
  - nlp
---
![](https://github.com/opokualbert/Creating-a-searchable-database-with-text-extracted-from-scanned-Pdfs-or-images/blob/master/jen-theodore-hbkWMj41Y0I-unsplash.jpg?raw=true)

In this short tutorial I show how to extract text from images and scanned pdfs and store the results in a database to make the document searchable. 

Pdf documents and images with text are difficult to work with. Most business people manually read through multiple pages to retrieve the information they are looking for. We want to use a python program that will take a pdf, whether scanned or not as well as any image that contains text and extract the text by page and index each page in a dataframe which can be stored in any database of your choice and be made available for users to write nlp search or mine the text on the table.

<!--more-->

The example we will use is a pdf document with a mini course on Weka by machine learning mastery. The pdf has 23 pages. We will use python packages wand, pillow and pytesseract to convert it to image and then extract each page text , all in one program.


For the package pytesseract to work, download and install tesseract-ocr from this link [tesseract-ocr](https://github.com/UB-Mannheim/tesseract/wiki).


The complete code is saved on [Github](https://github.com/opokualbert/Creating-a-searchable-database-with-text-extracted-from-scanned-Pdfs-or-images/blob/master/Pdf%20Text%20OCR%20-%20Searchable%20Pdf%20text%20in%20a%20Database.ipynb). I want to give credit to Ratul Doley for his work on youtube.


We import packages and set options.

* pip install pillow
* pip install pytesseract
* First install tesseract-ocr from this link https://github.com/UB-Mannheim/tesseract/wiki
* Then add this line for tesseract ocr to work

```
pytesseract.pytesseract.tesseract_cmd = r"C:\Program Files (x86)\Tesseract-OCR\tesseract.exe"
import pandas as pd
from PIL import Image
import pytesseract
import io
from wand.image import Image as wand
pd.set_option('max_colwidth', 2000)
pd.options.display.max_rows = 500
```

These few lines of code import the pdf and convert the file to images and then extract the text in the images. The resolution you set has an effect on the performance of the code. Higher resolution is better but makes the code slow.



```
pdf = wand(filename = "machine_learning_mastery_with_weka_mini_course.pdf", resolution = 300)
pdfImage = pdf.convert('jpeg')

Blobs = []

for g in pdfImage.sequence:
    page = wand(image = g)
    Blobs.append(page.make_blob('jpeg'))

extract = []

for b in Blobs:
    m = Image.open(io.BytesIO(b))
    text = pytesseract.image_to_string(m, lang = 'eng')
    extract.append(text)

print('Done extracting text')
```

We write a for loop to extract the words, their entities (level) as determined by spacy and the description of each of the entities and put them into a list called table. 

```
table = []
for ent in doc.ents:
    table.append([ent.text,ent.label_,spacy.explain(ent.label_)])
```

The code below will take the results from the above program and and convert it to a pandas dataframe and assign a category just incase you are adding more documents. You can also add a time stamp. Each row on the table represents a page from the pdf document and the index is the page number. 


```
df = pd.DataFrame(extract,columns =['Page_Text'])\
.replace(r'\n',' ', regex=True)
df['Category'] = 'WEKA'
df
```

This dataframe can be saved in a database for business users to query. I will write another post to show some nlp examples we can do on this table.





I welcome feedback and discussion. I can be reached on Twitter [@opalbert](https://twitter.com/opalbert).
