---
title: 数据库中DML、DDL、DCL的含义及区别
date: 2023-03-03 10:47:25
tags: 数据库
categories: 技术
---

![img](https://img1.baidu.com/it/u=3490013694,190534170&fm=253&fmt=auto&app=138&f=JPEG?w=600&h=356)

数据库管理系统软件是一种操纵和管理数据库的大型软件。

其功能包括数据库定义、数据操纵、数据库的运行管理、数据库建立和维护等。 

 数据库应用程序是指以数据库为基础，用VB或其他开发工具开发的、实现某种具体功能的程序。

一、DML与DDL的含义：

1、DML（Data Manipulation Language）数据操作语言-数据库的基本操作，SQL中处理数据等操作统称为数据操纵语言,简而言之就是实现了基本的“增删改查”操作。包括的关键字有：select、update、delete、insert、merge

2、DDL（Data Definition Language）数据定义语言-用于定义和管理 SQL 数据库中的所有对象的语言，对数据库中的某些对象(例如，database,table)进行管理。包括的关键字有：

create、alter、drop、truncate、comment、grant、revoke

二、DML与DDL的区别：

1.DML操作是可以手动控制事务的开启、提交和回滚的。

2.DDL操作是隐性提交的，不能rollback！
