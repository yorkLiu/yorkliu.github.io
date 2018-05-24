---
title: 利用Keras探索声呐物体分类
copyright: true
date: 2018-05-24 10:18:38
tags:
    - 机器学习
    - Keras
categories:
    - 机器学习
    - Keras
---

# 利用Keras探索声呐物体分类数据集

## 目录

- [1. 数据集介绍](#1.-数据集介绍)
- [2. 数据加载分析](#2.-数据加载分析)
- [3. 数据处理](#3.-数据处理)

# 1. 数据集介绍

本章使用声呐数据，包括声呐在不同物体的返回。数据有60个变量，代表不同角度的返回值。目标是将石头和金属筒（矿石）分开。

所有的数据都是连续的，从0到1；输出变量中M代表矿石，R代表石头，需要转换为1和0。数据集有208条数据

**数据下载地址**: http://archive.ics.uci.edu/ml/machine-learning-databases/undocumented/connectionist-bench/sonar/sonar.all-data

# 2. 数据加载分析


```python
import pandas as pd
import numpy as np

dataset = pd.read_csv('../data/sonar.csv', header=None)
```


```python
dataset.head()
```


```python
dataset.describe().transpose()
```


```python
dataset.iloc[:,60].unique()
```
array(['R', 'M'], dtype=object)

```python
print dataset.groupby(60).size()
```
    60
    M    111
    R     97
    dtype: int64


"sonar.csv"中，只有两种类别
- R: 石头
- M: 矿石

# 3. 构建一个简单的网络


```python
from keras.models import Sequential
from keras.layers import Dense
from keras.wrappers.scikit_learn import KerasClassifier
from sklearn.cross_validation import cross_val_score, StratifiedKFold
from sklearn.preprocessing import LabelEncoder, StandardScaler
from sklearn.pipeline import Pipeline

```


```python
seed = 60
np.random.seed(seed)
```


```python
X = dataset.iloc[:,0:60].values
y = dataset.iloc[:,60].values

```

    [[0.02   0.0371 0.0428 ... 0.0084 0.009  0.0032]
     [0.0453 0.0523 0.0843 ... 0.0049 0.0052 0.0044]
     [0.0262 0.0582 0.1099 ... 0.0164 0.0095 0.0078]
     ...
     [0.0522 0.0437 0.018  ... 0.0138 0.0077 0.0031]
     [0.0303 0.0353 0.049  ... 0.0079 0.0036 0.0048]
     [0.026  0.0363 0.0136 ... 0.0036 0.0061 0.0115]]


将 y 进行 **ONE-HOT** encoder


```python
encoder = LabelEncoder()
Y = encoder.fit_transform(y)

#Y = pd.get_dummies(y1).values
```


```python
def baseline_model():
    model = Sequential()
    model.add(Dense(100, input_dim=60, activation='relu'))
    model.add(Dense(1, activation='sigmoid'))

    model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'])
    return model
```


```python
estimator = KerasClassifier(build_fn=baseline_model, nb_epoch=150, batch_size=5, verbose=0)
kfold = StratifiedKFold(y=Y, n_folds=10, shuffle=True, random_state=seed)
scores = cross_val_score(estimator, X, Y, cv=kfold)
print 'Results %.2f %% (%.2f %%)' % (scores.mean()*100, scores.std()*100)
```

    Results 64.98 % (8.28 %)


accuracy 为: 64.98%, 标准差为: 8.28%, 效果也不是那么的差，因为实现的代码确实很简单

# 4. 预处理数据增加性能

预处理数据是个好习惯。神经网络喜欢输入类型的比例和分布一致，为了达到这点可以使用正则化，让数据的平均值是0，标准差是1，这样可以保留数据的分布情况。

scikit-learn的StandardScaler可以做到这点。不应该在整个数据集上直接应用正则化：应该只在测试数据上交叉验证时进行正则化处理，使正则化成为交叉验证的一环，让模型没有新数据的先验知识，防止模型发散。

scikit-learn的Pipeline可以直接做到这些。我们先定义一个 StandardScaler，然后进行验证：


```python
estimaters = []
estimaters.append(('standardize', StandardScaler()))
estimaters.append(('mlp', KerasClassifier(build_fn=baseline_model, nb_epoch=150, batch_size=5, verbose=0)))
pipeline = Pipeline(estimaters)

kfold = StratifiedKFold(y=Y, n_folds=10, shuffle=True, random_state=seed)
scores = cross_val_score(pipeline, X, Y, cv=kfold)
print 'Standardized %.2f %% (%.2f %%)' % (scores.mean()*100, scores.std()*100)
```

    Standardized 75.01 % (6.97 %)


增加了 Standardize 的确提高了 accuracy, 降低了 标准差

# 5. 调整模型的拓扑和神经元

神经网络有很多参数，例如初始化权重、激活函数、优化算法等等。我们一直没有说到调整网络的拓扑结构：扩大或缩小网络。我们试验一下：


## 4.1 缩小网络

有可能数据中有冗余：原始数据是不同角度的信号，有可能其中某些角度有相关性。我们把第一层隐层缩小一些，强行提取特征试试。

我们把之前的模型隐层的100个神经元减半到50个，这样神经网络需要挑选最重要的信息。之前的正则化有效果,也一并在这里做一下.


```python
def baseline_model():
    model = Sequential()
    model.add(Dense(50, input_dim=60, activation='relu'))
    model.add(Dense(1, activation='sigmoid'))

    model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'])
    return model

np.random.seed(seed)
estimaters = []
estimaters.append(('standardize', StandardScaler()))
estimaters.append(('mlp', KerasClassifier(build_fn=baseline_model, nb_epoch=150, batch_size=5, verbose=0)))
pipeline = Pipeline(estimaters)

kfold = StratifiedKFold(y=Y, n_folds=10, shuffle=True, random_state=seed)
scores = cross_val_score(pipeline, X, Y, cv=kfold)
print 'Standardized %.2f %% (%.2f %%)' % (scores.mean()*100, scores.std()*100)
```

    Standardized 74.15 % (11.23 %)


## 4.2 扩大网络
扩大网络后，神经网络更有可能提取关键特征，以非线性方式组合。


```python
def baseline_model():
    model = Sequential()
    model.add(Dense(120, input_dim=60, activation='relu'))
    model.add(Dense(60, activation='relu'))
    model.add(Dense(30, activation='relu'))
    model.add(Dense(15, activation='relu'))
    model.add(Dense(1, activation='sigmoid'))

    model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'])
    return model

np.random.seed(seed)
# estimaters = []
# estimaters.append(('standardize', StandardScaler()))
# estimaters.append(('mlp', KerasClassifier(build_fn=baseline_model, nb_epoch=150, batch_size=20, verbose=0)))
# pipeline = Pipeline(estimaters)

estimator = KerasClassifier(build_fn=baseline_model, nb_epoch=50, batch_size=10, verbose=0)
kfold = StratifiedKFold(y=Y, n_folds=5, shuffle=True, random_state=seed)
scores = cross_val_score(estimator, X, Y, cv=kfold)
print 'Standardized %.2f %% (%.2f %%)' % (scores.mean()*100, scores.std()*100)
```

    Standardized 61.50 % (8.02 %)
