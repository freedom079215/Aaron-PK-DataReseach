---
layout: post
title: Sentiment Analysis for PTT
date: 2019-09-20
excerpt: ""
tags: NLP, Sentiment, Deeplearning
comments: true
---
# Sentiment Analysis

- 情緒分析範例

> Ref: http://puremonkey2010.blogspot.com/2017/09/toolkit-keras-imdb.html


### 1. Download IMDB Datasets
```python=
#!/usr/bin/env python3  
import os, urllib, logging  
import tarfile  
from urllib.request import urlretrieve  
  
###################  
# Step0: Global setting  
####################  
LOG_FORMAT = '%(asctime)s - %(name)s - %(levelname)s - %(message)s'  
logging.basicConfig(format=LOG_FORMAT)  
logger = logging.getLogger('IMDBb')  
logger.setLevel(logging.DEBUG)  
  
###################  
# Step1: Download IMDB  
####################  
url = "http://ai.stanford.edu/~amaas/data/sentiment/aclImdb_v1.tar.gz"  
filepath = '../data/aclImdb_v1.tar.gz'  
dataPath = '../data/aclImdb'  
if not os.path.isfile('../data/aclImdb_v1.tar.gz'):  
    print('Downloading from {}...'.format(url))  
    result = urlretrieve(url, '../data/aclImdb_v1.tar.gz')  
    print('download: {}'.format(result))  
    
if not os.path.isdir('../data/aclImdb'):  
    print('Extracting {} to datas...'.format(filepath))  
    tfile = tarfile.open('../data/aclImdb_v1.tar.gz', 'r:gz')  
    result = tfile.extractall('../data') 
```


### 2. Read IMDB Datasets
```python=
###################  
# Step2: Download IMDB  
####################  
from keras.preprocessing import sequence  
from keras.preprocessing.text import Tokenizer  
import re  
  
def rm_tags(text):  
    r'''  
    Remove HTML markers  
    '''  
    re_tag = re.compile(r'<[^>]+>')  
    return re_tag.sub('', text)  
  
def read_files(filetype):  
    r'''  
    Read data from IMDb folders  
  
    @param filetype(str):  
        "train" or "test"  
  
    @return:  
        Tuple(List of labels, List of articles)  
    '''  
    file_list = []  
    positive_path = os.path.join(os.path.join(dataPath, filetype), 'pos')  
    for f in os.listdir(positive_path):  
        file_list.append(os.path.join(positive_path, f))  
  
    negative_path = os.path.join(os.path.join(dataPath, filetype), 'neg')  
    for f in os.listdir(negative_path):  
        file_list.append(os.path.join(negative_path, f))  
  
    logger.debug('Read {} with {} files...'.format(filetype, len(file_list)))  
    all_labels = ([1] * 12500 + [0] * 12500)  
    all_texts = []  
    for fi in file_list:  
        logger.debug('Read {}...'.format(fi))  
        with open(fi, encoding='utf8') as fh:  
            all_texts += [rm_tags(" ".join(fh.readlines()))]  
  
    return all_labels, all_texts  
  
train_labels, train_text = read_files('train')  
test_labels, test_text = read_files('test')  
```

### 3. Review IMDB Datasets
```python=
train_texts[0] # 查看第 0 筆 "影評文字"
train_labels[0] # 第 0 筆的評價 label 為 1, 也就是 正面評價
train_texts[12501]
train_labels[12501] # 第 12,501 筆的評價 label 為 0, 也就是 負面評價
```

### 4. Create Token

```python=
###################  
# Step3: Tokenize  
####################  
MAX_LEN_OF_TOKEN = 100  
  
logger.info('Tokenizing document...')  
token = Tokenizer(num_words = 2000)  
''' Create a dictionary of 2,000 words '''  
token.fit_on_texts(train_text)  
''' Read in all training text and select top 2,000 words according to frequency sorting descendingly '''  
  
logger.info('Total {} document being handled...'.format(token.document_count))  
logger.info('Top 10 word index:')  
c = 0  
for t,i in token.word_index.items():  
    print("\t'{}'\t{}".format(t, i))  
    c += 1  
    if c == 10:  
        break  
print("")  
logger.info('Translating raw text into token number list...')  
x_train_seq = token.texts_to_sequences(train_text)  
x_test_seq = token.texts_to_sequences(test_text)  
  
logger.info('Padding/Trimming the token number list to same length={}...'.format(MAX_LEN_OF_TOKEN))  
x_train = sequence.pad_sequences(x_train_seq, maxlen=MAX_LEN_OF_TOKEN)  
x_test = sequence.pad_sequences(x_test_seq, maxlen=MAX_LEN_OF_TOKEN)  
```

