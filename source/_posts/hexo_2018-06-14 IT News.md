---
title: 2018-06-14 IT News
copyright: true
date: 2018-06-14 08:20:44
tags: IT NEWS
categories: IT NEWS
---
# IT 新闻 
 ## [Dubbo解析(二)-内核实现之SPI机制（下）](https://my.oschina.net/u/2377110/blog/1829922)
 > 上一章我们介绍了JDK的SPI机制，它旨在建立一种服务发现的规范。而Dubbo基于此根据框架的整体设计做了一些改进： 1. JDK的SPI机制会一次性实例化所有服务提供者实现，如果有提供者的初始化很耗时，但并不会使用会很耗费资源。**Dubbo则只存储了所有提供者的Class对象，实际使用时才构造对
 ## [分布式环境下的配置变更如何做到平滑](https://my.oschina.net/fileoptions/blog/1829700)
 > 时长有种感觉，技术和生活都是相同，很多看似深奥的技术原理都可以在生活中找到映射，比如。        假设一个场景，你有很多员工，分散在世界各地，你们平时都用邮件沟通，有一天你想切换为钉钉沟通，那么你会怎么做呢？首先，你会想到群发一个邮件给你的所有员工，然而，你无法确认员工是否查看了邮件，什么时候查
 ## [redis sds 实现的几个巧妙的想法](https://my.oschina.net/u/2950272/blog/1829687)
 > 翻redis 源码的时候，发现有些用法真是很巧妙的，一个是指针变换，一个是内存管理策略，当然后者是有利弊的。         sds.h 就这两个数据类型，这里，能够sds 和 sdshdr 相互转化。对外的接口，参数和返回只有char * 。redis 为每个char * 都维护了个数据结构 sd
 ## [精讲Redis内存模型](https://my.oschina.net/u/3779583/blog/1829617)
 > 前言 Redis是目前最火爆的内存数据库之一，通过在内存中读写数据，大大提高了读写速度，可以说Redis是实现网站高并发不可或缺的一部分。 我们使用Redis时，会接触Redis的5种对象类型（字符串、哈希、列表、集合、有序集合），丰富的类型是Redis相对于Memcached等的一大优势。在了解R
 ## [神奇的403](https://my.oschina.net/qixiaobo025/blog/1829469)
 > 背景 应曾老师要求 来写一篇神奇的403…… 分享会结束后 某曾老师高兴地说道：“试用出bug了……测试【^^！】说你修改了profile“ 先来看一下现象！ 请求返回403~！ 问题 /kzf6/maintain/getNoFinishMaintainList.do?idCar=406059193
 ## [apiDoc构建源代码注释的接口文档](https://my.oschina.net/wuweixiang/blog/1829384)
 > apiDoc官网：http://apidocjs.com/ 入门 前言 本文档中的所有示例都使用Javadoc-Style（可用于C＃，Go，Dart，Java，JavaScript，PHP，TypeScript和所有其他支持Javadoc的语言）： /** * This is a comment.
 ## [Guava 源码分析（Cache 原理）](https://my.oschina.net/crossoverjie/blog/1829337)
 > !\[1.jpeg\](https://i.loli.net/2018/06/12/5b1fea79e07cb.jpeg)  前言 Google 出的 \[Guava\](https://github.com/google/guava) 是 Java 核心增强的库，应用非常广泛。 我平时用的也挺频繁，这
 ## [flag -- 诡异的memcache标记](https://my.oschina.net/jijunjian/blog/1829272)
 > 引子　　　 　    打从去年一路北漂，进入无人货架行业，业务需求漫天飘，最近总算把工作都规划齐整。回望过去一年多的时间里，诸多东西值得整理，memcache就是其中一个。 　　 看到java的工资高些，队伍中好些人都想学习java，美其名曰：技术多元化。奈何团队中并没有相关经验的人，也深知大家殷切
 ## [脚本语言不行？JavaScript 重写 Office 365 已进入尾声](https://www.oschina.net/news/97060/office-365-is-rewriting-with-javascript)
 > 微软技术项目经理（Technical Program Manager ，TPM）、Webpack 核心团队成员 Sean Thomas Larkin  发 Twitter 透露了 Office 365 正在用 JavaScript 重写的消息。 这个消息源于一次简短的编程语言口水之争。一个备注 C+
 ## [如何像扎克伯格一样实际编码创造一个 Javis，做钢铁侠？](https://www.oschina.net/event/2279616?origin=zhzd)
 > 重点介绍任务型对话机器人的实现方案，并结合一个实际案例，介绍如何利用可视化编辑工具，简便地创建一个任务型对话机器人并集成到自己的业务系统或产品中。
 ## [微软 6 月 Patch Tuesday 修复影响所有 Windows 的漏洞](https://www.oschina.net/news/97058/microsoft-patch-tuesday-updates-for-1806)
 > 微软发布了 6 月 Patch Tuesday 的更新版本，解决了 50 个漏洞，其中有 11 个关键的远程代码执行漏洞和 39 个被定级为“重要”的漏洞。 据介绍，这些关键问题的影响涉及到 Edge 和 IE 浏览器。在发布修补程序前，除了脚本引擎中的 CVE-2018-8267 漏洞已经被利用并
 ## [Google 禁止从第三方网站安装 Chrome 扩展程序](https://www.oschina.net/news/97057/chrome-to-ban-inline-installation-of-extension)
 > 谷歌宣布将废除 Chrome 扩展的内联安装方式（inline installation），这将使用户无法从第三方网站安装 Chrome 扩展程序。 Google 一直试图改进 Chrome 浏览器扩展的安装体验，一般用户可以直接从 Google 网上应用商店安装所需 Chrome 扩展，而有些开发
 ## [每日一博 | Redis 内存模型深度解析与应用](https://my.oschina.net/u/3779583/blog/1829617)
 > 文章主要介绍Redis的内存模型（以3.0为例），包括Redis占用内存的情况及如何查询、不同的对象类型在内存中的编码方式、内存分配器(jemalloc)、简单动态字符串(SDS)、RedisObject等；然后在此基础上介绍几个Redis内存模型的应用。
 ## [码云推荐 | 可扩展的报警系统 Quick-Alarm](https://gitee.com/liuyueyi/quick-alarm)
 > Quick-Alarm 是一个通用报警框架，支持报警方式自定义、报警配置自定义、动态的报警配置、用户自定义报警规则拓展、报警方式自动切换规则设定，以及报警方式自定义自动切换规则拓展等。
 ## [v-region — 基于 Vue2 的中国行政区划选择器](https://www.oschina.net/p/v-region)
 > v-region 是基于 Vue2 的简洁、易用的中国行政区划选择器，包含常规表单下拉列表模式和UI下拉选择器模式；支持 “省/直辖市”-“市”-“区/县”-“乡/镇/街道”4级行政级别。支持插件预览，提供基础表单模式和选择器模式。
 ## [微软更改 Windwos Server 2008 SP2 的服务模式](https://www.oschina.net/news/97053/microsoft-changes-support-mode-of-win-server-08sp2)
 > 微软 12 日在其官网宣布改变对 Windows Server 2008 SP2 的支持模式。 微软自 2016 年 10 月起为 Windows 7 和 Windows 8.1 执行了“每月质量汇总”服务计划，Windows 服务和交付团队高级项目经理 Sachin Goyal 表示，今后 Win
 ## [OSChina 周四乱弹 —— 二次元世界很不安全](https://my.oschina.net/xxiaobian/blog/1829923)
 > 最近一哥们天天请一姑娘吃冰激凌，不解，问起原因。答曰：“科学表明女生经期的时候表白成功率大，又不好意思问姑娘的日子，就天天请冰激凌，等哪天姑娘不吃了肯定就是来了姨妈了。”
 ## [NMT 引入移动设备，Google Translate 离线翻译更逼真](https://www.oschina.net/news/97051/offline-translations-are-better-thanks-nmt)
 > 谷歌 12 日在官网宣布已将神经机器翻译（Neural Machine Translation，NMT）技术应用到移动设备 Android 和 iOS 上，使得 Google Translate 的离线翻译能力上了一个台阶。 谷歌于约两年前推出 NMT 技术，并将其引入 Google Transla
 ## [Firefox 61.0 Beta 13 (Quantum) 发布，支持暗色主题](https://www.oschina.net/news/97050/firefox-61-beta3-released)
 > Mozilla Firefox 61.0 Beta 已发布，主要带来了 Quantum 引擎性能上的改进，优化了处理 CSS 的过程，让页面加载速度更快，并且用户还会感觉到切换标签页也会变快。 同时 Firefox 61.0 Beta 还提供了对暗色主题的支持。有兴趣的朋友可以登录 Mozilla 
 ## [Kotlin 1.2.50 发布，在 Eclipse IDE 插件中获支持](https://www.oschina.net/news/97049/kotlin-1-2-50-released)
 > Kotlin 1.2.50 已发布，带来了一些 Bug 修复和工具更新。主要更新内容如下： 在 Eclipse IDE 插件中更新 Kotlin 支持 在标准库的常见部分和 JS 部分添加新功能 将 JUnit 5 支持带给 kotlin.test 改进了实验性脚本支持 在 IntelliJ IDE
 ## [RabbitMQ 3.6.16 和 3.7.6 发布，多协议消息代理](https://www.oschina.net/news/97048/rabbitmq-3-6-16-and-3-7-6-released)
 > RabbitMQ 3.6.16 和 3.7.6 已发布，RabbitMQ 3.6.16 是一个维护版本，主要包括来自 3.7.x 系列的选定 backports。 建议早期 3.6.x 版本的用户升级到 3.7.x 版本，如 3.7.6。3.7.6 主要包含系列 Bug 修复。 3.6.16 更新情
 ## [React 16.4.1 发布，构建用户界面的 JavaScript 库](https://www.oschina.net/news/97047/react-16-4-1-released)
 > React 16.4.1 已发布，主要更新内容如下： React 现在可以将 propType 分配给由 React.ForwardRef 返回的组件。(@bvaughn in 12911) React DOM 修复：当输入类型从其他类型更改为文本时崩溃。 (@spirosikmd in 121
 ## [Elasticsearch 6.3.0 和 5.6.0 发布，Bug 修复](https://www.oschina.net/news/97046/elasticsearch-630-and-560-released)
 > Elasticsearch 6.3.0 和 5.6.0 已发布 6.3.0 更新内容如下： BUGFIX: 在关闭管道时修正竞态条件 9285 BUGFIX: 确保永久队列检查点的原子创建 9303 BUGFIX: 修复了包含非 ASCII 字符的事件在通过持久队列后不正确编码的问题 9307
 ## [Redis 5.0 rc2，4.0.10 和 3.2.12 发布，修复安全问题](https://www.oschina.net/news/97045/redis-50rc2-4010-32122-released)
 > Redis 5.0 rc2，4.0.10 和 3.2.12 已发布。主要更新情况如下： 5.0 rc2 此版本修复了重要的安全问题。 此版本修复了 SCAN 命令系列错误。 此版本修复了 PSYNC2 边界案件到期。 与 Sentinel 相关的修复。 修复其他问题 详细更新内容 4.0.10 此版
 ## [TypeScript 2.9.2 发布，微软推出的 JavaScript 超集](https://www.oschina.net/news/97044/typescript-2-9-2-released)
 > TypeScript 2.9.2 已发布。此版本包含一组针对 TypeScript 2.9.1 的错误修复。 有关已解决问题的完整列表，请查看 TypeScript 2.9.2 的固定问题查询。 TypeScript 是由微软开发的自由和开源的编程语言，是 JavaScript 类型的超集，它可以编
 ## [Next.js 6.0.4-canary.6 发布，React 服务器端渲染框架](https://www.oschina.net/news/97043/next-js-6-0-4-canary-6-released)
 > Next.js 6.0.4-canary.6 已发布，更新内容如下： Check if the incoming header is set before applying to the response: 2a1075c Next.js 是一个用于在服务端渲染 React 应用程序的简单框架。 下
 ## [CodeMaid 10.5.119 发布，开源 Visual Studio 扩展](https://www.oschina.net/news/97042/codemaid-10-5-119-released)
 > CodeMaid 是一个开源的 Visual Studio 扩展， 用来清理和简化你的 C, C++, F, VB, PHP, PowerShell, JSON, XAML, XML, ASP, HTML, CSS, LESS, SCSS, JavaScript 和TypeScript 的代码以
 ## [IntelliJ IDEA 2018.1.5 正式发布，错误修复和改进](https://www.oschina.net/news/97041/intellij-idea-2018-1-5-released)
 > IntelliJ IDEA 2018.1.5 已正式发布。本次更新修复了几处回归错误，以及各种 bug 修复，点此查看本次更新的所有修复内容。 值得关注的改进 在使用/取消导航弹出窗口/菜单后，IDE 不会失去焦点 IDEA-191839. 导航到类(Сtrl+N /cmd+O)这个功能现在可以正常
 ## [魅族张兴业谈实践：利用Weex技术做魅族小程序](http://news.51cto.com/art/201806/576117.htm)
 > 魅族张兴业谈实践：利用Weex技术做魅族小程序
 ## [【WOT2018】如何借助AR提升企业竞争力？三位大咖教你轻松布局](http://developer.51cto.com/art/201806/576107.htm)
 > 【WOT2018】如何借助AR提升企业竞争力？三位大咖教你轻松布局
 ## [【有奖话题讨论】云时代下，企业IT运维如何应势而变？](http://news.51cto.com/art/201806/576105.htm)
 > 【有奖话题讨论】云时代下，企业IT运维如何应势而变？
 ## [运维的本质是什么？阿里“无人化”智能运维平台的演进](http://os.51cto.com/art/201806/576094.htm)
 > 运维的本质是什么？阿里“无人化”智能运维平台的演进
 ## [商助科技顾问总监郑泉：互联网企业如何实现基于场景的大数据营销](http://stor.51cto.com/art/201806/576098.htm)
 > 商助科技顾问总监郑泉：互联网企业如何实现基于场景的大数据营销
 ## [七款适用于企业的开源 VPN 工具](http://netsecurity.51cto.com/art/201806/575993.htm)
 > 七款适用于企业的开源 VPN 工具
 ## [荷兰央行：区块链目前无法满足金融基础设施需求](http://blockchain.51cto.com/art/201806/576042.htm)
 > 荷兰央行：区块链目前无法满足金融基础设施需求
 ## [Python爬取历年高考分数线，帮你预测2018年高考分数线](http://developer.51cto.com/art/201806/576084.htm)
 > Python爬取历年高考分数线，帮你预测2018年高考分数线
 ## [如何评估和选择IoT平台？](http://iot.51cto.com/art/201806/576193.htm)
 > 企业可以持续专注于自身 IoT 平台的基本任务和功能，从而摆脱噪音的干扰。用一种条理分明的方法来梳理哪些
 ## [物联网的“小媳妇”，就该被冷落么？](http://iot.51cto.com/art/201806/576192.htm)
 > 表面看来，农业物联网确实像偏远的乡村一样，跟不上大城市繁华的脚步，但若从整个大农业的角度具体分析，农
 ## [CSDN日报1806012——《欠薪的公司，不要做任何犹豫》](https://blog.csdn.net/blogdevteam/article/details/80667708)
 > CSDN日报1806012——《欠薪的公司，不要做任何犹豫》
 ## [大规模分布式系统的跟踪系统：Dapper设计给我们的启示](https://blog.csdn.net/liumiaocn/article/details/80657661)
 > 大规模分布式系统的跟踪系统：Dapper设计给我们的启示
 ## [奥布莱恩杯尘埃落定 人工智能立功了！](https://blog.csdn.net/sunhf_csdn/article/details/80671342)
 > 奥布莱恩杯尘埃落定 人工智能立功了！
 ## [JavaScript三元运算符的使用 进阶三元运算逻辑拓展篇](https://blog.csdn.net/superwebmaster/article/details/80677593)
 > JavaScript三元运算符的使用 进阶三元运算逻辑拓展篇
 ## [程序员：如何优雅地装逼](https://blog.csdn.net/m68FUTKMUrmtj/article/details/80544927)
 > 程序员：如何优雅地装逼
 ## [你靠什么在单位立足？此文堪称经典！](https://blog.csdn.net/Px01Ih8/article/details/80577810)
 > 你靠什么在单位立足？此文堪称经典！
 ## [我们为什么应该坚持写博客](https://blog.csdn.net/ityouknow/article/details/80589552)
 > 我们为什么应该坚持写博客
 ## [在互联网圈混，怎么能不知道这9个Java方向公众号](https://blog.csdn.net/g6U8W7p06dCO99fQ3/article/details/80571296)
 > 在互联网圈混，怎么能不知道这9个Java方向公众号
 ## [js原生创建模拟事件和自定义事件的方法](https://blog.csdn.net/shadow_zed/article/details/80666526)
 > js原生创建模拟事件和自定义事件的方法
 ## [华为资深工程师：码农很多，但程序员并不多......](https://blog.csdn.net/tTU1EvLDeLFq5btqiK/article/details/80655451)
 > 华为资深工程师：码农很多，但程序员并不多......
 ## [在IT圈混，怎么能不知道这些公众号？](https://blog.csdn.net/Mbx8X9u/article/details/80562386)
 > 在IT圈混，怎么能不知道这些公众号？
 ## [猝不及防，Google成功“造人”令人胆寒！人类迎来史上最惨失业潮…](https://blog.csdn.net/FnqTyr45/article/details/80685250)
 > 猝不及防，Google成功“造人”令人胆寒！人类迎来史上最惨失业潮…
 ## [10个web开发好用框架](https://blog.csdn.net/lmseo5hy/article/details/80667062)
 > 10个web开发好用框架
 ## [程序员如何在百忙之中不走岔路，不白忙！](https://blog.csdn.net/bntX2jSQfEHy7/article/details/80544896)
 > 程序员如何在百忙之中不走岔路，不白忙！
 ## [面试教程 | 多家AI公司面试之后，我学到了什么？](https://blog.csdn.net/eNohtZvQiJxo00aTz3y8/article/details/80649808)
 > 面试教程 | 多家AI公司面试之后，我学到了什么？
 ## [“机海战术”已死！后智能手机时代靠什么才能赢？](https://blog.csdn.net/csdnnews/article/details/80683330)
 > 点击上方“CSDN”，选择“置顶公众号”关键时刻，第一时间送达！昨天...
 ## [A 站彻底要凉？近千万条用户数据外泄！](https://blog.csdn.net/csdnnews/article/details/80681539)
 > 点击上方“CSDN”，选择“置顶公众号”关键时刻，第一时间送达！用户...
 ## [年薪 700 万也换不来区块链开发者的一次回眸](https://blog.csdn.net/csdnnews/article/details/80681541)
 > 点击上方“CSDN”，选择“置顶公众号”关键时刻，第一时间送达！【C...
 ## [苹果封杀加密货币！](https://blog.csdn.net/csdnnews/article/details/80675227)
 > 听说，挖币很挣钱？    手机挖币是不是骗局？    币还没挖着，手机的电量和 CPU 瞬间被榨干了！不知何时起，腥风血雨的币圈江湖杀入了大街小巷，一批批投资者、创业者、热钱纷纷涌入其中，「一念天堂，一念地狱」的现象也层出不穷，为避免这种现状愈演愈烈，监管政策随之而来。如今...
 ## [动辄年薪 25 万只是白菜价的人工智能黄了？](https://blog.csdn.net/csdnnews/article/details/80675076)
 > 近几年，人工智能风生水起，中美巨头纷纷布局，创业型公司纷至沓来，深度学习、机器学习、算法等技术名词不绝于耳，“智能”产品琳琅满目……然而精彩纷呈背后，各种「AI 威胁论」也水涨船高：  马斯克：如果你不担心人工智能的安全性，那么现在你应该担心。它比朝鲜核武器危险得多。    霍金：远离 ...
# 人工智能 
 ## [CSDN日报1806012——《欠薪的公司，不要做任何犹豫》](https://blog.csdn.net/blogdevteam/article/details/80667708)
 > CSDN日报1806012——《欠薪的公司，不要做任何犹豫》
 ## [\[PVLDB 12\] GraphLab : 分布式机器学习大规模图处理系统 学习总结](https://blog.csdn.net/qq_21125183/article/details/80678187)
 > \[PVLDB 12\] GraphLab : 分布式机器学习大规模图处理系统 学习总结
 ## [matlab版本POSIT算法来计算三维以及二维人脸模型的映射透视投影矩阵](https://blog.csdn.net/baidu_26408419/article/details/80678211)
 > matlab版本POSIT算法来计算三维以及二维人脸模型的映射透视投影矩阵
 ## [python的中文文本挖掘库snownlp进行购物评论文本情感分析实例](https://blog.csdn.net/hellozhxy/article/details/80678263)
 > python的中文文本挖掘库snownlp进行购物评论文本情感分析实例
 ## [什么BIM模型精度](https://blog.csdn.net/jxzx1007/article/details/80678335)
 > 什么BIM模型精度
 ## [Softmax回归函数](https://blog.csdn.net/laobai1015/article/details/80678338)
 > Softmax回归函数
 ## [【Machine Learning@Andrew Ng, Coursera】机器学习Week1 单变量线性回归笔记](https://blog.csdn.net/weixin_42395916/article/details/80678464)
 > 【Machine Learning@Andrew Ng, Coursera】机器学习Week1 单变量线性回归笔记
 ## [【机器学习课程-华盛顿大学】：3 分类 3.4 决策树过拟合](https://blog.csdn.net/weixin_41770169/article/details/80678471)
 > 【机器学习课程-华盛顿大学】：3 分类 3.4 决策树过拟合
 ## [高手如何做数据分析？这11招是你应该具备的技能](https://blog.csdn.net/qq_42154484/article/details/80678481)
 > 高手如何做数据分析？这11招是你应该具备的技能
 ## [端午快到了！去哪里最好玩？人最少？提前用Python分析一波！](https://blog.csdn.net/qq_42156420/article/details/80678483)
 > 端午快到了！去哪里最好玩？人最少？提前用Python分析一波！
 ## [网易北京研发中心-网易传媒部门深度学习算法实习生面试总结](https://blog.csdn.net/program_developer/article/details/80678551)
 > 网易北京研发中心-网易传媒部门深度学习算法实习生面试总结
 ## [入行机器学习算法，其实就是顺应时代发展](https://blog.csdn.net/korea1121/article/details/80678647)
 > 入行机器学习算法，其实就是顺应时代发展
 ## [小白上手深度学习，就等着哭吧](https://blog.csdn.net/korea1121/article/details/80678704)
 > 小白上手深度学习，就等着哭吧
 ## [官网实例详解4.2（antirectifier.py）-keras学习笔记四](https://blog.csdn.net/wyx100/article/details/80678735)
 > 官网实例详解4.2（antirectifier.py）-keras学习笔记四
 ## [PCA故障检测步骤](https://blog.csdn.net/sp353846548/article/details/80678740)
 > PCA故障检测步骤
 ## [\[Python人工智能\] 六.神经网络的评价指标、特征标准化和特征选择](http://blog.csdn.net/eastmount/article/details/80650980)
 > 从本系列文章开始，作者正式开始研究Python深度学习、神经网络及人工智能相关知识。前五篇文章讲解了神经网络基础概念、Theano库的安装过程及基础用法、theano实现回归神经网络、theano实现...
# PM 
 ## [如何解决晋升后的困惑？这里有5个方法](http://www.pmtoo.com/article/48134.html)
 > 当你刚升职的时候，产生困惑是很正常的事情，那面对这些困惑的时候，我们要如何处理呢？文中提供了五个方法，一起来看看~...
 ## [新4C：低成本引爆社群的秘籍](http://www.pmtoo.com/article/48123.html)
 > 新4C是指在合适的场景下，针对特定的社群，利用有传播力的内容或者话题，通过社群网络中人与人的连接的裂变实现快速扩散与传...
 ## [江小白案例借鉴：传统企业营销方式如何变革？](http://www.pmtoo.com/article/48119.html)
 > 在近几年江小白通过颠覆传统的营销方式，实现了低成本地快速切入市场。那传统企业能从江小白的营销案例中学到什么呢？在此基...
 ## [抖音国内日活破1.5亿 与微信矛盾公开化但并不依赖对方](http://www.pmtoo.com/article/48112.html)
 > 6月12日，抖音首次对外公布最新用户数据：截止目前，抖音国内日活用户突破1.5亿，月活用户超过3亿。同时现场对近期业界关注的...
 ## [阿里巴巴又出了一个读书应用，说要做严肃文学](http://www.pmtoo.com/article/48106.html)
 > 阿里巴巴又做了一个新的读书应用，叫天猫读书。这个是由阿里文学与天猫图书联合推出的数字阅读产品，在此之前，阿里巴巴旗下...
 ## [男妆消费崛起，95后“花美男”的美妆生意开始了](http://www.pmtoo.com/article/48068.html)
 > 在消费升级、明星效应与求美流行文化的影响下，越来越多男生开始护肤、化妆，蜕变成一个个精致的男孩。十年前...
 ## [“精致boy”、“炫酷girl”双双崛起，消费的性别边界正逐渐模糊](http://www.pmtoo.com/article/48065.html)
 > 随着男性自我形象管理意识的提升和两性分工的变化，传统基于性别标签定义的消费需求正在发生改变，消费的性别边界正在逐渐模...
 ## [为什么说阿里应该收购抖音？](http://www.pmtoo.com/article/48059.html)
 > 放眼整个互联网行业，百度、网易是无从渗透的老牌劲旅，只有羽翼待丰的头条，是阿里系能够指望得上的强援。头条以算法做内...
 ## [A站受黑客攻击，官方：请及时修改密码](http://www.pmtoo.com/article/48055.html)
 > 6 月 13 号消息： 13 号凌晨，Acfun在官网发布了一篇文章，文中称A站因被黑客攻击，千万位用户的数据可能已经泄露，请 20...
 ## [中国“购物女神”通过分享自己的缺点建立信任，超级网红都有这３大力量！](http://www.pmtoo.com/article/48051.html)
 > 黎贝卡拥有「购物女神」的称号，是中国最具影响力的时尚自媒体之一。她在微博上拥有超过300万粉丝，微信上拥有超过45...

    