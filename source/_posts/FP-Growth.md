---
title: FP-Growth
copyright: true
date: 2018-04-04 11:51:04
tags: 机器学习算法
categories: 机器学习算法
---
# FP-Growth

## FP-Growth 介绍
FP-Growth（Frequent Pattern Growth， 频繁模式增长），它比Apriori算法效率更高，在整个算法执行过程中，只需要遍历数据集2次，就可完成频繁模式的发现

FP-growth算法是伊利罗伊香槟分校的韩嘉炜教授于2004年[1]提出的，它是为了解决Apriori算法每次增加频繁项集的大小都要遍历整个数据库的缺点，特别是当数据集很大时，该算法执行速度要快于Apriori算法两个数量级。FP-growth算法的任务是将数据集存储在一个特定的称为FP树的结构之后发现频繁项集或者频繁项对，虽然它能够高效地发现频繁项集，但是不能用来发现关联规则，也就是只优化了Apriori算法两个功能中的前一个功能。

FP-growth算法基于Apriori构建，但采用了高级的数据结构减少扫描次数，大大加快了算法速度。FP-growth算法只需要对数据库进行两次扫描，而Apriori算法对于每个潜在的频繁项集都会扫描数据集判定给定模式是否频繁，因此FP-growth算法的速度要比Apriori算法快。

FP-growth算法发现频繁项集的基本过程如下：
- 构建FP树
- 从FP树中挖掘频繁项集

> FP-growth算法
  * 优点：一般要快于Apriori。
  * 缺点：实现比较困难，在某些数据集上性能会下降。
  * 适用数据类型：离散型数据。

## 示例
  [FP-Growth 算法实现及从新闻网站点击流中挖掘新闻报道 示例](https://github.com/yorkLiu/KeepReading/blob/master/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E5%AE%9E%E6%88%98Peter%E8%91%97-%E7%AC%94%E8%AE%B0/Code/FP-Growth.ipynb)

  **注意** 关于程序中用到的 **kosarak.dat** 文件，可以从我的百度盘中获取，或者其它地方获取
  [kosarak.dat 百度盘下载](https://pan.baidu.com/s/17eH1kMEV3wzsdLAiz1KVTQ)

## 总结
FP-growth算法是一种用于发现数据集中频繁模式的有效方法。FP-growth算法利用Apriori原则，执行更快。Apriori算法产生候选项集，然后扫描数据集来检查它们是否频繁。由于只对数据集扫描两次，因此FP-growth算法执行更快。在FP-growth算法中，数据集存储在一个称为FP树的结构中。FP树构建完成后，可以通过查找元素项的条件基及构建条件FP树来发现频繁项集。该过程不断以更多元素作为条件重复进行，直到FP树只包含一个元素为止。

FP-growth算法还有一个map-reduce版本的实现，它也很不错，可以扩展到多台机器上运行。Google使用该算法通过遍历大量文本来发现频繁共现词，其做法和我们刚才介绍的例子非常类似（参见扩展阅读：FP-growth算法）。

--------------
参考：
* https://blog.csdn.net/leo_xu06/article/details/51332428
* https://blog.csdn.net/daizongxue/article/details/77504158
* http://www.cnblogs.com/qwertWZ/p/4510857.html
