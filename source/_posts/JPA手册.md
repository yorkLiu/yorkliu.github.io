---
title: JPA手册
copyright: true
date: 2018-07-12 17:11:49
tags: JPA手册
categories:
  - Java
  - JPA
---

# 什么么是JPA?
 全称Java Persistence API，可以通过注解或者XML描述【对象-关系表】之间的映射关系，并将实体对象持久化到数据库中。
 为我们提供了：
   1. ORM映射元数据：JPA支持XML和注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中；
      如：@Entity、@Table、@Column、@Transient等注解。

   2. JPA 的API：用来操作实体对象，执行CRUD操作，框架在后台替我们完成所有的事情，开发者从繁琐的JDBC和SQL代码中解脱出来。
      如：entityManager.merge(T t)；

   3. JPQL查询语言：通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合。
      如：from Student s where s.name = ?

 **但是：**
 JPA仅仅是一种规范，也就是说JPA仅仅定义了一些接口，而接口是需要实现才能工作的。所以底层需要某种实现，而Hibernate就是实现了JPA接口的ORM框架。

 也就是说：

 JPA是一套ORM规范，Hibernate实现了JPA规范！如图：

 ![](https://oscimg.oschina.net/oscnet/fd5e9c1f88bcdb6c41f6685998b44e0f068.jpg)

# 什么是spring data JPA?
 Spirng Data JPA是spring提供的一套简化JPA开发的框架，按照约定好的【方法命名规则】写dao层接口，就可以在不写接口实现的情况下，实现对数据库的访问和操作。同时提供了很多除了CRUD之外的功能，如分页、排序、复杂查询等等。

 Spring Data JPA 可以理解为 JPA 规范的再次封装抽象，底层还是使用了 Hibernate 的 JPA 技术实现。如图：

 ![](https://oscimg.oschina.net/oscnet/3169065e3f44e0994f9a01e0df7ed38fe7a.jpg)

 接口约定命名规则：

 ![](https://oscimg.oschina.net/oscnet/2da91405d2138b4ddedb7b148f0d43bc219.jpg)

 实例：

 ![](https://oscimg.oschina.net/oscnet/719f690bc8bc60365371478aa84fa014ab7.jpg)

 ![](https://oscimg.oschina.net/oscnet/eddc9d37ac7656fd90a2e5093441775905f.jpg)

 springboot集成spring data jpa只需两步：

 第一步：导入maven坐标

 ![](https://oscimg.oschina.net/oscnet/2bac153bb2147283bb1b9dd0c7937e47ff7.jpg)

 第二步：yml配置文件中配置jpa信息

 ![](https://oscimg.oschina.net/oscnet/334e8961dbfee889d19cd2febfdf6d68153.jpg)
