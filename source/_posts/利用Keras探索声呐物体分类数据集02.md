---
title: 利用Keras探索声呐物体分类数据集02
copyright: true
date: 2018-05-24 12:57:19
tags:
    - 机器学习
    - Keras
categories:
    - 机器学习
    - Keras
---

## 目录

- [1. 加载数据](#1.-加载数据)
- [2. 创建模型](#2.-创建模型)
- [3. 使用混淆矩阵来进一步验证](#3.-使用混淆矩阵来进一步验证)

# 1. 加载数据


```python
import pandas as pd
import numpy as np
```


```python
dataset = pd.read_csv('../data/sonar.csv')
```


```python
dataset.head()
```



# 2. 创建模型


```python
X = dataset.iloc[:,0:60]
y = dataset.iloc[:,60]
```


```python
from keras.models import Sequential
from keras.layers import Dense
from sklearn.preprocessing import LabelEncoder
from sklearn.cross_validation import train_test_split

# seed = 60
# np.random.seed(seed)

encoder = LabelEncoder()
y1 = encoder.fit_transform(y)
Y = pd.get_dummies(y1).values

X_train, X_test, y_train, y_test = train_test_split(X, Y, test_size=0.2)
```


```python
model = Sequential()
model.add(Dense(100, input_shape=(60,), activation='relu'))
model.add(Dense(30, activation='relu'))
model.add(Dense(2, activation='sigmoid'))

model.compile(loss='categorical_crossentropy', optimizer='adam', metrics=['accuracy'])

print model.summary()
```

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    dense_31 (Dense)             (None, 100)               6100      
    _________________________________________________________________
    dense_32 (Dense)             (None, 30)                3030      
    _________________________________________________________________
    dense_33 (Dense)             (None, 2)                 62        
    =================================================================
    Total params: 9,192
    Trainable params: 9,192
    Non-trainable params: 0
    _________________________________________________________________
    None



```python
model.fit(X_train, y_train, epochs=50, batch_size=20, verbose=1)
```

    Epoch 1/50
    165/165 [==============================] - 0s 2ms/step - loss: 0.6822 - acc: 0.5455
    Epoch 2/50
    165/165 [==============================] - 0s 89us/step - loss: 0.6567 - acc: 0.5576
    Epoch 3/50
    165/165 [==============================] - 0s 79us/step - loss: 0.6388 - acc: 0.6303
    Epoch 4/50
    165/165 [==============================] - 0s 96us/step - loss: 0.6201 - acc: 0.6970
    Epoch 5/50
    165/165 [==============================] - 0s 87us/step - loss: 0.5982 - acc: 0.7273
    Epoch 6/50
    165/165 [==============================] - 0s 112us/step - loss: 0.5745 - acc: 0.7152
    Epoch 7/50
    165/165 [==============================] - 0s 106us/step - loss: 0.5503 - acc: 0.7455
    Epoch 8/50
    165/165 [==============================] - 0s 101us/step - loss: 0.5200 - acc: 0.7576
    Epoch 9/50
    165/165 [==============================] - 0s 110us/step - loss: 0.5151 - acc: 0.7212
    Epoch 10/50
    165/165 [==============================] - 0s 106us/step - loss: 0.4769 - acc: 0.7697
    Epoch 11/50
    165/165 [==============================] - 0s 103us/step - loss: 0.4538 - acc: 0.7939
    Epoch 12/50
    165/165 [==============================] - 0s 102us/step - loss: 0.4728 - acc: 0.8182
    Epoch 13/50
    165/165 [==============================] - 0s 112us/step - loss: 0.4350 - acc: 0.7818
    Epoch 14/50
    165/165 [==============================] - 0s 100us/step - loss: 0.4212 - acc: 0.8182
    Epoch 15/50
    165/165 [==============================] - 0s 101us/step - loss: 0.3978 - acc: 0.8303
    Epoch 16/50
    165/165 [==============================] - 0s 84us/step - loss: 0.3891 - acc: 0.8364
    Epoch 17/50
    165/165 [==============================] - 0s 97us/step - loss: 0.3796 - acc: 0.8303
    Epoch 18/50
    165/165 [==============================] - 0s 86us/step - loss: 0.3733 - acc: 0.8242
    Epoch 19/50
    165/165 [==============================] - 0s 105us/step - loss: 0.3673 - acc: 0.8424
    Epoch 20/50
    165/165 [==============================] - 0s 116us/step - loss: 0.3703 - acc: 0.8424
    Epoch 21/50
    165/165 [==============================] - 0s 104us/step - loss: 0.3629 - acc: 0.8303
    Epoch 22/50
    165/165 [==============================] - 0s 102us/step - loss: 0.3415 - acc: 0.8727
    Epoch 23/50
    165/165 [==============================] - 0s 85us/step - loss: 0.3452 - acc: 0.8606
    Epoch 24/50
    165/165 [==============================] - 0s 97us/step - loss: 0.3309 - acc: 0.8485
    Epoch 25/50
    165/165 [==============================] - 0s 82us/step - loss: 0.3284 - acc: 0.8485
    Epoch 26/50
    165/165 [==============================] - 0s 101us/step - loss: 0.3311 - acc: 0.8667
    Epoch 27/50
    165/165 [==============================] - 0s 103us/step - loss: 0.3105 - acc: 0.8848
    Epoch 28/50
    165/165 [==============================] - 0s 93us/step - loss: 0.3137 - acc: 0.8788
    Epoch 29/50
    165/165 [==============================] - 0s 77us/step - loss: 0.3223 - acc: 0.8545
    Epoch 30/50
    165/165 [==============================] - 0s 98us/step - loss: 0.2871 - acc: 0.8848
    Epoch 31/50
    165/165 [==============================] - 0s 105us/step - loss: 0.2887 - acc: 0.8848
    Epoch 32/50
    165/165 [==============================] - 0s 94us/step - loss: 0.2724 - acc: 0.8970
    Epoch 33/50
    165/165 [==============================] - 0s 80us/step - loss: 0.2920 - acc: 0.8667
    Epoch 34/50
    165/165 [==============================] - ETA: 0s - loss: 0.1849 - acc: 0.900 - 0s 91us/step - loss: 0.2619 - acc: 0.8909
    Epoch 35/50
    165/165 [==============================] - 0s 104us/step - loss: 0.2651 - acc: 0.9030
    Epoch 36/50
    165/165 [==============================] - 0s 92us/step - loss: 0.2527 - acc: 0.9091
    Epoch 37/50
    165/165 [==============================] - 0s 74us/step - loss: 0.2492 - acc: 0.9212
    Epoch 38/50
    165/165 [==============================] - 0s 103us/step - loss: 0.2413 - acc: 0.9091
    Epoch 39/50
    165/165 [==============================] - 0s 102us/step - loss: 0.2519 - acc: 0.8909
    Epoch 40/50
    165/165 [==============================] - 0s 76us/step - loss: 0.2286 - acc: 0.9212
    Epoch 41/50
    165/165 [==============================] - 0s 93us/step - loss: 0.2341 - acc: 0.8970
    Epoch 42/50
    165/165 [==============================] - 0s 107us/step - loss: 0.2233 - acc: 0.9152
    Epoch 43/50
    165/165 [==============================] - 0s 75us/step - loss: 0.2219 - acc: 0.9273
    Epoch 44/50
    165/165 [==============================] - 0s 87us/step - loss: 0.2209 - acc: 0.9212
    Epoch 45/50
    165/165 [==============================] - 0s 88us/step - loss: 0.2117 - acc: 0.9394
    Epoch 46/50
    165/165 [==============================] - 0s 92us/step - loss: 0.2071 - acc: 0.9152
    Epoch 47/50
    165/165 [==============================] - 0s 87us/step - loss: 0.1961 - acc: 0.9394
    Epoch 48/50
    165/165 [==============================] - 0s 76us/step - loss: 0.2127 - acc: 0.9273
    Epoch 49/50
    165/165 [==============================] - 0s 90us/step - loss: 0.1891 - acc: 0.9515
    Epoch 50/50
    165/165 [==============================] - 0s 72us/step - loss: 0.1796 - acc: 0.9636





    <keras.callbacks.History at 0x11d88af50>




```python
y_pred = model.predict(X_test)
print y_pred.mean()
```

     0.03981658



```python
y_test_class = np.argmax(y_test, axis=1)
y_pred_class = np.argmax(y_pred, axis=1)
print y_test_class
print y_pred_class
```

    [1 0 0 1 1 0 1 1 1 1 0 0 1 0 1 1 0 0 0 0 1 0 1 1 0 0 0 1 1 1 0 1 0 1 0 0 0
     0 1 1 0 0]
    [1 0 0 0 1 0 1 1 1 1 0 0 1 0 1 1 1 0 0 0 1 0 1 1 0 0 0 1 1 1 0 1 0 1 0 0 0
     0 1 1 0 0]


# 3. 使用混淆矩阵来进一步验证


```python
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report

print classification_report(y_test_class, y_pred_class, target_names=['R', 'M'])
print confusion_matrix(y_test_class, y_pred_class)

acct = accuracy_score(y_test_class, y_pred_class)
print 'Accuracy is: %.2f %%' % ( acct * 100)
```

                 precision    recall  f1-score   support

              R       0.95      0.95      0.95        22
              M       0.95      0.95      0.95        20

    avg / total       0.95      0.95      0.95        42

    [[21  1]
     [ 1 19]]
    Accuracy is: 95.24 %



```python
from keras.layers import Dropout
from keras.constraints import maxnorm
from keras.optimizers import Adam

model = Sequential()
model.add(Dense(100, input_shape=(60,), activation='relu', W_constraint=maxnorm(3)))
model.add(Dropout(0.2))
model.add(Dense(30, activation='relu', W_constraint=maxnorm(3)))
model.add(Dropout(0.2))
model.add(Dense(2, activation='sigmoid'))

adam = Adam(lr=0.1)
model.compile(loss='categorical_crossentropy', optimizer=adam, metrics=['accuracy'])

print model.summary()
```

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    dense_41 (Dense)             (None, 100)               6100      
    _________________________________________________________________
    dropout_5 (Dropout)          (None, 100)               0         
    _________________________________________________________________
    dense_42 (Dense)             (None, 30)                3030      
    _________________________________________________________________
    dropout_6 (Dropout)          (None, 30)                0         
    _________________________________________________________________
    dense_43 (Dense)             (None, 2)                 62        
    =================================================================
    Total params: 9,192
    Trainable params: 9,192
    Non-trainable params: 0
    _________________________________________________________________
    None


    /Users/yongliu/anaconda/lib/python2.7/site-packages/ipykernel_launcher.py:6: UserWarning: Update your `Dense` call to the Keras 2 API: `Dense(100, activation="relu", input_shape=(60,), kernel_constraint=<keras.con...)`

    /Users/yongliu/anaconda/lib/python2.7/site-packages/ipykernel_launcher.py:8: UserWarning: Update your `Dense` call to the Keras 2 API: `Dense(30, activation="relu", kernel_constraint=<keras.con...)`




```python
model.fit(X_train, y_train, epochs=50, batch_size=20, verbose=1)
y_pred = model.predict(X_test)
print y_pred.mean()
acct = accuracy_score(y_test_class, y_pred_class)
print 'Accuracy is: %.2f %%' % ( acct * 100)
```

    Epoch 1/50
    165/165 [==============================] - 0s 3ms/step - loss: 1.0228 - acc: 0.5152
    Epoch 2/50
    165/165 [==============================] - 0s 100us/step - loss: 0.6931 - acc: 0.5394
    Epoch 3/50
    165/165 [==============================] - 0s 103us/step - loss: 0.6931 - acc: 0.5394
    Epoch 4/50
    165/165 [==============================] - 0s 115us/step - loss: 0.6931 - acc: 0.5394
    Epoch 5/50
    165/165 [==============================] - 0s 114us/step - loss: 0.6931 - acc: 0.5394
    Epoch 6/50
    165/165 [==============================] - 0s 117us/step - loss: 0.6931 - acc: 0.5394
    Epoch 7/50
    165/165 [==============================] - 0s 93us/step - loss: 0.6931 - acc: 0.5394
    Epoch 8/50
    165/165 [==============================] - 0s 115us/step - loss: 0.6931 - acc: 0.5394
    Epoch 9/50
    165/165 [==============================] - 0s 118us/step - loss: 0.6931 - acc: 0.5394
    Epoch 10/50
    165/165 [==============================] - 0s 98us/step - loss: 0.6931 - acc: 0.5394
    Epoch 11/50
    165/165 [==============================] - 0s 114us/step - loss: 0.6931 - acc: 0.5394
    Epoch 12/50
    165/165 [==============================] - 0s 116us/step - loss: 0.6931 - acc: 0.5394
    Epoch 13/50
    165/165 [==============================] - 0s 116us/step - loss: 0.6931 - acc: 0.5394
    Epoch 14/50
    165/165 [==============================] - 0s 93us/step - loss: 0.6931 - acc: 0.5394
    Epoch 15/50
    165/165 [==============================] - 0s 122us/step - loss: 0.6931 - acc: 0.5394
    Epoch 16/50
    165/165 [==============================] - 0s 107us/step - loss: 0.6931 - acc: 0.5394
    Epoch 17/50
    165/165 [==============================] - 0s 99us/step - loss: 0.6931 - acc: 0.5394
    Epoch 18/50
    165/165 [==============================] - 0s 116us/step - loss: 0.6931 - acc: 0.5394
    Epoch 19/50
    165/165 [==============================] - 0s 115us/step - loss: 0.6931 - acc: 0.5394
    Epoch 20/50
    165/165 [==============================] - 0s 91us/step - loss: 0.6931 - acc: 0.5394
    Epoch 21/50
    165/165 [==============================] - 0s 117us/step - loss: 0.6931 - acc: 0.5394
    Epoch 22/50
    165/165 [==============================] - 0s 110us/step - loss: 0.6931 - acc: 0.5394
    Epoch 23/50
    165/165 [==============================] - 0s 105us/step - loss: 0.6931 - acc: 0.5394
    Epoch 24/50
    165/165 [==============================] - 0s 108us/step - loss: 0.6931 - acc: 0.5394
    Epoch 25/50
    165/165 [==============================] - 0s 96us/step - loss: 0.6931 - acc: 0.5394
    Epoch 26/50
    165/165 [==============================] - 0s 112us/step - loss: 0.6931 - acc: 0.5394
    Epoch 27/50
    165/165 [==============================] - 0s 101us/step - loss: 0.6931 - acc: 0.5394
    Epoch 28/50
    165/165 [==============================] - 0s 96us/step - loss: 0.6931 - acc: 0.5394
    Epoch 29/50
    165/165 [==============================] - 0s 112us/step - loss: 0.6931 - acc: 0.5394
    Epoch 30/50
    165/165 [==============================] - 0s 93us/step - loss: 0.6931 - acc: 0.5394
    Epoch 31/50
    165/165 [==============================] - 0s 97us/step - loss: 0.6931 - acc: 0.5394
    Epoch 32/50
    165/165 [==============================] - 0s 116us/step - loss: 0.6931 - acc: 0.5394
    Epoch 33/50
    165/165 [==============================] - 0s 90us/step - loss: 0.6931 - acc: 0.5394
    Epoch 34/50
    165/165 [==============================] - 0s 105us/step - loss: 0.6931 - acc: 0.5394
    Epoch 35/50
    165/165 [==============================] - 0s 104us/step - loss: 0.6931 - acc: 0.5394
    Epoch 36/50
    165/165 [==============================] - 0s 98us/step - loss: 0.6931 - acc: 0.5394
    Epoch 37/50
    165/165 [==============================] - 0s 102us/step - loss: 0.6931 - acc: 0.5394
    Epoch 38/50
    165/165 [==============================] - 0s 114us/step - loss: 0.6931 - acc: 0.5394
    Epoch 39/50
    165/165 [==============================] - 0s 102us/step - loss: 0.6931 - acc: 0.5394
    Epoch 40/50
    165/165 [==============================] - 0s 101us/step - loss: 0.6931 - acc: 0.5394
    Epoch 41/50
    165/165 [==============================] - 0s 98us/step - loss: 0.6931 - acc: 0.5394
    Epoch 42/50
    165/165 [==============================] - 0s 104us/step - loss: 0.6931 - acc: 0.5394
    Epoch 43/50
    165/165 [==============================] - 0s 92us/step - loss: 0.6931 - acc: 0.5394
    Epoch 44/50
    165/165 [==============================] - 0s 103us/step - loss: 0.6931 - acc: 0.5394
    Epoch 45/50
    165/165 [==============================] - 0s 97us/step - loss: 0.6931 - acc: 0.5394
    Epoch 46/50
    165/165 [==============================] - 0s 90us/step - loss: 0.6931 - acc: 0.5394
    Epoch 47/50
    165/165 [==============================] - 0s 99us/step - loss: 0.6931 - acc: 0.5394
    Epoch 48/50
    165/165 [==============================] - 0s 82us/step - loss: 0.6931 - acc: 0.5394
    Epoch 49/50
    165/165 [==============================] - 0s 92us/step - loss: 0.6931 - acc: 0.5394
    Epoch 50/50
    165/165 [==============================] - 0s 80us/step - loss: 0.6931 - acc: 0.5394
    1.0
    Accuracy is: 95.24 %
