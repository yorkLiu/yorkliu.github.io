---
title: 使用Keras为多类分类开发并验证一个神经网络
copyright: true
date: 2018-05-24 12:54:11
tags:
    - 机器学习
    - Keras
categories:
    - 机器学习
    - Keras
---

## 目录

- [1. 数据准备](#1.-数据准备)
- [2. 加载数据](#2.-加载数据)
- [3. 导入库和函数](#3.-导入库和函数)
- [4. 设计神经网络](#4.-设计神经网络)
- [5. 用K折交叉检验测试模型](#5.-用K折交叉检验测试模型)

https://www.kaggle.com/akashsri99/deep-learning-iris-dataset-keras/code


## 1. 数据准备
    1. https://archive.ics.uci.edu/ml/datasets/iris
    2. https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data

这个数据集已经被充分研究过，4个输入变量都是数字，量纲都是厘米。每个数据代表花朵的不同参数，输出是分类结果。数据的属性是（厘米）：

1. 萼片长度
2. 萼片宽度
3. 花瓣长度
4. 花瓣宽度
5. 类别    

## 2. 加载数据


```python
import pandas as pd
import numpy as np

iris = pd.read_csv('../data/iris.csv', header=None)
```


```python
iris.head()
```



```python
iris.describe()
```


```python
iris.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 150 entries, 0 to 149
    Data columns (total 5 columns):
    0    150 non-null float64
    1    150 non-null float64
    2    150 non-null float64
    3    150 non-null float64
    4    150 non-null object
    dtypes: float64(4), object(1)
    memory usage: 5.9+ KB



```python
iris.describe().transpose()
```



```python
iris.iloc[:,4].unique()
```


    array(['Iris-setosa', 'Iris-versicolor', 'Iris-virginica'], dtype=object)



## 3. 导入库和函数
导入所需要的库和函数，包括深度学习包Keras、数据处理包pandas和模型测试包scikit-learn


```python
from keras.models import Sequential
from keras.layers import Dense
from keras.wrappers.scikit_learn import KerasClassifier
from keras.utils import np_utils
from sklearn.cross_validation import KFold, cross_val_score
from sklearn.preprocessing import LabelEncoder
from sklearn.pipeline import Pipeline
import warnings
warnings.filterwarnings("ignore")
```


```python
seed = 7
np.random.seed(seed)
```

从上面打印出来的信息来看，iris.csv 文件中有三种类别，分别是： 'Iris-setosa', 'Iris-versicolor', 'Iris-virginica'， 所以我们需要将文字以 **数字**的形式来表示 (one-hot)编码


```python
dataset = iris.values
X = dataset[:,0:4].astype(float)
Y = dataset[:,4]
```


```python
encoder = LabelEncoder()
encoder.fit(Y)

encode_Y = encoder.transform(Y)
dummy_y = np_utils.to_categorical(encode_Y)
print dummy_y[0:10]
```

    [[1. 0. 0.]
     [1. 0. 0.]
     [1. 0. 0.]
     [1. 0. 0.]
     [1. 0. 0.]
     [1. 0. 0.]
     [1. 0. 0.]
     [1. 0. 0.]
     [1. 0. 0.]
     [1. 0. 0.]]


## 4. 设计神经网络
Keras提供了KerasClassifier，可以将网络封装，在scikit-learn上用。KerasClassifier的初始化变量是模型名称，返回供训练的神经网络模型。

我们写一个函数，为鸢尾花分类问题创建一个神经网络：这个全连接网络只有1个带有4个神经元的隐层，和输入的变量数相同。为了效果，隐层使用整流函数作为激活函数。因为我们用了 **ONE-HOT**编码，网络的输出必须是3个变量，每个变量代表一种花，最大的变量代表预测种类。网络的结构是：
> 4个神经元 输入层 -> [4个神经元 隐层] -> 3个神经元 输出层

输出层的函数是S型函数，把可能性映射到概率的0到1。优化算法选择ADAM随机梯度下降，损失函数是对数函数，在Keras中叫categorical_crossentropy：


```python
def baseline_model():
    model = Sequential()
    model.add(Dense(4, input_dim=4, init='uniform', activation='relu'))
    model.add(Dense(3, init='normal', activation='sigmoid'))

    model.compile(loss='categorical_crossentropy', optimizer='adam', metrics=['accuracy'])
#     print model.summary()
    return model
```


```python
estimator = KerasClassifier(build_fn=baseline_model, nb_epoch=300, batch_size=5, verbose=0)
```

## 5. 用K折交叉检验测试模型

现在可以测试模型效果了。scikit-learn有很多种办法可以测试模型，其中最重要的就是K折检验。我们先设定模型的测试方法：K设为10（默认值很好），在分割前随机重排数据：



```python
kfold = KFold(n=len(X), n_folds=10, shuffle=True, random_state=seed)
```


```python
results = cross_val_score(estimator, X, dummy_y, cv=kfold)
print 'Baseline %.2f %% (STD: %.2f %%)' % (results.mean() * 100, results.std()*100)
```

    Baseline 42.67 % (STD: 15.83 %)



```python
from sklearn.cross_validation import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, dummy_y, test_size=0.3, random_state=seed)
model = baseline_model()
results = model.fit(X_train, y_train, validation_data=(X_test, y_test), nb_epoch=200, batch_size=30, verbose=0)
scores = model.evaluate(X,dummy_y)
print '%s: %.3f' % (model.metrics_names[1], scores[1]*100)

```

    150/150 [==============================] - 0s 63us/step
    acc: 66.000



```python
from sklearn.metrics import classification_report, confusion_matrix

y_predict = model.predict(X_test)

y_test_class=np.argmax(y_test, axis=1)
y_predict_class=np.argmax(y_predict, axis=1)

print(classification_report(y_test_class, y_predict_class))
print(confusion_matrix(y_test_class, y_predict_class))
```

                 precision    recall  f1-score   support

              0       1.00      1.00      1.00        12
              1       0.48      1.00      0.65        16
              2       0.00      0.00      0.00        17

    avg / total       0.44      0.62      0.50        45

    [[12  0  0]
     [ 0 16  0]
     [ 0 17  0]]