### 5.Build the MLP Model

> 底下代碼依序進行: 
> 1. 建立 Embedding 層: 將 "數字 list" 轉為 "向量 list". (這邊使用 32 維度來表式數字 1-2000) 
> 2. 建立多層感知器 (MLP): 
> * 平坦層: 共有 3,200 = 32 * 100 個神經元, 共有 100 個 Token 數字, 每一個 Token 使用 32 維度來表示數字 1-2000.
> * 隱藏層: 共有 256 個神經元
> * 輸出層: 只有一個神經元. 1 表示正面評價; 0 表示負面評價.
> 

```python=
###################  
# Step4: Building MODEL  
####################  
from keras.models import Sequential  
from keras.layers.core import Dense, Dropout, Activation, Flatten  
from keras.layers.embeddings import Embedding  
  
MODEL_TYPE = 'mlp'  
  
if MODEL_TYPE == 'mlp':  
    model = Sequential()  
    model.add(Embedding(output_dim=32,  
                        input_dim=2000,  
                        input_length=100))  
    model.add(Dropout(0.2))  
    '''Drop 20% neuron during training '''  
    model.add(Flatten())  
    model.add(Dense(units=256, activation='relu'))  
    ''' Total 256 neuron in hidden layers'''  
    model.add(Dropout(0.35))  
    model.add(Dense(units=1, activation='sigmoid'))  
    ''' Define output layer with 'sigmoid activation' '''  
  
logger.info('Model summary:\n{}\n'.format(model.summary()))  
```

### 6. Trainning Model with Back Propagation model
```python=
###################  
# Step5: Training  
###################  
logger.info('Start training process...')  
model.compile(loss='binary_crossentropy',  
              optimizer='adam',  
              metrics=['accuracy'])  
train_history = model.fit(x_train, train_labels, batch_size=100, epochs=10, verbose=2, validation_split=0.2)  
```

### 7. Evaluate Model Performance
```python=
###################  
# Step6: Evaluation  
###################  
logger.info('Start evaluation...')  
scores = model.evaluate(x_test, test_labels, verbose=1)  
print("")  
logger.info('Score={}'.format(scores[1]))  
```

### 8. Prediction
```python=
predict = model.predict_classes(x_test) #進行預測
predict[:10] # 顯示前 10 筆預測結果

predict_classes = predict.reshape(-1) # 將預測結果轉為一維陣列
predict_classes[:10]
```
- Predict Specific Data
```python=
predict_classes = model.predict_classes(x_test).reshape(-1)  
print("")  
sentiDict = {1:'Pos', 0:'Neg'}  
def display_test_Sentiment(i):  
    r'''  
    Show prediction on i'th test data  
    '''  
    logger.debug('{}\'th test data:\n{}\n'.format(i, test_text[i]))  
    logger.info('Ground truth: {}; prediction result: {}'.format(sentiDict[test_labels[i]], sentiDict[predict_classes[i]]))  
  
logger.info('Show prediction on 2\'th test data:')  
display_test_Sentiment(2)  
```

### 9. Improve with larger Dictionary
> 建立字典的字數: 原本是 2,000 個字的字典, 這邊會增加到 3,800 個字的字典.
>數字 list 截長補短的長度: 原本 "數字 list" 長度都是 100 個數字, 這邊改為 380 個數字.
>

```python=
...  
###################  
# Step3: Tokenize  
####################  
MAX_LEN_OF_TOKEN = 380  
DICT_NUM_WORDS = 3800  
  
logger.info('Tokenizing document...')  
token = Tokenizer(num_words = DICT_NUM_WORDS)  
''' Create a dictionary of 2,000 words '''  
token.fit_on_texts(train_text)  
''' Read in all training text and select top 2,000 words according to frequency sorting descendingly '''  
  
logger.info('Total {} document being handled...'.format(token.document_count))  
logger.info('Top 10 word index:')  
c = 0  
for t,i in token.word_index.items():  
    print("\t'{}'\t{}".format(t, i))  
    c += 1  
    if c == 10:  
        break  
print("")  
logger.info('Translating raw text into token number list...')  
x_train_seq = token.texts_to_sequences(train_text)  
x_test_seq = token.texts_to_sequences(test_text)  
  
logger.info('Padding/Trimming the token number list to length={}...'.format(MAX_LEN_OF_TOKEN))  
x_train = sequence.pad_sequences(x_train_seq, maxlen=MAX_LEN_OF_TOKEN)  
x_test = sequence.pad_sequences(x_test_seq, maxlen=MAX_LEN_OF_TOKEN)  
...  
```
