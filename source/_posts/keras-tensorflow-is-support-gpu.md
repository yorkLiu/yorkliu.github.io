---
title: 如何知道Keras或者是Tensorflow是否支持GPU
copyright: true
date: 2018-06-08 09:56:35
tags:
    - 机器学习
    - Keras
categories:
    - 机器学习
    - Keras
---
# GPU 加速

我们知道`Tensorflow`是有支持`GPU`的版本的，如果我们用`GPU`来加速训练模型速度将会有显著的提升!
当然`Keras`也为我们提供了非常便捷的支持`multi_gpu_model`,如以下代码:
```
from keras.utils.training_utils import multi_gpu_model

with tf.device('/cpu:0'):
            model = Xception(weights=None,
                             input_shape=(height, width, 3),
                             classes=num_classes)
# gpus=2 （表示用2个gpu来加速，前提是你的机器上要有2上显示）
multi_gpu_model(model, gpus=2)
```

有些时候运行以下代码会得到以下错误提示:
```
ValueError: To call `multi_gpu_model` with `gpus=2`, we expect the following devices to be available: ['/cpu:0', '/gpu:0', '/gpu:1']. However this machine only has: ['/cpu:0']. Try reducing `gpus`.

```
这表明，你的机器上有可能没有那么多的显示数量，还有一个重要原因是你所安装的`Tensorflow`或者`Keras`本身并没有`GPU`加速的功能.

# 如何检查`TensorFlow`或者`Keras`是否支持`GPU`加速
- *`Tensorflow`*
  ```
  from tensorflow.python.client import device_lib
  print(device_lib.list_local_devices())
  ```
  如果得到以下 输出信息，则表示你所安装的`tensorflow`版本是支持 `GPU`的。如果输出的信息中没有`gpu`的字样，则说明你安装的`tensorlfow`并不支持`GPU`加速
  [
    name: "/cpu:0"device_type: "CPU",
    name: "/gpu:0"device_type: "GPU"
  ]
- *`Keras`*
  ```
  from keras import backend as K
  print(K.tensorflow_backend._get_available_gpus())
  ```
  如果输出为 `[]`空，则表示你所安装的`Keras`版本并不支持`GPU`

# 安装`GPU`加速的`Tensorflow`与`Keras`
- *`Tensoflor`*
  ```
  $ pip install tensorflow      # Python 2.7; CPU support (no GPU support)
  $ pip3 install tensorflow     # Python 3.n; CPU support (no GPU support)
  $ pip install tensorflow-gpu  # Python 2.7;  GPU support
  $ pip3 install tensorflow-gpu # Python 3.n; GPU support
  ```
- *`Keras`*  
  `Keras`可以能过 [Anaconda](https://www.anaconda.com/download/?spm=a2c4e.11153940.blogcont236400.18.40685056izjqma#macos)来安装带`GPU`加速的`Keras`版本!
  ```
  conda install keras-gpu
  ```

# 总而言之
总而言之，言而总之，如何检查可以根据以下几个步骤:
1. 你的系统是否有`Nvidia` 的显卡驱动 (`AMD`是无法工作的)
2. 是否已经安装了`GPU`版本的`Tensorflow`和`Keras` (貌似 tensorlfow 1.2及以后的版本在 *macos* 上都不支持 `GPU`, 貌似网上以可以找到方法来破解)
3. 是否已经安装的[CUDA](https://developer.nvidia.com/cuda-zone), `CUDA`有以下几个安装包, 如果你的网络可以翻墙也可以看[Gootle Tensorflow的官方文档](https://www.tensorflow.org/install/install_linux)
    - [CUDA ToolKit安装请点这里](https://developer.nvidia.com/cuda-zone), [CUDA ToolKit](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/)
    - [cuDNN SDK](https://developer.nvidia.com/cudnn), [cuDNN SDK安装请点这里](https://docs.nvidia.com/deeplearning/sdk/cudnn-install/)
4. 根据本文提到的以上方法检查`GPu`是否可以正常的工作了！
