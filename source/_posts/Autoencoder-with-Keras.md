---
title: Autoencoder with Keras (Keras 实现 自编码器)
copyright: true
date: 2018-05-24 13:07:07
tags:
    - 机器学习
    - Keras
categories:
    - 机器学习
    - Keras
---

## 目录
- [1. Autoencoder 简介](#1.-Autoencoder 简介)
- [2. 自编码器](#2.-自编码器)
    - [2.1 基于全连接的简单自编码器（Baseline）](#2.1-基于全连接的简单自编码器（Baseline）)
    - [2.2 稀疏自编码（对Encoder层的输出进行限制](#2.2-稀疏自编码（对Encoder层的输出进行限制）)
    - [2.3 深度自编码器(对编码器和解码器的网络的深度进行扩展)](#2.3-深度自编码器(对编码器和解码器的网络的深度进行扩展))
    - [2.4 深度卷积自编码器器](#2.4-深度卷积自编码器器)


# 1. Autoencoder 简介

自编码器(Autoencoder):顾名思义,自编码器可以拆分为:自+编码器,当然有了编码器,自然对应的就会存在解码器,所以自编码器就可以理解为:自+编码器+解码器,用下图来解释就是,我们先将原始的数据通过某种操作进行(神经网络)**编码**得到加密数据A,然后我们对加密数据A进行**解码(神经网络)**得到解码后的数据B,而串联加密和解密设计的就是我们的**损失函数**,自编码中的损失函数的设计和我们的输入数据相关,它的Target或者Label就是我们的输入,所以我们可以认为这是一个**自**学习的过程,这就是我们的自编码器.
![](/uploads/autoencoder_schema.jpg)

在自编码器中,中间加密的数据的维度往往较低,因此自编码常常被用作降维操作等,不过实践中我们发现,在图片压缩等问题中,训练的自编码器的加密效果相比于JPEG之类的方法效果还是相对较差的。

不过自编码器本就不是专门用来做数据压缩的,它一个非常大的作用就是可以用作很多神经网络的预训练[2],从而帮助提高模型的准确率;此外它还可以帮助我们进行数据去噪处理以及低维数据可视化来辅助我们挖掘数据内部的结构特性等。不仅如此,自编码器还常常被认为可以为无监督学习带来重大突破的一件利器。


好了,那既然有这么大的潜力,下面我们就来学习一下常用的Autoencoder技术。我们按照下面的流程介绍几种流行的Autoencoder技术.

- 基于全连接的简单自编码器
- 稀疏自编码器
- 深度全连接自编码器
- 深度卷积自编码器器
- 图片去噪自编码模型
- 变分自编码器

# 2. 自编码器

## 2.1 基于全连接的简单自编码器（Baseline）
**网络结构**

此处我们先编写一个最简单的自编码器,编码用一层网(激活函数为ReLU),解码也用一层网络(激活函数为Sigmoid),然后我们用Sigmoid函数激活之后的输出作为我们模型的输出,具体的网络结构如下.


```python
from keras.layers import Input, Dense
from keras.models import Model
from keras.datasets import mnist
import numpy as np


encoding_dim=32
input_img = Input(shape=(784,))
encoded = Dense(encoding_dim, activation='relu')(input_img)
decoded = Dense(784, activation='sigmoid')(encoded)

autoencoder = Model(input_img, decoded)

print autoencoder.summary()

# define the encoder model
encoder = Model(input_img, encoded)

print encoder.summary()

# define the decoder model
encoded_input = Input(shape=(encoding_dim,))
decode_layer = autoencoder.layers[-1]
decoder = Model(encoded_input, decode_layer(encoded_input))

print decoder.summary()

```

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_72 (InputLayer)        (None, 784)               0         
    _________________________________________________________________
    dense_176 (Dense)            (None, 32)                25120     
    _________________________________________________________________
    dense_177 (Dense)            (None, 784)               25872     
    =================================================================
    Total params: 50,992
    Trainable params: 50,992
    Non-trainable params: 0
    _________________________________________________________________
    None
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_72 (InputLayer)        (None, 784)               0         
    _________________________________________________________________
    dense_176 (Dense)            (None, 32)                25120     
    =================================================================
    Total params: 25,120
    Trainable params: 25,120
    Non-trainable params: 0
    _________________________________________________________________
    None
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_73 (InputLayer)        (None, 32)                0         
    _________________________________________________________________
    dense_177 (Dense)            (None, 784)               25872     
    =================================================================
    Total params: 25,872
    Trainable params: 25,872
    Non-trainable params: 0
    _________________________________________________________________
    None


以上代码就将 autoencoder 模型创建好了，下面我们用mnist数据集来验证


```python
(x_train, y_train), (x_test, y_test) = mnist.load_data()
```


```python
print x_train.shape
print y_train.shape
print x_test.shape
print y_test.shape
```

    (60000, 28, 28)
    (60000,)
    (10000, 28, 28)
    (10000,)


因为Mnist数据集中的图片原始是[0-255]之间的,所以此处我们先对其进行预处理将所有像素点缩放至[0-1]之间,然后再进行模型的训练和测试



```python
x_train = x_train.astype('float32') / 255.
x_test = x_test.astype('float32') / 255.
x_train = x_train.reshape((len(x_train), np.prod(x_train.shape[1:])))
x_test = x_test.reshape((len(x_test), np.prod(x_test.shape[1:])))

print('X_train.shape:', x_train.shape, 'y_train.shape:', y_train.shape)
```

    ('X_train.shape:', (60000, 784), 'y_train.shape:', (60000,))



```python
autoencoder.compile(optimizer='adam', loss='binary_crossentropy')
autoencoder.fit(x_train, x_train,epochs=50,batch_size=256,shuffle=True,validation_data=(x_test, x_test))
```

    Train on 60000 samples, validate on 10000 samples
    Epoch 1/50
    60000/60000 [==============================] - 2s 32us/step - loss: 0.2790 - val_loss: 0.1938
    Epoch 2/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.1740 - val_loss: 0.1561
    Epoch 3/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.1451 - val_loss: 0.1333
    Epoch 4/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.1281 - val_loss: 0.1207
    Epoch 5/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.1177 - val_loss: 0.1123
    Epoch 6/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.1105 - val_loss: 0.1064
    Epoch 7/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.1053 - val_loss: 0.1020
    Epoch 8/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.1016 - val_loss: 0.0990
    Epoch 9/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0991 - val_loss: 0.0968
    Epoch 10/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0974 - val_loss: 0.0954
    Epoch 11/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.0962 - val_loss: 0.0944
    Epoch 12/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0954 - val_loss: 0.0939
    Epoch 13/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.0949 - val_loss: 0.0934
    Epoch 14/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.0945 - val_loss: 0.0931
    Epoch 15/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.0942 - val_loss: 0.0930
    Epoch 16/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.0941 - val_loss: 0.0928
    Epoch 17/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0939 - val_loss: 0.0925
    Epoch 18/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0937 - val_loss: 0.0925
    Epoch 19/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0936 - val_loss: 0.0924
    Epoch 20/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0935 - val_loss: 0.0923
    Epoch 21/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0934 - val_loss: 0.0922
    Epoch 22/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0934 - val_loss: 0.0922
    Epoch 23/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0933 - val_loss: 0.0921
    Epoch 24/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0933 - val_loss: 0.0921
    Epoch 25/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0932 - val_loss: 0.0921
    Epoch 26/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0932 - val_loss: 0.0920
    Epoch 27/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0931 - val_loss: 0.0920
    Epoch 28/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0931 - val_loss: 0.0918
    Epoch 29/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0930 - val_loss: 0.0919
    Epoch 30/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0930 - val_loss: 0.0919
    Epoch 31/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0930 - val_loss: 0.0918
    Epoch 32/50
    60000/60000 [==============================] - 2s 29us/step - loss: 0.0930 - val_loss: 0.0919
    Epoch 33/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0929 - val_loss: 0.0918
    Epoch 34/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0929 - val_loss: 0.0919
    Epoch 35/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0929 - val_loss: 0.0919
    Epoch 36/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0929 - val_loss: 0.0917
    Epoch 37/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0929 - val_loss: 0.0917
    Epoch 38/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0928 - val_loss: 0.0917
    Epoch 39/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0928 - val_loss: 0.0917
    Epoch 40/50
    60000/60000 [==============================] - 2s 33us/step - loss: 0.0928 - val_loss: 0.0917
    Epoch 41/50
    60000/60000 [==============================] - 2s 32us/step - loss: 0.0928 - val_loss: 0.0917
    Epoch 42/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0928 - val_loss: 0.0916
    Epoch 43/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0928 - val_loss: 0.0916
    Epoch 44/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0927 - val_loss: 0.0916
    Epoch 45/50
    60000/60000 [==============================] - 2s 29us/step - loss: 0.0927 - val_loss: 0.0916
    Epoch 46/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0927 - val_loss: 0.0917
    Epoch 47/50
    60000/60000 [==============================] - 2s 29us/step - loss: 0.0927 - val_loss: 0.0916
    Epoch 48/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0927 - val_loss: 0.0915
    Epoch 49/50
    60000/60000 [==============================] - 2s 32us/step - loss: 0.0927 - val_loss: 0.0915
    Epoch 50/50
    60000/60000 [==============================] - 2s 29us/step - loss: 0.0927 - val_loss: 0.0915





    <keras.callbacks.History at 0x12e9bce50>




```python
encoded_imgs = encoder.predict(x_test)
decoded_imgs = decoder.predict(encoded_imgs)
```


```python
import matplotlib.pyplot as plt

def showimages(x_test, decoded_images):
    n = 10
    plt.figure(figsize=(20, 4))
    for i in range(n):
        ax = plt.subplot(2, n, i+1)
        ax.imshow(x_test[i].reshape(28, 28))
        plt.gray()
        ax.get_xaxis().set_visible(False)
        ax.get_yaxis().set_visible(False)

        ax = plt.subplot(2, n, i+1+n)
        ax.imshow(decoded_images[i].reshape(28, 28))
        plt.gray()
        ax.get_xaxis().set_visible(False)
        ax.get_yaxis().set_visible(False)
    plt.show()     
```


```python
showimages(x_test, decoded_imgs)
```


![png](/uploads/Autoencoder%20with%20Keras_files/Autoencoder%20with%20Keras_12_0.png)


## 2.2 稀疏自编码（对Encoder层的输出进行限制）
上面的自编码器是最基础的自编码器,当我们将中间的激活函数改为线性的激活函数,同时将输出的激活函数改为线性的激活函数之后,上述的自编码器等价于PCA(主成分分析),关于自编码器与SVD,PCA的关系的分析可以参考论文《Auto-association by multilayer perceptrons and singular value decomposition》[4]. 这一节我们在上一节的基础上对中间层进行进一步操作。我们强制要求中间层是稀疏的(此处我们通过加L1正则项来完成),换言之，每次操作中间隐藏单元中只有少数的单元会被激活,而这些激活的单元就是重点。在Keras中,该操作可以通过activity_regularizer函数来达到我们所需的目的。


```python
from keras import regularizers

encoding_dim=32
input_img = Input(shape=(784,))
encoded = Dense(encoding_dim, activation='relu', activity_regularizer=regularizers.l1(10e-10))(input_img)
decoded = Dense(784, activation='sigmoid')(encoded)

autoencoder = Model(input_img, decoded)

print autoencoder.summary()

# define the encoder model
encoder = Model(input_img, encoded)

print encoder.summary()

# define the decoder model
encoded_input = Input(shape=(encoding_dim,))
decode_layer = autoencoder.layers[-1]
decoder = Model(encoded_input, decode_layer(encoded_input))

print decoder.summary()

(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train = x_train.astype('float32') / 255.
x_test = x_test.astype('float32') / 255.
x_train = x_train.reshape((len(x_train), np.prod(x_train.shape[1:])))
x_test = x_test.reshape((len(x_test), np.prod(x_test.shape[1:])))

print('X_train.shape:', x_train.shape, 'y_train.shape:', y_train.shape)

autoencoder.compile(optimizer='adam', loss='binary_crossentropy')
autoencoder.fit(x_train, x_train,epochs=50,batch_size=256,shuffle=True,validation_data=(x_test, x_test))

encoded_imgs = encoder.predict(x_test)
decoded_imgs = decoder.predict(encoded_imgs)
```

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_27 (InputLayer)        (None, 784)               0         
    _________________________________________________________________
    dense_33 (Dense)             (None, 32)                25120     
    _________________________________________________________________
    dense_34 (Dense)             (None, 784)               25872     
    =================================================================
    Total params: 50,992
    Trainable params: 50,992
    Non-trainable params: 0
    _________________________________________________________________
    None
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_27 (InputLayer)        (None, 784)               0         
    _________________________________________________________________
    dense_33 (Dense)             (None, 32)                25120     
    =================================================================
    Total params: 25,120
    Trainable params: 25,120
    Non-trainable params: 0
    _________________________________________________________________
    None
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_28 (InputLayer)        (None, 32)                0         
    _________________________________________________________________
    dense_34 (Dense)             (None, 784)               25872     
    =================================================================
    Total params: 25,872
    Trainable params: 25,872
    Non-trainable params: 0
    _________________________________________________________________
    None
    ('X_train.shape:', (60000, 784), 'y_train.shape:', (60000,))
    Train on 60000 samples, validate on 10000 samples
    Epoch 1/50
    60000/60000 [==============================] - 2s 35us/step - loss: 0.2754 - val_loss: 0.1887
    Epoch 2/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.1706 - val_loss: 0.1534
    Epoch 3/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.1443 - val_loss: 0.1342
    Epoch 4/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.1288 - val_loss: 0.1217
    Epoch 5/50
    60000/60000 [==============================] - 2s 29us/step - loss: 0.1181 - val_loss: 0.1128
    Epoch 6/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.1106 - val_loss: 0.1063
    Epoch 7/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.1053 - val_loss: 0.1019
    Epoch 8/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.1015 - val_loss: 0.0987
    Epoch 9/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0988 - val_loss: 0.0965
    Epoch 10/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0970 - val_loss: 0.0951
    Epoch 11/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0958 - val_loss: 0.0941
    Epoch 12/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0950 - val_loss: 0.0934
    Epoch 13/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.0945 - val_loss: 0.0931
    Epoch 14/50
    60000/60000 [==============================] - 2s 25us/step - loss: 0.0942 - val_loss: 0.0928
    Epoch 15/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0940 - val_loss: 0.0926
    Epoch 16/50
    60000/60000 [==============================] - 2s 26us/step - loss: 0.0938 - val_loss: 0.0924
    Epoch 17/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0936 - val_loss: 0.0924
    Epoch 18/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0935 - val_loss: 0.0922
    Epoch 19/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0934 - val_loss: 0.0922
    Epoch 20/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0933 - val_loss: 0.0920
    Epoch 21/50
    60000/60000 [==============================] - 2s 29us/step - loss: 0.0933 - val_loss: 0.0920
    Epoch 22/50
    60000/60000 [==============================] - 2s 32us/step - loss: 0.0932 - val_loss: 0.0920
    Epoch 23/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0932 - val_loss: 0.0919
    Epoch 24/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0931 - val_loss: 0.0919
    Epoch 25/50
    60000/60000 [==============================] - 2s 27us/step - loss: 0.0931 - val_loss: 0.0919
    Epoch 26/50
    60000/60000 [==============================] - 2s 28us/step - loss: 0.0930 - val_loss: 0.0918
    Epoch 27/50
    60000/60000 [==============================] - 2s 31us/step - loss: 0.0930 - val_loss: 0.0918
    Epoch 28/50
    60000/60000 [==============================] - 2s 29us/step - loss: 0.0930 - val_loss: 0.0917
    Epoch 29/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0930 - val_loss: 0.0918
    Epoch 30/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0929 - val_loss: 0.0917
    Epoch 31/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0929 - val_loss: 0.0918
    Epoch 32/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0929 - val_loss: 0.0917
    Epoch 33/50
    60000/60000 [==============================] - 2s 31us/step - loss: 0.0928 - val_loss: 0.0917
    Epoch 34/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0928 - val_loss: 0.0917
    Epoch 35/50
    60000/60000 [==============================] - 2s 32us/step - loss: 0.0928 - val_loss: 0.0917
    Epoch 36/50
    60000/60000 [==============================] - 2s 31us/step - loss: 0.0928 - val_loss: 0.0916
    Epoch 37/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0928 - val_loss: 0.0916
    Epoch 38/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0928 - val_loss: 0.0916
    Epoch 39/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0928 - val_loss: 0.0917
    Epoch 40/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0927 - val_loss: 0.0916
    Epoch 41/50
    60000/60000 [==============================] - 2s 29us/step - loss: 0.0927 - val_loss: 0.0916
    Epoch 42/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0927 - val_loss: 0.0915
    Epoch 43/50
    60000/60000 [==============================] - 2s 29us/step - loss: 0.0927 - val_loss: 0.0916
    Epoch 44/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0927 - val_loss: 0.0915
    Epoch 45/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0927 - val_loss: 0.0916
    Epoch 46/50
    60000/60000 [==============================] - 2s 33us/step - loss: 0.0927 - val_loss: 0.0915
    Epoch 47/50
    60000/60000 [==============================] - 2s 32us/step - loss: 0.0927 - val_loss: 0.0916
    Epoch 48/50
    60000/60000 [==============================] - 2s 31us/step - loss: 0.0926 - val_loss: 0.0915
    Epoch 49/50
    60000/60000 [==============================] - 2s 31us/step - loss: 0.0926 - val_loss: 0.0915
    Epoch 50/50
    60000/60000 [==============================] - 2s 30us/step - loss: 0.0926 - val_loss: 0.0915



```python
showimages(x_test, decoded_imgs)
```


![png](/uploads/Autoencoder%20with%20Keras_files/Autoencoder%20with%20Keras_15_0.png)


貌似也没有什么太大的变化

# 2.3 深度自编码器(对编码器和解码器的网络的深度进行扩展)
上面的两个模型都是简单的两层网络,第一层用于编码,第二层用于解码。此处我们对自编码器的深度进行扩展


```python

input_img = Input(shape=(784,))
encoded = Dense(128, activation='relu')(input_img)
encoded = Dense(64, activation='relu')(encoded)
encoded = Dense(32, activation='relu')(encoded)

decoded = Dense(64, activation='relu')(encoded)
decoded = Dense(128, activation='relu')(decoded)
decoded = Dense(784, activation='sigmoid')(decoded)

autoencoder = Model(input_img, decoded)
autoencoder.compile(optimizer='adam', loss='binary_crossentropy')
print autoencoder.summary()

autoencoder.fit(x_train, x_train,
                epochs=50,
                batch_size=256,
                shuffle=True,
                validation_data=(x_test, x_test))

encoded_imgs_deep = encoder.predict(x_test)
decoded_imgs_deep = decoder.predict(encoded_imgs_deep)
```

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_74 (InputLayer)        (None, 784)               0         
    _________________________________________________________________
    dense_178 (Dense)            (None, 128)               100480    
    _________________________________________________________________
    dense_179 (Dense)            (None, 64)                8256      
    _________________________________________________________________
    dense_180 (Dense)            (None, 32)                2080      
    _________________________________________________________________
    dense_181 (Dense)            (None, 64)                2112      
    _________________________________________________________________
    dense_182 (Dense)            (None, 128)               8320      
    _________________________________________________________________
    dense_183 (Dense)            (None, 784)               101136    
    =================================================================
    Total params: 222,384
    Trainable params: 222,384
    Non-trainable params: 0
    _________________________________________________________________
    None
    Train on 60000 samples, validate on 10000 samples
    Epoch 1/50
    60000/60000 [==============================] - 3s 57us/step - loss: 0.2421 - val_loss: 0.1642
    Epoch 2/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.1457 - val_loss: 0.1315
    Epoch 3/50
    60000/60000 [==============================] - 3s 42us/step - loss: 0.1264 - val_loss: 0.1198
    Epoch 4/50
    60000/60000 [==============================] - 2s 40us/step - loss: 0.1185 - val_loss: 0.1146
    Epoch 5/50
    60000/60000 [==============================] - 2s 40us/step - loss: 0.1138 - val_loss: 0.1103
    Epoch 6/50
    60000/60000 [==============================] - 3s 42us/step - loss: 0.1105 - val_loss: 0.1075
    Epoch 7/50
    60000/60000 [==============================] - 3s 44us/step - loss: 0.1076 - val_loss: 0.1049
    Epoch 8/50
    60000/60000 [==============================] - 3s 43us/step - loss: 0.1051 - val_loss: 0.1027
    Epoch 9/50
    60000/60000 [==============================] - 3s 42us/step - loss: 0.1029 - val_loss: 0.1010
    Epoch 10/50
    60000/60000 [==============================] - 3s 44us/step - loss: 0.1011 - val_loss: 0.0996
    Epoch 11/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.0994 - val_loss: 0.0977
    Epoch 12/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.0980 - val_loss: 0.0963
    Epoch 13/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.0967 - val_loss: 0.0950
    Epoch 14/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0957 - val_loss: 0.0944
    Epoch 15/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0948 - val_loss: 0.0935
    Epoch 16/50
    60000/60000 [==============================] - 3s 48us/step - loss: 0.0939 - val_loss: 0.0928
    Epoch 17/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0931 - val_loss: 0.0918
    Epoch 18/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.0923 - val_loss: 0.0913
    Epoch 19/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0915 - val_loss: 0.0906
    Epoch 20/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0908 - val_loss: 0.0903
    Epoch 21/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0903 - val_loss: 0.0901
    Epoch 22/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0899 - val_loss: 0.0888
    Epoch 23/50
    60000/60000 [==============================] - 3s 48us/step - loss: 0.0894 - val_loss: 0.0888
    Epoch 24/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0889 - val_loss: 0.0883
    Epoch 25/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.0885 - val_loss: 0.0878
    Epoch 26/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0882 - val_loss: 0.0872
    Epoch 27/50
    60000/60000 [==============================] - 3s 49us/step - loss: 0.0878 - val_loss: 0.0873
    Epoch 28/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0874 - val_loss: 0.0866
    Epoch 29/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0871 - val_loss: 0.0866
    Epoch 30/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.0868 - val_loss: 0.0864
    Epoch 31/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0865 - val_loss: 0.0860
    Epoch 32/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0863 - val_loss: 0.0857
    Epoch 33/50
    60000/60000 [==============================] - 3s 48us/step - loss: 0.0860 - val_loss: 0.0855
    Epoch 34/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0858 - val_loss: 0.0852
    Epoch 35/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0855 - val_loss: 0.0849
    Epoch 36/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0853 - val_loss: 0.0845
    Epoch 37/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0851 - val_loss: 0.0849
    Epoch 38/50
    60000/60000 [==============================] - 3s 49us/step - loss: 0.0849 - val_loss: 0.0842
    Epoch 39/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0847 - val_loss: 0.0840
    Epoch 40/50
    60000/60000 [==============================] - 3s 51us/step - loss: 0.0846 - val_loss: 0.0840
    Epoch 41/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0844 - val_loss: 0.0836
    Epoch 42/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0842 - val_loss: 0.0836
    Epoch 43/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0840 - val_loss: 0.0834
    Epoch 44/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0839 - val_loss: 0.0830
    Epoch 45/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.0837 - val_loss: 0.0832
    Epoch 46/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0836 - val_loss: 0.0830
    Epoch 47/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.0834 - val_loss: 0.0832
    Epoch 48/50
    60000/60000 [==============================] - 3s 45us/step - loss: 0.0833 - val_loss: 0.0830
    Epoch 49/50
    60000/60000 [==============================] - 3s 47us/step - loss: 0.0832 - val_loss: 0.0829
    Epoch 50/50
    60000/60000 [==============================] - 3s 46us/step - loss: 0.0831 - val_loss: 0.0828



```python
showimages(x_test, decoded_imgs_deep)
```


![png](/uploads/Autoencoder%20with%20Keras_files/Autoencoder%20with%20Keras_19_0.png)


不知道为decode 出来这这个样子

## 2.4 深度卷积自编码器器

CNN是目前自然语言处理中和RNN并驾齐驱的两种最常见的深度学习模型。图1展示了在NLP任务中使用CNN模型的典型网络结构。一般而言，输入的字或者词用Word Embedding的方式表达，这样本来一维的文本信息输入就转换成了二维的输入结构，假设输入X包含m个字符，而每个字符的Word Embedding的长度为d，那么输入就是m*d的二维向量。
![](/uploads/CNN-with-Pooling.png)

下面这组动画很好的诠释了 "卷积"与 "Pooling层"的关系
![](/uploads/CNN-with-Pooling.gif)

这里可以看出，因为NLP中的句子长度是不同的，所以CNN的输入矩阵大小是不确定的，这取决于m的大小是多少。卷积层本质上是个特征抽取层，可以设定超参数F来指定设立多少个特征抽取器（Filter），对于某个Filter来说，可以想象有一个k*d大小的移动窗口从输入矩阵的第一个字开始不断往后移动，其中k是Filter指定的窗口大小，d是Word Embedding长度。对于某个时刻的窗口，通过神经网络的非线性变换，将这个窗口内的输入值转换为某个特征值，随着窗口不断往后移动，这个Filter对应的特征值不断产生，形成这个Filter的特征向量。这就是卷积层抽取特征的过程。每个Filter都如此操作，形成了不同的特征抽取器。Pooling 层则对Filter的特征进行降维操作，形成最终的特征。一般在Pooling层之后连接全联接层神经网络，形成最后的分类过程。



可见，卷积和Pooling是CNN中最重要的两个步骤。下面我们重点介绍NLP中CNN模型常见的Pooling操作方法。

**CNN中的Max Pooling Over Time操作**

MaxPooling Over Time是NLP中CNN模型中最常见的一种下采样操作。意思是对于某个Filter抽取到若干特征值，只取其中得分最大的那个值作为Pooling层保留值，其它特征值全部抛弃，值最大代表只保留这些特征中最强的，而抛弃其它弱的此类特征。



CNN中采用Max Pooling操作有几个好处：首先，这个操作可以保证特征的位置与旋转不变性，因为不论这个强特征在哪个位置出现，都会不考虑其出现位置而能把它提出来。对于图像处理来说这种位置与旋转不变性是很好的特性，但是对于NLP来说，这个特性其实并不一定是好事，因为在很多NLP的应用场合，特征的出现位置信息是很重要的，比如主语出现位置一般在句子头，宾语一般出现在句子尾等等，这些位置信息其实有时候对于分类任务来说还是很重要的，但是Max Pooling 基本把这些信息抛掉了。



其次，MaxPooling能减少模型参数数量，有利于减少模型过拟合问题。因为经过Pooling操作后，往往把2D或者1D的数组转换为单一数值，这样对于后续的Convolution层或者全联接隐层来说无疑单个Filter的参数或者隐层神经元个数就减少了。



 再者，对于NLP任务来说，Max Pooling有个额外的好处；在此处，可以把变长的输入X整理成固定长度的输入。因为CNN最后往往会接全联接层，而其神经元个数是需要事先定好的，如果输入是不定长的那么很难设计网络结构。前文说过,CNN模型的输入X长度是不确定的，而通过Pooling 操作，每个Filter固定取1个值，那么有多少个Filter，Pooling层就有多少个神经元，这样就可以把全联接层神经元个数固定住（如图2所示），这个优点也是非常重要的。
>Pooling层神经元个数等于Filters个数
> ![](/uploads/CNN-with-Pooling-02.png)

**Max Pooling 缺点**

但是，CNN模型采取MaxPooling Over Time也有一些值得注意的缺点：首先就如上所述，特征的位置信息在这一步骤完全丢失。在卷积层其实是保留了特征的位置信息的，但是通过取唯一的最大值，现在在Pooling层只知道这个最大值是多少，但是其出现位置信息并没有保留；另外一个明显的缺点是：有时候有些强特征会出现多次，比如我们常见的TF-IDF公式，TF就是指某个特征出现的次数，出现次数越多说明这个特征越强，但是因为Max Pooling只保留一个最大值，所以即使某个特征出现多次，现在也只能看到一次，就是说同一特征的强度信息丢失了。这是Max Pooling Over Time典型的两个缺点。



其实，我们常说“危机危机”，对这个词汇乐观的解读是“危险就是机遇”。同理，发现模型的缺点是个好事情，因为创新往往就是通过改进模型的缺点而引发出来的。那么怎么改进Pooling层的机制能够缓解上述问题呢？下面两个常见的改进Pooling机制就是干这个事情的。


```python
from keras.layers import Input, Dense, Conv2D, MaxPooling2D, UpSampling2D
from keras.models import Model

input_img = Input(shape=(28,28,1))

x = Conv2D(16, (3, 3), activation='relu', padding='same')(input_img)
x = MaxPooling2D((2,2), padding='same')(x)
x = Conv2D(8, (3, 3), activation='relu', padding='same')(x)
x = MaxPooling2D((2,2), padding='same')(x)
x = Conv2D(8, (3,3), activation='relu', padding='same')(x)
encoded = MaxPooling2D((2,2), padding='same')(x)

x = Conv2D(8, (3,3), activation='relu', padding='same')(encoded)
x = UpSampling2D((2,2))(x)
x = Conv2D(8, (3,3), activation='relu', padding='same')(x)
x = UpSampling2D((2,2),)(x)
x = Conv2D(16, (3,3), activation='relu')(x)
x = UpSampling2D((2,2))(x)
decoded = Conv2D(1, (3,3), activation='sigmoid', padding='same')(x)

autoencoder = Model(input_img, decoded)
autoencoder.compile(loss='binary_crossentropy', optimizer='adam')
autoencoder.summary()

```

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_87 (InputLayer)        (None, 28, 28, 1)         0         
    _________________________________________________________________
    conv2d_78 (Conv2D)           (None, 28, 28, 16)        160       
    _________________________________________________________________
    max_pooling2d_43 (MaxPooling (None, 14, 14, 16)        0         
    _________________________________________________________________
    conv2d_79 (Conv2D)           (None, 14, 14, 8)         1160      
    _________________________________________________________________
    max_pooling2d_44 (MaxPooling (None, 7, 7, 8)           0         
    _________________________________________________________________
    conv2d_80 (Conv2D)           (None, 7, 7, 8)           584       
    _________________________________________________________________
    max_pooling2d_45 (MaxPooling (None, 4, 4, 8)           0         
    _________________________________________________________________
    conv2d_81 (Conv2D)           (None, 4, 4, 8)           584       
    _________________________________________________________________
    up_sampling2d_28 (UpSampling (None, 8, 8, 8)           0         
    _________________________________________________________________
    conv2d_82 (Conv2D)           (None, 8, 8, 8)           584       
    _________________________________________________________________
    up_sampling2d_29 (UpSampling (None, 16, 16, 8)         0         
    _________________________________________________________________
    conv2d_83 (Conv2D)           (None, 14, 14, 16)        1168      
    _________________________________________________________________
    up_sampling2d_30 (UpSampling (None, 28, 28, 16)        0         
    _________________________________________________________________
    conv2d_84 (Conv2D)           (None, 28, 28, 1)         145       
    =================================================================
    Total params: 4,385
    Trainable params: 4,385
    Non-trainable params: 0
    _________________________________________________________________



```python
from keras.datasets import mnist
import numpy as np

(x_train,_), (x_test,_) = mnist.load_data()

print x_train.shape
print x_test.shape

x_train = x_train.astype('float32') / 255
x_test = x_test.astype('float32') / 255

x_train = np.reshape(x_train, (len(x_train), 28, 28, 1))
x_test = np.reshape(x_test, (len(x_test), 28, 28, 1))
print x_train.shape
print x_test.shape

```

    (60000, 28, 28)
    (10000, 28, 28)
    (60000, 28, 28, 1)
    (10000, 28, 28, 1)


open `terminal` and start the tensorboard
```bash
tensorboard --logdir=~/Downloads/autoencoder/
```


```python
from keras.callbacks import TensorBoard

autoencoder.fit(x_train, x_train,
                epochs=50,
                batch_size=128,
                validation_data=(x_test, x_test),
                callbacks=[TensorBoard(log_dir='~/Downloads/autoencoder/')])

```

    Train on 60000 samples, validate on 10000 samples
    Epoch 1/50
    60000/60000 [==============================] - 58s 974us/step - loss: 0.2141 - val_loss: 0.1449
    Epoch 2/50
    60000/60000 [==============================] - 56s 940us/step - loss: 0.1350 - val_loss: 0.1258
    Epoch 3/50
    60000/60000 [==============================] - 54s 902us/step - loss: 0.1226 - val_loss: 0.1178
    Epoch 4/50
    60000/60000 [==============================] - 54s 905us/step - loss: 0.1163 - val_loss: 0.1129
    Epoch 5/50
    60000/60000 [==============================] - 55s 923us/step - loss: 0.1121 - val_loss: 0.1093
    Epoch 6/50
    60000/60000 [==============================] - 55s 910us/step - loss: 0.1091 - val_loss: 0.1073
    Epoch 7/50
    60000/60000 [==============================] - 58s 971us/step - loss: 0.1069 - val_loss: 0.1048
    Epoch 8/50
    60000/60000 [==============================] - 60s 1ms/step - loss: 0.1052 - val_loss: 0.1033
    Epoch 9/50
    60000/60000 [==============================] - 58s 967us/step - loss: 0.1038 - val_loss: 0.1022
    Epoch 10/50
    60000/60000 [==============================] - 57s 945us/step - loss: 0.1026 - val_loss: 0.1012
    Epoch 11/50
    60000/60000 [==============================] - 55s 918us/step - loss: 0.1014 - val_loss: 0.0999
    Epoch 12/50
    60000/60000 [==============================] - 59s 976us/step - loss: 0.1005 - val_loss: 0.0990
    Epoch 13/50
    60000/60000 [==============================] - 59s 977us/step - loss: 0.0996 - val_loss: 0.0992
    Epoch 14/50
    60000/60000 [==============================] - 61s 1ms/step - loss: 0.0989 - val_loss: 0.0976
    Epoch 15/50
    60000/60000 [==============================] - 61s 1ms/step - loss: 0.0983 - val_loss: 0.0972
    Epoch 16/50
    60000/60000 [==============================] - 61s 1ms/step - loss: 0.0976 - val_loss: 0.0964
    Epoch 17/50
    60000/60000 [==============================] - 60s 1ms/step - loss: 0.0971 - val_loss: 0.0956
    Epoch 18/50
    60000/60000 [==============================] - 60s 1ms/step - loss: 0.0965 - val_loss: 0.0951
    Epoch 19/50
    60000/60000 [==============================] - 59s 978us/step - loss: 0.0961 - val_loss: 0.0949
    Epoch 20/50
    60000/60000 [==============================] - 57s 956us/step - loss: 0.0958 - val_loss: 0.0951
    Epoch 21/50
    60000/60000 [==============================] - 57s 955us/step - loss: 0.0954 - val_loss: 0.0941
    Epoch 22/50
    60000/60000 [==============================] - 57s 954us/step - loss: 0.0951 - val_loss: 0.0937
    Epoch 23/50
    60000/60000 [==============================] - 58s 967us/step - loss: 0.0948 - val_loss: 0.0935
    Epoch 24/50
    60000/60000 [==============================] - 59s 986us/step - loss: 0.0944 - val_loss: 0.0932
    Epoch 25/50
    60000/60000 [==============================] - 59s 988us/step - loss: 0.0942 - val_loss: 0.0931
    Epoch 26/50
    60000/60000 [==============================] - 59s 987us/step - loss: 0.0939 - val_loss: 0.0927
    Epoch 27/50
    60000/60000 [==============================] - 59s 978us/step - loss: 0.0937 - val_loss: 0.0925
    Epoch 28/50
    60000/60000 [==============================] - 57s 956us/step - loss: 0.0934 - val_loss: 0.0924
    Epoch 29/50
    60000/60000 [==============================] - 58s 973us/step - loss: 0.0932 - val_loss: 0.0919
    Epoch 30/50
    60000/60000 [==============================] - 57s 945us/step - loss: 0.0931 - val_loss: 0.0920
    Epoch 31/50
    60000/60000 [==============================] - 57s 949us/step - loss: 0.0928 - val_loss: 0.0915
    Epoch 32/50
    60000/60000 [==============================] - 58s 962us/step - loss: 0.0926 - val_loss: 0.0918
    Epoch 33/50
    60000/60000 [==============================] - 59s 983us/step - loss: 0.0924 - val_loss: 0.0911
    Epoch 34/50
    60000/60000 [==============================] - 58s 973us/step - loss: 0.0923 - val_loss: 0.0910
    Epoch 35/50
    60000/60000 [==============================] - 60s 999us/step - loss: 0.0921 - val_loss: 0.0909
    Epoch 36/50
    60000/60000 [==============================] - 59s 979us/step - loss: 0.0920 - val_loss: 0.0910
    Epoch 37/50
    60000/60000 [==============================] - 58s 973us/step - loss: 0.0918 - val_loss: 0.0905
    Epoch 38/50
    60000/60000 [==============================] - 60s 993us/step - loss: 0.0917 - val_loss: 0.0909
    Epoch 39/50
    60000/60000 [==============================] - 60s 992us/step - loss: 0.0916 - val_loss: 0.0905
    Epoch 40/50
    60000/60000 [==============================] - 59s 988us/step - loss: 0.0914 - val_loss: 0.0905
    Epoch 41/50
    60000/60000 [==============================] - 60s 998us/step - loss: 0.0913 - val_loss: 0.0904
    Epoch 42/50
    60000/60000 [==============================] - 59s 990us/step - loss: 0.0912 - val_loss: 0.0901
    Epoch 43/50
    60000/60000 [==============================] - 60s 996us/step - loss: 0.0911 - val_loss: 0.0899
    Epoch 44/50
    60000/60000 [==============================] - 60s 999us/step - loss: 0.0910 - val_loss: 0.0898
    Epoch 45/50
    60000/60000 [==============================] - 60s 993us/step - loss: 0.0909 - val_loss: 0.0899
    Epoch 46/50
    60000/60000 [==============================] - 56s 936us/step - loss: 0.0908 - val_loss: 0.0900
    Epoch 47/50
    60000/60000 [==============================] - 58s 973us/step - loss: 0.0907 - val_loss: 0.0898
    Epoch 48/50
    60000/60000 [==============================] - 60s 997us/step - loss: 0.0907 - val_loss: 0.0897
    Epoch 49/50
    60000/60000 [==============================] - 60s 996us/step - loss: 0.0905 - val_loss: 0.0899
    Epoch 50/50
    60000/60000 [==============================] - 60s 995us/step - loss: 0.0905 - val_loss: 0.0896





    <keras.callbacks.History at 0x140e47710>



当`TensorBoard`启动起来之后，就可以访问 http://0.0.0.0:6006/ 来看 `loss` and `val_loss`的变化曲线了!
![](/uploads/CNN_Tensorboard.png)


```python
decode_imgs = autoencoder.predict(x_test)

showimages(x_test, decoded_imgs)
```


![png](/uploads/Autoencoder%20with%20Keras_files/Autoencoder%20with%20Keras_27_0.png)


没什么太大的变化，但看上去经过 卷积之后的图片稍微清晰了一点点，但训练时间的确增加了不少!

# 2.5 去噪自编码器

去噪自编码器相比于上述的自编码器最大的区别就是在于我们将原始数据输入模型时,先对数据进行加噪处理,但是我们输出的label仍然是我们未加噪的数据,这样做的好处可以加到模型的鲁棒性


```python
noise_factor = 0.5

x_train_noise = x_train + noise_factor * np.random.normal(size=x_train.shape)
x_test_noise = x_test + noise_factor * np.random.normal(size=x_test.shape)

x_train_noise = np.clip(x_train_noise, 0., 1.)
x_test_noise = np.clip(x_test_noise, 0., 1.)
```


```python
showimages(x_test, x_test_noise)
```


![png](/uploads/Autoencoder%20with%20Keras_files/Autoencoder%20with%20Keras_31_0.png)



```python
autoencoder.fit(x_train_noise, x_train,
                epochs=5,
                batch_size=128,
                validation_data=(x_test_noise, x_test),
                callbacks=[TensorBoard(log_dir='~/Downloads/autoencoder')])

```

    Train on 60000 samples, validate on 10000 samples
    Epoch 1/5
    60000/60000 [==============================] - 55s 923us/step - loss: 0.1238 - val_loss: 0.1210
    Epoch 2/5
    60000/60000 [==============================] - 59s 989us/step - loss: 0.1218 - val_loss: 0.1203
    Epoch 3/5
    60000/60000 [==============================] - 57s 957us/step - loss: 0.1213 - val_loss: 0.1198
    Epoch 4/5
    60000/60000 [==============================] - 63s 1ms/step - loss: 0.1208 - val_loss: 0.1195
    Epoch 5/5
    60000/60000 [==============================] - 62s 1ms/step - loss: 0.1205 - val_loss: 0.1195





    <keras.callbacks.History at 0x1428147d0>




```python
decoded_imgs_noised = autoencoder.predict(x_test_noise)

showimages(x_test_noise, decoded_imgs_noised)
```


![png](/uploads/Autoencoder%20with%20Keras_files/Autoencoder%20with%20Keras_33_0.png)


 效果还是非常不错的！！！
 因为上面只轮了 5 个epoch, 如果轮的次数多一些，效果应该还会再好些！


 附上一张 TensorBoard图，一上面进行比较
 ![](/uploads/CNN_Tensorboard_with_noise.png)

【参考】

- https://blog.keras.io/building-autoencoders-in-keras.html
- https://www.kesci.com/apps/home/project/5a3902de0e1fc52691fdd5cb
