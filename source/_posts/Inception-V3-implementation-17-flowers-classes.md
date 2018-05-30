---
title: 使用 Google Inception V3模型进行迁移学习之——牛津大学花朵分类
copyright: true
date: 2018-05-30 11:08:01
tags:
    - 机器学习
    - Keras
categories:
    - 机器学习
    - Keras
---

# 本章知识点:

- Inception V3 模型
- 迁移学习(transform Learning & Fine Tune)
- Keras ImageDataGenerator 强化图片数据集
- 如何给图片添加标签
- 将图片数据转换成 Keras ImageDataGenerator 所需要的格式

# 目录

- [1.概述](#1.概述)
- [2.数据集处理](#2.数据集处理)
- [3.用Keras中的ImageDataGenerator来增强数据集](#3.用Keras中的ImageDataGenerator来增强数据集)
- [4.用Google Inception V3 模型做迁移学习](#4.用Google-Inception-V3-模型做迁移学习)

【参考】

- [Keras and Tensorflow中如何使用迁移学习来建立一个图像识别系统](https://deeplearningsandbox.com/how-to-use-transfer-learning-and-fine-tuning-in-keras-and-tensorflow-to-build-an-image-recognition-94b0b02444f2) 或者 [He's GitHub](https://github.com/DeepLearningSandbox/DeepLearningSandbox/tree/master/transfer_learning)
- [GlobalAveragePooling 详解](http://blog.leanote.com/post/sunalbert/Global-average-pooling)
- [GlobalAveragePooling 详解](https://blog.csdn.net/losteng/article/details/51520555)


# 1.概述

深度学习可以说是一门数据驱动的学科，各种有名的CNN模型，无一不是在大型的数据库上进行的训练。像ImageNet这种规模的数据库，动辄上百万张图片。对于普通的机器学习工作者、学习者来说，面对的任务各不相同，很难拿到如此大规模的数据集。同时也没有谷歌，Facebook那种大公司惊人的算力支持，想从0训练一个深度CNN网络，基本是不可能的。但是好在已经训练好的模型的参数，往往经过简单的调整和训练，就可以很好的迁移到其他不同的数据集上，同时也无需大量的算力支撑，便能在短时间内训练得出满意的效果。这便是迁移学习。究其根本，就是虽然图像的数据集不同，但是底层的特征却是有大部分通用的。

**迁移学习主要分为两种**:

- 第一种即所谓的transfer learning，迁移训练时，移掉最顶层，比如ImageNet训练任务的顶层就是一个1000输出的全连接层，换上新的顶层，比如输出为10的全连接层，然后训练的时候，只训练最后两层，即原网络的倒数第二层和新换的全连接输出层。可以说transfer learning将底层的网络当做了一个特征提取器来使用。
- 第二种叫做fine tune，和transfer learning一样，换一个新的顶层，但是这一次在训练的过程中，所有的（或大部分）其它层都会经过训练。也就是底层的权重也会随着训练进行调整。

一个典型的迁移学习过程是这样的。首先通过transfer learning对新的数据集进行训练，训练过一定epoch之后，改用fine tune方法继续训练，同时降低学习率。这样做是因为如果一开始就采用fine tune方法的话，网络还没有适应新的数据，那么在进行参数更新的时候，比较大的梯度可能会导致原本训练的比较好的参数被污染，反而导致效果下降。

我们将尝试使用谷歌提出的Inception V3模型来对一个花朵数据集进行迁移学习的训练。

数据集为17种不同的花朵，每种有80张样本，一共1360张图像，属于典型的小样本集。数据下载地址：http://www.robots.ox.ac.uk/~vgg/data/flowers/17/

根据原文的描述:
> **Overview**

> We have created a 17 category flower dataset with **80 images for each class**. The flowers chosen are some common flowers in the UK. The images have large scale, pose and light variations and there are also classes with large varations of images within the class and close similarity to other classes. The categories can be seen in the figure below. We randomly split the dataset into 3 different training, validation and test sets. A subset of the images have been groundtruth labelled for segmentation.

如下所示:
![](../data/images/17_flowers_categories.jpg)


# 2.数据集处理

源数据集是没有标签的，并且也都没有分类，所有的类别的图片都在同一个folder中，因此我们要将数据集进行分类，因为我们下面还要用到 Keras 中的ImageDataGenerator来增强数据集，因些还得将数据按照ImageDataGenerator的格式存储
```bash
data
  |_class0
    |image0
    |....
  |_class1
    |image0
    |...
  ....  
```  

该数据集共有17种花，分别为:

| | |
| :--- | :--- |
|0| Buttercup|
|1| Colts Foot|
|2| Daffodil|
|3| Daisy|
|4| Dandelion|
|5| Fritillary|
|6| Iris|
|7| Pansy|
|8| Sunflower|
|9| Windflower|
|10| Snowdrop|
|11|LilyValley|
|12| Bluebell|
|13| Crocus|
|14| Tigerlily|
|15| Tulip|
|16| Cowslip|

我们将数据集分成：
- train data: 800
- validation data: 260
- test data: 260


```python
# 图片下载下来存放的路径
PATH = '../data/my-dataset-images/17_flowers/'
# 将图片分类之后存储的路径
CLASSED_PATH='../data/my-dataset-images/17_flowers_classes/'
```


```python
import glob
import os
from shutil import copyfile, rmtree
import numpy as np

np.random.seed(0) #确保每次 random的顺序都是一样一样的

def create_dataset(path, target_path, phase='train', current_idx=0, d_size=100):
    """
    ::path image path
    ::target_path the new image store path
    ::phase (train, validation, test)
    ::d_size dataset size
    """
    train_size = 800
    # 17 catetories x 80 images for ecah category = 1360 pictures
    val_size = (1360-train_size) / 2
    test_size = val_size
    filenames = glob.glob(os.path.join(path, '*.jpg')) if os.path.isdir(path) else glob.glob(path)

    labels = []

    for i, item in enumerate(filenames):
        c = str(i//80)
        labels.append('%s %s' % (c, os.path.basename(item)))  

    np.random.shuffle(labels)

    if phase not in ['train', 'validation', 'test']:
        print 'phase error'
        exit()

    gen_path = os.path.join(target_path, phase)

    for i in range(current_idx, current_idx+d_size):
        item = labels[i]
        r = item.split(' ')
        img_class = r[0]
        img_name = r[1]

        target_img_floder = os.path.join(gen_path, img_class)
        target_img_name = os.path.join(target_img_floder, img_name)
        if not os.path.exists(target_img_floder):
            os.makedirs(target_img_floder)

        copyfile(os.path.join(path, img_name),target_img_name)

    return current_idx+d_size

def load_dataset(path, target_path):
    # before generating clear the target_path
    if os.path.exists(target_path):
        rmtree(target_path)

    print 'Generating training dataset 800 samples'
    train_index = create_dataset(path, target_path, phase='train', current_idx=0, d_size=800)
    print 'Generating validation dataset 260 samples'
    val_index = create_dataset(path, target_path, phase='validation', current_idx=train_index, d_size=260)
    print 'Generating test dataset 260 samples'
    test_index = create_dataset(path, target_path, phase='test', current_idx=val_index, d_size=260)
```


```python
d = load_dataset(PATH,CLASSED_PATH)
```

    Generating training dataset 800 samples
    Generating validation dataset 260 samples
    Generating test dataset 260 samples



```python
import subprocess
print subprocess.check_output(['ls', CLASSED_PATH])
```

    test
    train
    validation



运行完上面的`load_dataset(PATH,CLASSED_PATH)`方法，在`CLASSED_PATH`folder下就会生成以上数据了，这时，我们的数据已经你准备好了

# 3.用Keras中的ImageDataGenerator来增强数据集

由于该数据集中的数据并不多(相比于MNIST数据集)，总共才1360张图片(80 x 17)，对于机器学习来说，数据集越多，越全面，则训练的模型准确度会越高，因此我们使用keras中的 **ImageDataGenerator**来进行图片数据的增强学习(ImageDataGenerator 会生成更多的样本)

构建了图像生成器，用于数据扩充。数据扩充在是在训练过程中使用的一种方法，可通过对图像进行随机变换来随机改变图像。通过这些变换，网络将会持续看到增加的示例，这有助于网络更好地泛化到验证数据上，但同时也可能在训练集上表现得更差。 在多数情况下，这种权衡是值得的

更详细的参数说明[ImageDataGenerator中文说明](http://keras-cn.readthedocs.io/en/latest/preprocessing/image/)


```python
from keras.preprocessing.image import ImageDataGenerator
from keras.applications.inception_v3 import InceptionV3, preprocess_input
import warnings
warnings.filterwarnings('ignore')

train_datagen = ImageDataGenerator(
    preprocessing_function = preprocess_input, # ((x/255)-0.5)*2  归一化到±1之间
    rotation_range=30,
    width_shift_range=0.2,
    height_shift_range=0.2,
    zoom_range=0.2,
    horizontal_flip=True
)

val_datagen = ImageDataGenerator(
    preprocessing_function = preprocess_input, # ((x/255)-0.5)*2  归一化到±1之间
    rotation_range=30,
    width_shift_range=0.2,
    height_shift_range=0.2,
    zoom_range=0.2,
    horizontal_flip=True
)

```

这里用到的数据集和之前都不同，之前用的是一些公共的、Keras内置的数据集，这次用到的是自己准备的数据集。由于数据的图像大小比较大，不适合一次全部载入到内存中，所以使用了flow_from_directory方法来按批次从硬盘读取图像数据，并实时进行图像增强


```python
train_generator = train_datagen.flow_from_directory(directory=os.path.join(CLASSED_PATH, 'train'),
                                                    target_size=(299,299), # Inception V3 规定的图片大小(299 x 299)
                                                    batch_size=64)

val_generator = val_datagen.flow_from_directory(directory=os.path.join(CLASSED_PATH, 'validation'),
                                                target_size=(299, 299),
                                                batch_size=64)
```
    Found 800 images belonging to 17 classes.
    Found 260 images belonging to 17 classes.


# 4.用Google Inception V3 模型做迁移学习

首先我们需要加载骨架模型，这里用的**InceptionV3**模型，其两个参数比较重要，一个是weights，如果是`imagenet`,Keras就会自动下载已经在ImageNet上训练好的参数，如果是`None`，系统会通过随机的方式初始化参数，目前该参数只有这两个选择。另一个参数是`include_top`，如果是True，输出是1000个节点的全连接层。如果是False，会去掉顶层，输出一个8 * 8 * 2048的张量。

**ps：在keras.applications里还有很多其他的预置模型，比如VGG，ResNet，以及适用于移动端的MobileNet等。**

一般我们做迁移训练，都是要去掉顶层，后面接上各种自定义的其它新层。这已经成为了训练新任务惯用的套路。
输出层先用`GlobalAveragePooling2D`函数将8 x 8 x 2048的输出转换成1 x 2048的张量。后面接了一个1024个节点的全连接层，最后是一个17个节点的输出层，用`softmax`激活函数。


```python
from keras.applications import InceptionV3
from keras.layers import Dense, GlobalAveragePooling2D
from keras.models import Model
from keras.utils.vis_utils import plot_model

inception_v3_model = InceptionV3(weights='imagenet', include_top=False)

x = inception_v3_model.output
x= GlobalAveragePooling2D()(x) #  GlobalAveragePooling2D 将 MxNxC 的张量转换成 1xC 张量，C是通道数
x = Dense(1024, activation='relu')(x)

predictions = Dense(17, activation='softmax')(x)

model = Model(inputs=inception_v3_model.input, outputs=predictions)
# plot_model(model, 't1mode.png')
model.summary()

```
    __________________________________________________________________________________________________
    Layer (type)                    Output Shape         Param #     Connected to                     
    ==================================================================================================
    input_1 (InputLayer)            (None, None, None, 3 0                                            
    __________________________________________________________________________________________________
    conv2d_1 (Conv2D)               (None, None, None, 3 864         input_1[0][0]                    
    __________________________________________________________________________________________________
    batch_normalization_1 (BatchNor (None, None, None, 3 96          conv2d_1[0][0]                   
    __________________________________________________________________________________________________
    activation_1 (Activation)       (None, None, None, 3 0           batch_normalization_1[0][0]      
    __________________________________________________________________________________________________
    conv2d_2 (Conv2D)               (None, None, None, 3 9216        activation_1[0][0]               
    __________________________________________________________________________________________________
    batch_normalization_2 (BatchNor (None, None, None, 3 96          conv2d_2[0][0]                   
    __________________________________________________________________________________________________
    activation_2 (Activation)       (None, None, None, 3 0           batch_normalization_2[0][0]      
    __________________________________________________________________________________________________
    conv2d_3 (Conv2D)               (None, None, None, 6 18432       activation_2[0][0]               
    __________________________________________________________________________________________________
    batch_normalization_3 (BatchNor (None, None, None, 6 192         conv2d_3[0][0]                   
    __________________________________________________________________________________________________
    activation_3 (Activation)       (None, None, None, 6 0           batch_normalization_3[0][0]      
    __________________________________________________________________________________________________
    max_pooling2d_1 (MaxPooling2D)  (None, None, None, 6 0           activation_3[0][0]               
    __________________________________________________________________________________________________
    conv2d_4 (Conv2D)               (None, None, None, 8 5120        max_pooling2d_1[0][0]            
    __________________________________________________________________________________________________
    batch_normalization_4 (BatchNor (None, None, None, 8 240         conv2d_4[0][0]                   
    __________________________________________________________________________________________________
    activation_4 (Activation)       (None, None, None, 8 0           batch_normalization_4[0][0]      
    __________________________________________________________________________________________________
    conv2d_5 (Conv2D)               (None, None, None, 1 138240      activation_4[0][0]               
    __________________________________________________________________________________________________
    batch_normalization_5 (BatchNor (None, None, None, 1 576         conv2d_5[0][0]                   
    __________________________________________________________________________________________________
    activation_5 (Activation)       (None, None, None, 1 0           batch_normalization_5[0][0]      
    __________________________________________________________________________________________________
    max_pooling2d_2 (MaxPooling2D)  (None, None, None, 1 0           activation_5[0][0]               
    __________________________________________________________________________________________________
    conv2d_9 (Conv2D)               (None, None, None, 6 12288       max_pooling2d_2[0][0]            
    __________________________________________________________________________________________________
    batch_normalization_9 (BatchNor (None, None, None, 6 192         conv2d_9[0][0]                   
    __________________________________________________________________________________________________
    activation_9 (Activation)       (None, None, None, 6 0           batch_normalization_9[0][0]      
    __________________________________________________________________________________________________
    conv2d_7 (Conv2D)               (None, None, None, 4 9216        max_pooling2d_2[0][0]            
    __________________________________________________________________________________________________
    conv2d_10 (Conv2D)              (None, None, None, 9 55296       activation_9[0][0]               
    __________________________________________________________________________________________________
    batch_normalization_7 (BatchNor (None, None, None, 4 144         conv2d_7[0][0]                   
    __________________________________________________________________________________________________
    batch_normalization_10 (BatchNo (None, None, None, 9 288         conv2d_10[0][0]                  
    __________________________________________________________________________________________________
    activation_7 (Activation)       (None, None, None, 4 0           batch_normalization_7[0][0]      
    __________________________________________________________________________________________________
    activation_10 (Activation)      (None, None, None, 9 0           batch_normalization_10[0][0]     
    __________________________________________________________________________________________________
    average_pooling2d_1 (AveragePoo (None, None, None, 1 0           max_pooling2d_2[0][0]            
    __________________________________________________________________________________________________
    conv2d_6 (Conv2D)               (None, None, None, 6 12288       max_pooling2d_2[0][0]            
    __________________________________________________________________________________________________
    conv2d_8 (Conv2D)               (None, None, None, 6 76800       activation_7[0][0]               
    __________________________________________________________________________________________________
    conv2d_11 (Conv2D)              (None, None, None, 9 82944       activation_10[0][0]              
    __________________________________________________________________________________________________
    conv2d_12 (Conv2D)              (None, None, None, 3 6144        average_pooling2d_1[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_6 (BatchNor (None, None, None, 6 192         conv2d_6[0][0]                   
    __________________________________________________________________________________________________
    batch_normalization_8 (BatchNor (None, None, None, 6 192         conv2d_8[0][0]                   
    __________________________________________________________________________________________________
    batch_normalization_11 (BatchNo (None, None, None, 9 288         conv2d_11[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_12 (BatchNo (None, None, None, 3 96          conv2d_12[0][0]                  
    __________________________________________________________________________________________________
    activation_6 (Activation)       (None, None, None, 6 0           batch_normalization_6[0][0]      
    __________________________________________________________________________________________________
    activation_8 (Activation)       (None, None, None, 6 0           batch_normalization_8[0][0]      
    __________________________________________________________________________________________________
    activation_11 (Activation)      (None, None, None, 9 0           batch_normalization_11[0][0]     
    __________________________________________________________________________________________________
    activation_12 (Activation)      (None, None, None, 3 0           batch_normalization_12[0][0]     
    __________________________________________________________________________________________________
    mixed0 (Concatenate)            (None, None, None, 2 0           activation_6[0][0]               
                                                                     activation_8[0][0]               
                                                                     activation_11[0][0]              
                                                                     activation_12[0][0]              
    __________________________________________________________________________________________________
    conv2d_16 (Conv2D)              (None, None, None, 6 16384       mixed0[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_16 (BatchNo (None, None, None, 6 192         conv2d_16[0][0]                  
    __________________________________________________________________________________________________
    activation_16 (Activation)      (None, None, None, 6 0           batch_normalization_16[0][0]     
    __________________________________________________________________________________________________
    conv2d_14 (Conv2D)              (None, None, None, 4 12288       mixed0[0][0]                     
    __________________________________________________________________________________________________
    conv2d_17 (Conv2D)              (None, None, None, 9 55296       activation_16[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_14 (BatchNo (None, None, None, 4 144         conv2d_14[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_17 (BatchNo (None, None, None, 9 288         conv2d_17[0][0]                  
    __________________________________________________________________________________________________
    activation_14 (Activation)      (None, None, None, 4 0           batch_normalization_14[0][0]     
    __________________________________________________________________________________________________
    activation_17 (Activation)      (None, None, None, 9 0           batch_normalization_17[0][0]     
    __________________________________________________________________________________________________
    average_pooling2d_2 (AveragePoo (None, None, None, 2 0           mixed0[0][0]                     
    __________________________________________________________________________________________________
    conv2d_13 (Conv2D)              (None, None, None, 6 16384       mixed0[0][0]                     
    __________________________________________________________________________________________________
    conv2d_15 (Conv2D)              (None, None, None, 6 76800       activation_14[0][0]              
    __________________________________________________________________________________________________
    conv2d_18 (Conv2D)              (None, None, None, 9 82944       activation_17[0][0]              
    __________________________________________________________________________________________________
    conv2d_19 (Conv2D)              (None, None, None, 6 16384       average_pooling2d_2[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_13 (BatchNo (None, None, None, 6 192         conv2d_13[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_15 (BatchNo (None, None, None, 6 192         conv2d_15[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_18 (BatchNo (None, None, None, 9 288         conv2d_18[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_19 (BatchNo (None, None, None, 6 192         conv2d_19[0][0]                  
    __________________________________________________________________________________________________
    activation_13 (Activation)      (None, None, None, 6 0           batch_normalization_13[0][0]     
    __________________________________________________________________________________________________
    activation_15 (Activation)      (None, None, None, 6 0           batch_normalization_15[0][0]     
    __________________________________________________________________________________________________
    activation_18 (Activation)      (None, None, None, 9 0           batch_normalization_18[0][0]     
    __________________________________________________________________________________________________
    activation_19 (Activation)      (None, None, None, 6 0           batch_normalization_19[0][0]     
    __________________________________________________________________________________________________
    mixed1 (Concatenate)            (None, None, None, 2 0           activation_13[0][0]              
                                                                     activation_15[0][0]              
                                                                     activation_18[0][0]              
                                                                     activation_19[0][0]              
    __________________________________________________________________________________________________
    conv2d_23 (Conv2D)              (None, None, None, 6 18432       mixed1[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_23 (BatchNo (None, None, None, 6 192         conv2d_23[0][0]                  
    __________________________________________________________________________________________________
    activation_23 (Activation)      (None, None, None, 6 0           batch_normalization_23[0][0]     
    __________________________________________________________________________________________________
    conv2d_21 (Conv2D)              (None, None, None, 4 13824       mixed1[0][0]                     
    __________________________________________________________________________________________________
    conv2d_24 (Conv2D)              (None, None, None, 9 55296       activation_23[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_21 (BatchNo (None, None, None, 4 144         conv2d_21[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_24 (BatchNo (None, None, None, 9 288         conv2d_24[0][0]                  
    __________________________________________________________________________________________________
    activation_21 (Activation)      (None, None, None, 4 0           batch_normalization_21[0][0]     
    __________________________________________________________________________________________________
    activation_24 (Activation)      (None, None, None, 9 0           batch_normalization_24[0][0]     
    __________________________________________________________________________________________________
    average_pooling2d_3 (AveragePoo (None, None, None, 2 0           mixed1[0][0]                     
    __________________________________________________________________________________________________
    conv2d_20 (Conv2D)              (None, None, None, 6 18432       mixed1[0][0]                     
    __________________________________________________________________________________________________
    conv2d_22 (Conv2D)              (None, None, None, 6 76800       activation_21[0][0]              
    __________________________________________________________________________________________________
    conv2d_25 (Conv2D)              (None, None, None, 9 82944       activation_24[0][0]              
    __________________________________________________________________________________________________
    conv2d_26 (Conv2D)              (None, None, None, 6 18432       average_pooling2d_3[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_20 (BatchNo (None, None, None, 6 192         conv2d_20[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_22 (BatchNo (None, None, None, 6 192         conv2d_22[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_25 (BatchNo (None, None, None, 9 288         conv2d_25[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_26 (BatchNo (None, None, None, 6 192         conv2d_26[0][0]                  
    __________________________________________________________________________________________________
    activation_20 (Activation)      (None, None, None, 6 0           batch_normalization_20[0][0]     
    __________________________________________________________________________________________________
    activation_22 (Activation)      (None, None, None, 6 0           batch_normalization_22[0][0]     
    __________________________________________________________________________________________________
    activation_25 (Activation)      (None, None, None, 9 0           batch_normalization_25[0][0]     
    __________________________________________________________________________________________________
    activation_26 (Activation)      (None, None, None, 6 0           batch_normalization_26[0][0]     
    __________________________________________________________________________________________________
    mixed2 (Concatenate)            (None, None, None, 2 0           activation_20[0][0]              
                                                                     activation_22[0][0]              
                                                                     activation_25[0][0]              
                                                                     activation_26[0][0]              
    __________________________________________________________________________________________________
    conv2d_28 (Conv2D)              (None, None, None, 6 18432       mixed2[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_28 (BatchNo (None, None, None, 6 192         conv2d_28[0][0]                  
    __________________________________________________________________________________________________
    activation_28 (Activation)      (None, None, None, 6 0           batch_normalization_28[0][0]     
    __________________________________________________________________________________________________
    conv2d_29 (Conv2D)              (None, None, None, 9 55296       activation_28[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_29 (BatchNo (None, None, None, 9 288         conv2d_29[0][0]                  
    __________________________________________________________________________________________________
    activation_29 (Activation)      (None, None, None, 9 0           batch_normalization_29[0][0]     
    __________________________________________________________________________________________________
    conv2d_27 (Conv2D)              (None, None, None, 3 995328      mixed2[0][0]                     
    __________________________________________________________________________________________________
    conv2d_30 (Conv2D)              (None, None, None, 9 82944       activation_29[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_27 (BatchNo (None, None, None, 3 1152        conv2d_27[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_30 (BatchNo (None, None, None, 9 288         conv2d_30[0][0]                  
    __________________________________________________________________________________________________
    activation_27 (Activation)      (None, None, None, 3 0           batch_normalization_27[0][0]     
    __________________________________________________________________________________________________
    activation_30 (Activation)      (None, None, None, 9 0           batch_normalization_30[0][0]     
    __________________________________________________________________________________________________
    max_pooling2d_3 (MaxPooling2D)  (None, None, None, 2 0           mixed2[0][0]                     
    __________________________________________________________________________________________________
    mixed3 (Concatenate)            (None, None, None, 7 0           activation_27[0][0]              
                                                                     activation_30[0][0]              
                                                                     max_pooling2d_3[0][0]            
    __________________________________________________________________________________________________
    conv2d_35 (Conv2D)              (None, None, None, 1 98304       mixed3[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_35 (BatchNo (None, None, None, 1 384         conv2d_35[0][0]                  
    __________________________________________________________________________________________________
    activation_35 (Activation)      (None, None, None, 1 0           batch_normalization_35[0][0]     
    __________________________________________________________________________________________________
    conv2d_36 (Conv2D)              (None, None, None, 1 114688      activation_35[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_36 (BatchNo (None, None, None, 1 384         conv2d_36[0][0]                  
    __________________________________________________________________________________________________
    activation_36 (Activation)      (None, None, None, 1 0           batch_normalization_36[0][0]     
    __________________________________________________________________________________________________
    conv2d_32 (Conv2D)              (None, None, None, 1 98304       mixed3[0][0]                     
    __________________________________________________________________________________________________
    conv2d_37 (Conv2D)              (None, None, None, 1 114688      activation_36[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_32 (BatchNo (None, None, None, 1 384         conv2d_32[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_37 (BatchNo (None, None, None, 1 384         conv2d_37[0][0]                  
    __________________________________________________________________________________________________
    activation_32 (Activation)      (None, None, None, 1 0           batch_normalization_32[0][0]     
    __________________________________________________________________________________________________
    activation_37 (Activation)      (None, None, None, 1 0           batch_normalization_37[0][0]     
    __________________________________________________________________________________________________
    conv2d_33 (Conv2D)              (None, None, None, 1 114688      activation_32[0][0]              
    __________________________________________________________________________________________________
    conv2d_38 (Conv2D)              (None, None, None, 1 114688      activation_37[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_33 (BatchNo (None, None, None, 1 384         conv2d_33[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_38 (BatchNo (None, None, None, 1 384         conv2d_38[0][0]                  
    __________________________________________________________________________________________________
    activation_33 (Activation)      (None, None, None, 1 0           batch_normalization_33[0][0]     
    __________________________________________________________________________________________________
    activation_38 (Activation)      (None, None, None, 1 0           batch_normalization_38[0][0]     
    __________________________________________________________________________________________________
    average_pooling2d_4 (AveragePoo (None, None, None, 7 0           mixed3[0][0]                     
    __________________________________________________________________________________________________
    conv2d_31 (Conv2D)              (None, None, None, 1 147456      mixed3[0][0]                     
    __________________________________________________________________________________________________
    conv2d_34 (Conv2D)              (None, None, None, 1 172032      activation_33[0][0]              
    __________________________________________________________________________________________________
    conv2d_39 (Conv2D)              (None, None, None, 1 172032      activation_38[0][0]              
    __________________________________________________________________________________________________
    conv2d_40 (Conv2D)              (None, None, None, 1 147456      average_pooling2d_4[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_31 (BatchNo (None, None, None, 1 576         conv2d_31[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_34 (BatchNo (None, None, None, 1 576         conv2d_34[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_39 (BatchNo (None, None, None, 1 576         conv2d_39[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_40 (BatchNo (None, None, None, 1 576         conv2d_40[0][0]                  
    __________________________________________________________________________________________________
    activation_31 (Activation)      (None, None, None, 1 0           batch_normalization_31[0][0]     
    __________________________________________________________________________________________________
    activation_34 (Activation)      (None, None, None, 1 0           batch_normalization_34[0][0]     
    __________________________________________________________________________________________________
    activation_39 (Activation)      (None, None, None, 1 0           batch_normalization_39[0][0]     
    __________________________________________________________________________________________________
    activation_40 (Activation)      (None, None, None, 1 0           batch_normalization_40[0][0]     
    __________________________________________________________________________________________________
    mixed4 (Concatenate)            (None, None, None, 7 0           activation_31[0][0]              
                                                                     activation_34[0][0]              
                                                                     activation_39[0][0]              
                                                                     activation_40[0][0]              
    __________________________________________________________________________________________________
    conv2d_45 (Conv2D)              (None, None, None, 1 122880      mixed4[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_45 (BatchNo (None, None, None, 1 480         conv2d_45[0][0]                  
    __________________________________________________________________________________________________
    activation_45 (Activation)      (None, None, None, 1 0           batch_normalization_45[0][0]     
    __________________________________________________________________________________________________
    conv2d_46 (Conv2D)              (None, None, None, 1 179200      activation_45[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_46 (BatchNo (None, None, None, 1 480         conv2d_46[0][0]                  
    __________________________________________________________________________________________________
    activation_46 (Activation)      (None, None, None, 1 0           batch_normalization_46[0][0]     
    __________________________________________________________________________________________________
    conv2d_42 (Conv2D)              (None, None, None, 1 122880      mixed4[0][0]                     
    __________________________________________________________________________________________________
    conv2d_47 (Conv2D)              (None, None, None, 1 179200      activation_46[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_42 (BatchNo (None, None, None, 1 480         conv2d_42[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_47 (BatchNo (None, None, None, 1 480         conv2d_47[0][0]                  
    __________________________________________________________________________________________________
    activation_42 (Activation)      (None, None, None, 1 0           batch_normalization_42[0][0]     
    __________________________________________________________________________________________________
    activation_47 (Activation)      (None, None, None, 1 0           batch_normalization_47[0][0]     
    __________________________________________________________________________________________________
    conv2d_43 (Conv2D)              (None, None, None, 1 179200      activation_42[0][0]              
    __________________________________________________________________________________________________
    conv2d_48 (Conv2D)              (None, None, None, 1 179200      activation_47[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_43 (BatchNo (None, None, None, 1 480         conv2d_43[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_48 (BatchNo (None, None, None, 1 480         conv2d_48[0][0]                  
    __________________________________________________________________________________________________
    activation_43 (Activation)      (None, None, None, 1 0           batch_normalization_43[0][0]     
    __________________________________________________________________________________________________
    activation_48 (Activation)      (None, None, None, 1 0           batch_normalization_48[0][0]     
    __________________________________________________________________________________________________
    average_pooling2d_5 (AveragePoo (None, None, None, 7 0           mixed4[0][0]                     
    __________________________________________________________________________________________________
    conv2d_41 (Conv2D)              (None, None, None, 1 147456      mixed4[0][0]                     
    __________________________________________________________________________________________________
    conv2d_44 (Conv2D)              (None, None, None, 1 215040      activation_43[0][0]              
    __________________________________________________________________________________________________
    conv2d_49 (Conv2D)              (None, None, None, 1 215040      activation_48[0][0]              
    __________________________________________________________________________________________________
    conv2d_50 (Conv2D)              (None, None, None, 1 147456      average_pooling2d_5[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_41 (BatchNo (None, None, None, 1 576         conv2d_41[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_44 (BatchNo (None, None, None, 1 576         conv2d_44[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_49 (BatchNo (None, None, None, 1 576         conv2d_49[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_50 (BatchNo (None, None, None, 1 576         conv2d_50[0][0]                  
    __________________________________________________________________________________________________
    activation_41 (Activation)      (None, None, None, 1 0           batch_normalization_41[0][0]     
    __________________________________________________________________________________________________
    activation_44 (Activation)      (None, None, None, 1 0           batch_normalization_44[0][0]     
    __________________________________________________________________________________________________
    activation_49 (Activation)      (None, None, None, 1 0           batch_normalization_49[0][0]     
    __________________________________________________________________________________________________
    activation_50 (Activation)      (None, None, None, 1 0           batch_normalization_50[0][0]     
    __________________________________________________________________________________________________
    mixed5 (Concatenate)            (None, None, None, 7 0           activation_41[0][0]              
                                                                     activation_44[0][0]              
                                                                     activation_49[0][0]              
                                                                     activation_50[0][0]              
    __________________________________________________________________________________________________
    conv2d_55 (Conv2D)              (None, None, None, 1 122880      mixed5[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_55 (BatchNo (None, None, None, 1 480         conv2d_55[0][0]                  
    __________________________________________________________________________________________________
    activation_55 (Activation)      (None, None, None, 1 0           batch_normalization_55[0][0]     
    __________________________________________________________________________________________________
    conv2d_56 (Conv2D)              (None, None, None, 1 179200      activation_55[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_56 (BatchNo (None, None, None, 1 480         conv2d_56[0][0]                  
    __________________________________________________________________________________________________
    activation_56 (Activation)      (None, None, None, 1 0           batch_normalization_56[0][0]     
    __________________________________________________________________________________________________
    conv2d_52 (Conv2D)              (None, None, None, 1 122880      mixed5[0][0]                     
    __________________________________________________________________________________________________
    conv2d_57 (Conv2D)              (None, None, None, 1 179200      activation_56[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_52 (BatchNo (None, None, None, 1 480         conv2d_52[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_57 (BatchNo (None, None, None, 1 480         conv2d_57[0][0]                  
    __________________________________________________________________________________________________
    activation_52 (Activation)      (None, None, None, 1 0           batch_normalization_52[0][0]     
    __________________________________________________________________________________________________
    activation_57 (Activation)      (None, None, None, 1 0           batch_normalization_57[0][0]     
    __________________________________________________________________________________________________
    conv2d_53 (Conv2D)              (None, None, None, 1 179200      activation_52[0][0]              
    __________________________________________________________________________________________________
    conv2d_58 (Conv2D)              (None, None, None, 1 179200      activation_57[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_53 (BatchNo (None, None, None, 1 480         conv2d_53[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_58 (BatchNo (None, None, None, 1 480         conv2d_58[0][0]                  
    __________________________________________________________________________________________________
    activation_53 (Activation)      (None, None, None, 1 0           batch_normalization_53[0][0]     
    __________________________________________________________________________________________________
    activation_58 (Activation)      (None, None, None, 1 0           batch_normalization_58[0][0]     
    __________________________________________________________________________________________________
    average_pooling2d_6 (AveragePoo (None, None, None, 7 0           mixed5[0][0]                     
    __________________________________________________________________________________________________
    conv2d_51 (Conv2D)              (None, None, None, 1 147456      mixed5[0][0]                     
    __________________________________________________________________________________________________
    conv2d_54 (Conv2D)              (None, None, None, 1 215040      activation_53[0][0]              
    __________________________________________________________________________________________________
    conv2d_59 (Conv2D)              (None, None, None, 1 215040      activation_58[0][0]              
    __________________________________________________________________________________________________
    conv2d_60 (Conv2D)              (None, None, None, 1 147456      average_pooling2d_6[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_51 (BatchNo (None, None, None, 1 576         conv2d_51[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_54 (BatchNo (None, None, None, 1 576         conv2d_54[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_59 (BatchNo (None, None, None, 1 576         conv2d_59[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_60 (BatchNo (None, None, None, 1 576         conv2d_60[0][0]                  
    __________________________________________________________________________________________________
    activation_51 (Activation)      (None, None, None, 1 0           batch_normalization_51[0][0]     
    __________________________________________________________________________________________________
    activation_54 (Activation)      (None, None, None, 1 0           batch_normalization_54[0][0]     
    __________________________________________________________________________________________________
    activation_59 (Activation)      (None, None, None, 1 0           batch_normalization_59[0][0]     
    __________________________________________________________________________________________________
    activation_60 (Activation)      (None, None, None, 1 0           batch_normalization_60[0][0]     
    __________________________________________________________________________________________________
    mixed6 (Concatenate)            (None, None, None, 7 0           activation_51[0][0]              
                                                                     activation_54[0][0]              
                                                                     activation_59[0][0]              
                                                                     activation_60[0][0]              
    __________________________________________________________________________________________________
    conv2d_65 (Conv2D)              (None, None, None, 1 147456      mixed6[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_65 (BatchNo (None, None, None, 1 576         conv2d_65[0][0]                  
    __________________________________________________________________________________________________
    activation_65 (Activation)      (None, None, None, 1 0           batch_normalization_65[0][0]     
    __________________________________________________________________________________________________
    conv2d_66 (Conv2D)              (None, None, None, 1 258048      activation_65[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_66 (BatchNo (None, None, None, 1 576         conv2d_66[0][0]                  
    __________________________________________________________________________________________________
    activation_66 (Activation)      (None, None, None, 1 0           batch_normalization_66[0][0]     
    __________________________________________________________________________________________________
    conv2d_62 (Conv2D)              (None, None, None, 1 147456      mixed6[0][0]                     
    __________________________________________________________________________________________________
    conv2d_67 (Conv2D)              (None, None, None, 1 258048      activation_66[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_62 (BatchNo (None, None, None, 1 576         conv2d_62[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_67 (BatchNo (None, None, None, 1 576         conv2d_67[0][0]                  
    __________________________________________________________________________________________________
    activation_62 (Activation)      (None, None, None, 1 0           batch_normalization_62[0][0]     
    __________________________________________________________________________________________________
    activation_67 (Activation)      (None, None, None, 1 0           batch_normalization_67[0][0]     
    __________________________________________________________________________________________________
    conv2d_63 (Conv2D)              (None, None, None, 1 258048      activation_62[0][0]              
    __________________________________________________________________________________________________
    conv2d_68 (Conv2D)              (None, None, None, 1 258048      activation_67[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_63 (BatchNo (None, None, None, 1 576         conv2d_63[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_68 (BatchNo (None, None, None, 1 576         conv2d_68[0][0]                  
    __________________________________________________________________________________________________
    activation_63 (Activation)      (None, None, None, 1 0           batch_normalization_63[0][0]     
    __________________________________________________________________________________________________
    activation_68 (Activation)      (None, None, None, 1 0           batch_normalization_68[0][0]     
    __________________________________________________________________________________________________
    average_pooling2d_7 (AveragePoo (None, None, None, 7 0           mixed6[0][0]                     
    __________________________________________________________________________________________________
    conv2d_61 (Conv2D)              (None, None, None, 1 147456      mixed6[0][0]                     
    __________________________________________________________________________________________________
    conv2d_64 (Conv2D)              (None, None, None, 1 258048      activation_63[0][0]              
    __________________________________________________________________________________________________
    conv2d_69 (Conv2D)              (None, None, None, 1 258048      activation_68[0][0]              
    __________________________________________________________________________________________________
    conv2d_70 (Conv2D)              (None, None, None, 1 147456      average_pooling2d_7[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_61 (BatchNo (None, None, None, 1 576         conv2d_61[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_64 (BatchNo (None, None, None, 1 576         conv2d_64[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_69 (BatchNo (None, None, None, 1 576         conv2d_69[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_70 (BatchNo (None, None, None, 1 576         conv2d_70[0][0]                  
    __________________________________________________________________________________________________
    activation_61 (Activation)      (None, None, None, 1 0           batch_normalization_61[0][0]     
    __________________________________________________________________________________________________
    activation_64 (Activation)      (None, None, None, 1 0           batch_normalization_64[0][0]     
    __________________________________________________________________________________________________
    activation_69 (Activation)      (None, None, None, 1 0           batch_normalization_69[0][0]     
    __________________________________________________________________________________________________
    activation_70 (Activation)      (None, None, None, 1 0           batch_normalization_70[0][0]     
    __________________________________________________________________________________________________
    mixed7 (Concatenate)            (None, None, None, 7 0           activation_61[0][0]              
                                                                     activation_64[0][0]              
                                                                     activation_69[0][0]              
                                                                     activation_70[0][0]              
    __________________________________________________________________________________________________
    conv2d_73 (Conv2D)              (None, None, None, 1 147456      mixed7[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_73 (BatchNo (None, None, None, 1 576         conv2d_73[0][0]                  
    __________________________________________________________________________________________________
    activation_73 (Activation)      (None, None, None, 1 0           batch_normalization_73[0][0]     
    __________________________________________________________________________________________________
    conv2d_74 (Conv2D)              (None, None, None, 1 258048      activation_73[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_74 (BatchNo (None, None, None, 1 576         conv2d_74[0][0]                  
    __________________________________________________________________________________________________
    activation_74 (Activation)      (None, None, None, 1 0           batch_normalization_74[0][0]     
    __________________________________________________________________________________________________
    conv2d_71 (Conv2D)              (None, None, None, 1 147456      mixed7[0][0]                     
    __________________________________________________________________________________________________
    conv2d_75 (Conv2D)              (None, None, None, 1 258048      activation_74[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_71 (BatchNo (None, None, None, 1 576         conv2d_71[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_75 (BatchNo (None, None, None, 1 576         conv2d_75[0][0]                  
    __________________________________________________________________________________________________
    activation_71 (Activation)      (None, None, None, 1 0           batch_normalization_71[0][0]     
    __________________________________________________________________________________________________
    activation_75 (Activation)      (None, None, None, 1 0           batch_normalization_75[0][0]     
    __________________________________________________________________________________________________
    conv2d_72 (Conv2D)              (None, None, None, 3 552960      activation_71[0][0]              
    __________________________________________________________________________________________________
    conv2d_76 (Conv2D)              (None, None, None, 1 331776      activation_75[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_72 (BatchNo (None, None, None, 3 960         conv2d_72[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_76 (BatchNo (None, None, None, 1 576         conv2d_76[0][0]                  
    __________________________________________________________________________________________________
    activation_72 (Activation)      (None, None, None, 3 0           batch_normalization_72[0][0]     
    __________________________________________________________________________________________________
    activation_76 (Activation)      (None, None, None, 1 0           batch_normalization_76[0][0]     
    __________________________________________________________________________________________________
    max_pooling2d_4 (MaxPooling2D)  (None, None, None, 7 0           mixed7[0][0]                     
    __________________________________________________________________________________________________
    mixed8 (Concatenate)            (None, None, None, 1 0           activation_72[0][0]              
                                                                     activation_76[0][0]              
                                                                     max_pooling2d_4[0][0]            
    __________________________________________________________________________________________________
    conv2d_81 (Conv2D)              (None, None, None, 4 573440      mixed8[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_81 (BatchNo (None, None, None, 4 1344        conv2d_81[0][0]                  
    __________________________________________________________________________________________________
    activation_81 (Activation)      (None, None, None, 4 0           batch_normalization_81[0][0]     
    __________________________________________________________________________________________________
    conv2d_78 (Conv2D)              (None, None, None, 3 491520      mixed8[0][0]                     
    __________________________________________________________________________________________________
    conv2d_82 (Conv2D)              (None, None, None, 3 1548288     activation_81[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_78 (BatchNo (None, None, None, 3 1152        conv2d_78[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_82 (BatchNo (None, None, None, 3 1152        conv2d_82[0][0]                  
    __________________________________________________________________________________________________
    activation_78 (Activation)      (None, None, None, 3 0           batch_normalization_78[0][0]     
    __________________________________________________________________________________________________
    activation_82 (Activation)      (None, None, None, 3 0           batch_normalization_82[0][0]     
    __________________________________________________________________________________________________
    conv2d_79 (Conv2D)              (None, None, None, 3 442368      activation_78[0][0]              
    __________________________________________________________________________________________________
    conv2d_80 (Conv2D)              (None, None, None, 3 442368      activation_78[0][0]              
    __________________________________________________________________________________________________
    conv2d_83 (Conv2D)              (None, None, None, 3 442368      activation_82[0][0]              
    __________________________________________________________________________________________________
    conv2d_84 (Conv2D)              (None, None, None, 3 442368      activation_82[0][0]              
    __________________________________________________________________________________________________
    average_pooling2d_8 (AveragePoo (None, None, None, 1 0           mixed8[0][0]                     
    __________________________________________________________________________________________________
    conv2d_77 (Conv2D)              (None, None, None, 3 409600      mixed8[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_79 (BatchNo (None, None, None, 3 1152        conv2d_79[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_80 (BatchNo (None, None, None, 3 1152        conv2d_80[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_83 (BatchNo (None, None, None, 3 1152        conv2d_83[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_84 (BatchNo (None, None, None, 3 1152        conv2d_84[0][0]                  
    __________________________________________________________________________________________________
    conv2d_85 (Conv2D)              (None, None, None, 1 245760      average_pooling2d_8[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_77 (BatchNo (None, None, None, 3 960         conv2d_77[0][0]                  
    __________________________________________________________________________________________________
    activation_79 (Activation)      (None, None, None, 3 0           batch_normalization_79[0][0]     
    __________________________________________________________________________________________________
    activation_80 (Activation)      (None, None, None, 3 0           batch_normalization_80[0][0]     
    __________________________________________________________________________________________________
    activation_83 (Activation)      (None, None, None, 3 0           batch_normalization_83[0][0]     
    __________________________________________________________________________________________________
    activation_84 (Activation)      (None, None, None, 3 0           batch_normalization_84[0][0]     
    __________________________________________________________________________________________________
    batch_normalization_85 (BatchNo (None, None, None, 1 576         conv2d_85[0][0]                  
    __________________________________________________________________________________________________
    activation_77 (Activation)      (None, None, None, 3 0           batch_normalization_77[0][0]     
    __________________________________________________________________________________________________
    mixed9_0 (Concatenate)          (None, None, None, 7 0           activation_79[0][0]              
                                                                     activation_80[0][0]              
    __________________________________________________________________________________________________
    concatenate_1 (Concatenate)     (None, None, None, 7 0           activation_83[0][0]              
                                                                     activation_84[0][0]              
    __________________________________________________________________________________________________
    activation_85 (Activation)      (None, None, None, 1 0           batch_normalization_85[0][0]     
    __________________________________________________________________________________________________
    mixed9 (Concatenate)            (None, None, None, 2 0           activation_77[0][0]              
                                                                     mixed9_0[0][0]                   
                                                                     concatenate_1[0][0]              
                                                                     activation_85[0][0]              
    __________________________________________________________________________________________________
    conv2d_90 (Conv2D)              (None, None, None, 4 917504      mixed9[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_90 (BatchNo (None, None, None, 4 1344        conv2d_90[0][0]                  
    __________________________________________________________________________________________________
    activation_90 (Activation)      (None, None, None, 4 0           batch_normalization_90[0][0]     
    __________________________________________________________________________________________________
    conv2d_87 (Conv2D)              (None, None, None, 3 786432      mixed9[0][0]                     
    __________________________________________________________________________________________________
    conv2d_91 (Conv2D)              (None, None, None, 3 1548288     activation_90[0][0]              
    __________________________________________________________________________________________________
    batch_normalization_87 (BatchNo (None, None, None, 3 1152        conv2d_87[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_91 (BatchNo (None, None, None, 3 1152        conv2d_91[0][0]                  
    __________________________________________________________________________________________________
    activation_87 (Activation)      (None, None, None, 3 0           batch_normalization_87[0][0]     
    __________________________________________________________________________________________________
    activation_91 (Activation)      (None, None, None, 3 0           batch_normalization_91[0][0]     
    __________________________________________________________________________________________________
    conv2d_88 (Conv2D)              (None, None, None, 3 442368      activation_87[0][0]              
    __________________________________________________________________________________________________
    conv2d_89 (Conv2D)              (None, None, None, 3 442368      activation_87[0][0]              
    __________________________________________________________________________________________________
    conv2d_92 (Conv2D)              (None, None, None, 3 442368      activation_91[0][0]              
    __________________________________________________________________________________________________
    conv2d_93 (Conv2D)              (None, None, None, 3 442368      activation_91[0][0]              
    __________________________________________________________________________________________________
    average_pooling2d_9 (AveragePoo (None, None, None, 2 0           mixed9[0][0]                     
    __________________________________________________________________________________________________
    conv2d_86 (Conv2D)              (None, None, None, 3 655360      mixed9[0][0]                     
    __________________________________________________________________________________________________
    batch_normalization_88 (BatchNo (None, None, None, 3 1152        conv2d_88[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_89 (BatchNo (None, None, None, 3 1152        conv2d_89[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_92 (BatchNo (None, None, None, 3 1152        conv2d_92[0][0]                  
    __________________________________________________________________________________________________
    batch_normalization_93 (BatchNo (None, None, None, 3 1152        conv2d_93[0][0]                  
    __________________________________________________________________________________________________
    conv2d_94 (Conv2D)              (None, None, None, 1 393216      average_pooling2d_9[0][0]        
    __________________________________________________________________________________________________
    batch_normalization_86 (BatchNo (None, None, None, 3 960         conv2d_86[0][0]                  
    __________________________________________________________________________________________________
    activation_88 (Activation)      (None, None, None, 3 0           batch_normalization_88[0][0]     
    __________________________________________________________________________________________________
    activation_89 (Activation)      (None, None, None, 3 0           batch_normalization_89[0][0]     
    __________________________________________________________________________________________________
    activation_92 (Activation)      (None, None, None, 3 0           batch_normalization_92[0][0]     
    __________________________________________________________________________________________________
    activation_93 (Activation)      (None, None, None, 3 0           batch_normalization_93[0][0]     
    __________________________________________________________________________________________________
    batch_normalization_94 (BatchNo (None, None, None, 1 576         conv2d_94[0][0]                  
    __________________________________________________________________________________________________
    activation_86 (Activation)      (None, None, None, 3 0           batch_normalization_86[0][0]     
    __________________________________________________________________________________________________
    mixed9_1 (Concatenate)          (None, None, None, 7 0           activation_88[0][0]              
                                                                     activation_89[0][0]              
    __________________________________________________________________________________________________
    concatenate_2 (Concatenate)     (None, None, None, 7 0           activation_92[0][0]              
                                                                     activation_93[0][0]              
    __________________________________________________________________________________________________
    activation_94 (Activation)      (None, None, None, 1 0           batch_normalization_94[0][0]     
    __________________________________________________________________________________________________
    mixed10 (Concatenate)           (None, None, None, 2 0           activation_86[0][0]              
                                                                     mixed9_1[0][0]                   
                                                                     concatenate_2[0][0]              
                                                                     activation_94[0][0]              
    __________________________________________________________________________________________________
    global_average_pooling2d_1 (Glo (None, 2048)         0           mixed10[0][0]                    
    __________________________________________________________________________________________________
    dense_1 (Dense)                 (None, 1024)         2098176     global_average_pooling2d_1[0][0]
    __________________________________________________________________________________________________
    dense_2 (Dense)                 (None, 17)           17425       dense_1[0][0]                    
    ==================================================================================================
    Total params: 23,918,385
    Trainable params: 23,883,953
    Non-trainable params: 34,432
    __________________________________________________________________________________________________


可以调用
```python
from keras.utils.vis_utils import plot_model
plot_model(model, '17_flowers_classes_with_inception_v3_model.png')
```
将 model的图画出来，可以直观的看一下是怎么往下走的！(还是蛮多层，蛮复杂的!)

构建完新模型后需要进行模型的配置。下面的两个函数分别对transfer learning和fine tune两种方法分别进行了配置。每个函数有两个参数，分别是model和base_model。这里可能会有同学有疑问，上面定义了model，这里又将base_model一起做配置，对base_model的更改会对model产生影响么？
答案是会的。如果你debug追进去看的话，可以看到model的第一层和base_model的第一层是指向同一个内存地址的。这里将base_model作为参数，只是为了方便对骨架模型进行设置。

- setup_to_transfer_learning： 这个函数将骨架模型的所有层都设置为不可训练
- setup_to_fine_tune：这个函数将骨架模型中的前几层设置为不可训练，后面的所有Inception模块都设置为可训练。 这里面的GAP_LAYER需要配合打印图和调试的方法确认正确的值。


```python
# import tensorflow as tf
# from keras.applications import Xception
# from keras.utils import multi_gpu_model
# G =1

# with tf.device('/cpu:0'):
#     model = Xception(weights=None,
#                      input_shape=(299, 299, 3),
#                      classes=800)
# multi_gpu_model(model, gpus=2)

import os
os.environ["CUDA_DEVICE_ORDER"]="PCI_BUS_ID"
os.environ["CUDA_VISIBLE_DEVICES"]="1" #model will be trained on GPU 1
```


```python
from keras.optimizers import Adam, RMSprop, SGD

def setup_to_transfer_learning(model, base_model):
    """
    In this method set base_model each layer not trainable.
    """
    for layer in base_model.layers:
        layer.trainable = False

    model.compile(optimizer='sgd', loss='categorical_crossentropy', metrics=['accuracy'])

def setup_to_fine_tune(model, base_model):
    GAP_LAYER=17
    for layer in base_model.layers[:GAP_LAYER+1]:
        layer.trainable = True

    for layer in base_model.layers[GAP_LAYER+1:]:
        layer.trainable = True

    model.compile(optimizer=Adam(lr=0.0001), loss='categorical_crossentropy', metrics=['accuracy'])

```

下面开始训练，这段代码也演示了如何在全部训练过程中改变模型。


```python
from datetime import datetime
start = datetime.now()

batch_size=64

setup_to_transfer_learning(model, inception_v3_model)


transfer_learning_m = model.fit_generator(generator=train_generator,
                                          epochs=2,
                                          steps_per_epoch=800//batch_size, #train dataset has 800 samples
                                          validation_data=val_generator,
                                          validation_steps=260//batch_size,
                                          class_weight='auto',
                                          workers=8)

model.save('../data/models/17_flowers_iv3_transfer_learning_model.h5')
end = datetime.now()
print 'Traning the Transfer Learning total spend:', (end - start)
```

    Epoch 1/2
    12/12 [==============================] - 176s 15s/step - loss: 2.8624 - acc: 0.0898 - val_loss: 2.7887 - val_acc: 0.0898
    Epoch 2/2
    12/12 [==============================] - 182s 15s/step - loss: 2.6725 - acc: 0.1497 - val_loss: 2.6705 - val_acc: 0.1875
    Traning the Transfer Learning total spend:


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-14-c0321365b541> in <module>()
         17 model.save('../data/models/17_flowers_iv3_transfer_learning_model.h5')
         18
    ---> 19 print 'Traning the Transfer Learning total spend:', (end - start)


    NameError: name 'end' is not defined



```python
start = datetime.now()
setup_to_fine_tune(model, inception_v3_model)
fine_tune_m = model.fit_generator(generator=train_generator,
                                          epochs=2,
                                          steps_per_epoch=800//batch_size, #train dataset has 800 samples
                                          validation_data=val_generator,
                                          validation_steps=1,
                                          class_weight='auto')

model.save('../data/models/17_flowers_iv3_fine_tune_model.h5')
end = datetime.now()

print 'Traning the Fine Tune model:', (end - start)
```

    Epoch 1/2
    12/12 [==============================] - 505s 42s/step - loss: 1.7389 - acc: 0.6230 - val_loss: 0.7229 - val_acc: 0.9375
    Epoch 2/2
    12/12 [==============================] - 531s 44s/step - loss: 0.3801 - acc: 0.9388 - val_loss: 0.1730 - val_acc: 0.9688
     Traning the Fine Tune model: 0:17:27.002911


经过 "Fine Tune" training 之后效果还是不错了的，因为我减少了training 的samples(800 个samples的确太慢了，需要太多的时间), Validation Accuracy 既然达到了 96.8%, 实为惊人!

【**总结**】
1. 了解了Inception V3模型
2. 知道并理解了 Transfer Learning & Fine Tune 及两者的区别，优缺点
3. 知道并实现了Kera中的ImageDataGenerator 数据格式
4. 对文中提到的 GAP不是很了解

-----
【参考】

- [Keras and Tensorflow中如何使用迁移学习来建立一个图像识别系统](https://deeplearningsandbox.com/how-to-use-transfer-learning-and-fine-tuning-in-keras-and-tensorflow-to-build-an-image-recognition-94b0b02444f2) 或者 [He's GitHub](https://github.com/DeepLearningSandbox/DeepLearningSandbox/tree/master/transfer_learning)
- [GlobalAveragePooling 详解](http://blog.leanote.com/post/sunalbert/Global-average-pooling)
- [GlobalAveragePooling 详解](https://blog.csdn.net/losteng/article/details/51520555)
