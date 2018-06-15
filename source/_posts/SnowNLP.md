---
title: SnowNLP 中文分词, 情感分析之利器
copyright: true
date: 2018-06-14 12:48:51
tags: 机器学习
categories:
  - 机器学习
  - 情感分析
---
# SnowNLP 简介
[SnowNLP](https://pypi.org/project/snownlp/)是一个python写的类库，可以方便的处理中文文本内容。这是今天在邮件列表中看到的，感觉还挺有意思，象：转换成拼音，繁体转简体，提取文本关键词（Textrank算法），提取文本摘要（Textrank算法）好象在一些场合挺有用

> 注: 这个库是国人自己开发的python类库，专门针对中文文本进行挖掘，里面已经有了算法，需要自己调用函数，根据不同的文本构建语料库就可以，真的太方便了

# SnowNLP 主要功能
- 中文分词（Character-Based Generative Model）
- 词性标注（TnT 3-gram 隐马）
- 情感分析（现在训练数据主要是买卖东西时的评价，可以用自己的语料库来替换原有的，再训练，效果还是不错的）
- 文本分类（Naive Bayes）
- 转换成拼音（Trie树实现的最大匹配）
- 繁体转简体（Trie树实现的最大匹配）
- 提取文本关键词（TextRank算法）
- 提取文本摘要（TextRank算法）
- TF-IDF
- Tokenization（分割成句子）
- 文本相似（BM25）
- 支持python3（感谢erning）

*更多请参考:*
- SnowNLP 官网: https://pypi.org/project/snownlp/
- SnowNLP GitHub: https://github.com/isnowfy/snownlp


# SnowNLP用法
上一段代码，非常简单(注: SnowNLP用的是`unicode`编码, 如果是从Excel、text、csv等文件中读取，不要忘记decode and encode成`unicode`编码)
```Python
from snownlp import SnowNLP

s = SnowNLP(u'这个东西真心很赞')

s.words         # [u'这个', u'东西', u'真心',
                #  u'很', u'赞']

s.tags          # [(u'这个', u'r'), (u'东西', u'n'),
                #  (u'真心', u'd'), (u'很', u'd'),
                #  (u'赞', u'Vg')]

s.sentiments    # 0.9769663402895832 positive的概率

s.pinyin        # [u'zhe', u'ge', u'dong', u'xi',
                #  u'zhen', u'xin', u'hen', u'zan']

s = SnowNLP(u'「繁體字」「繁體中文」的叫法在臺灣亦很常見。')

s.han           # u'「繁体字」「繁体中文」的叫法
                # 在台湾亦很常见。'

text = u'''
自然语言处理是计算机科学领域与人工智能领域中的一个重要方向。
它研究能实现人与计算机之间用自然语言进行有效通信的各种理论和方法。
自然语言处理是一门融语言学、计算机科学、数学于一体的科学。
因此，这一领域的研究将涉及自然语言，即人们日常使用的语言，
所以它与语言学的研究有着密切的联系，但又有重要的区别。
自然语言处理并不是一般地研究自然语言，
而在于研制能有效地实现自然语言通信的计算机系统，
特别是其中的软件系统。因而它是计算机科学的一部分。
'''

s = SnowNLP(text)

s.keywords(3)	# [u'语言', u'自然', u'计算机']

s.summary(3)	# [u'因而它是计算机科学的一部分',
                #  u'自然语言处理是一门融语言学、计算机科学、
				#	 数学于一体的科学',
				#  u'自然语言处理是计算机科学领域与人工智能
				#	 领域中的一个重要方向']
s.sentences

s = SnowNLP([[u'这篇', u'文章'],
             [u'那篇', u'论文'],
             [u'这个']])
s.tf
s.idf
s.sim([u'文章'])# [0.3756070762985226, 0, 0]
```

# 关于如何训练自己的的语料库
现在提供训练的包括分词，词性标注，情感分析，而且都提供了我用来训练的原始文件 以分词为例 分词在`snownlp/seg`目录下
```python
from snownlp import seg
seg.train('data.txt')
seg.save('seg.marshal')
# from snownlp import tag
# tag.train('199801.txt')
# tag.save('tag.marshal')
# from snownlp import sentiment
# sentiment.train('neg.txt', 'pos.txt')
# sentiment.save('sentiment.marshal')
```
这样训练好的文件就存储为`seg.marshal`了，之后修改`snownlp/seg/__init__.py`里的`data_path`指向刚训练好的文件即可


【注】
- 知网发布“情感分析用词语集（beta版）http://www.keenage.com/html/c_bulletin_2007.htm
- Python 文本挖掘：使用情感词典进行情感分析（情感词典 ） http://rzcoding.blog.163.com/blog/static/2222810172013101991918346/
- Python 文本挖掘：使用情感词典进行情感分析（算法及程序设计） http://rzcoding.blog.163.com/blog/static/2222810172013101844033170/
