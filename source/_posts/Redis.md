---
title: Redis
date: 2023-07-27 17:34:37
tags: 技术
categories: 技术
---

## 1.使用案例

* 计数器
* 限速器
* 使用 bitmap 实现用户上线次数统计
* String类型的使用场景
  * 计数器
  * 统计多单位的数量:uuid:123444:follow 0
  * 粉丝数
  * 对象存储缓存
* List类型的使用场景
  * 消息排队
  * 消息队列
  * 栈
* Hash类型的使用场景
  * Hash变更的数据 user name age,尤其是用户信息之类的,经常变动的信息!
  * Hash更适合对象的存储,String更适合字符串存储!
* Set类型的使用场景
  * 交集、并集、差集等场景的使用
* ZSet类型的使用场景
  * set排序,存储班级成绩表 工资表排序
  * 普通消息,1.重要消息 2.带权重进行判断
  * 排行榜应用实现,取Top N测试
* Geospatial(地理位置)
  * 通过georadius就可以完成附近的人功能
  * withcoord:带上坐标
  * withdist:带上距离,单位与半径单位相同
  * COUNT n:只显示前n个(按距离递增排序)
* Hyperloglog(基数统计)
  * 网页的访问量

## 2.线程模型

[(132条消息) Redis源码剖析——线程模型_redis线程模型_oywLearning的博客-CSDN博客](https://blog.csdn.net/weixin_44479862/article/details/126512100)

## 3.哨兵模式

在 Redis 主从复制模式中，因为系统不具备自动恢复的功能，所以当主服务器（master）宕机后，需要手动把一台从服务器（slave）切换为主服务器。在这个过程中，不仅需要人为干预，而且还会造成一段时间内服务器处于不可用状态，同时数据安全性也得不到保障，因此主从模式的可用性较低，不适用于线上生产环境。

Redis 官方推荐一种高可用方案，也就是 Redis Sentinel 哨兵模式，它弥补了主从模式的不足。Sentinel 通过监控的方式获取主机的工作状态是否正常，当主机发生故障时， Sentinel 会自动进行 Failover（即故障转移），并将其监控的从机提升主服务器（master），从而保证了系统的高可用性。

[Redis集群：Sentinel哨兵模式（详细图解） (biancheng.net)](http://c.biancheng.net/redis/sentinel-model.html)

## 4.集群

[(132条消息) Redis集群（Cluster）_Hpuers的博客-CSDN博客](https://blog.csdn.net/qq_46370017/article/details/126347976)

## 5.缓存击穿\缓存穿透\缓存雪崩解决方案

[(132条消息) [Redis\]缓存穿透、缓存击穿、缓存雪崩问题及解决方法_Bruce1801的博客-CSDN博客](https://blog.csdn.net/m0_51963973/article/details/131362602)

## 6.Jedis

Jedis就是集成了redis的一些命令操作，封装了redis的java客户端。并且提供了连接池管理。

[(132条消息) Redis之jedis_redis jedis_yzm4399的博客-CSDN博客](https://blog.csdn.net/qq_43654581/article/details/121770415)

## 7.Redission

[(132条消息) 专题四 Redis分布式锁中——Redission_v_BinWei_v的博客-CSDN博客](https://blog.csdn.net/ohwang/article/details/125038268)

## 8.MongoDB使用案例

**MongoDB的适用场景**

* 网站数据：MongoDB非常适合实时插入，更新与查询，并具备网站实时数据存储所需的复制及高度伸缩性。
* 缓存：由于性能很高，Mongo也适合作为信息存储的缓存层。在系统重启之后，由Mongo搭建的持久化缓存层可以避免下层数据资源过载。
* 大尺寸、低价值的数据：使用传统的关系型数据库存储一些大尺寸低价值数据时比较浪费资源，在此之前，很多程序员往往选择使用传统文件存储的方式。
* 高伸缩性的场景：Mongo非常适合由数十台或数百台服务器组成的数据库，Mongo的线路图中已经包含对MapReduce引擎的内置支持以及集群高可用的解决方案。
* 用于对象及JSON数据的存储：Mongo的BSON数据格式非常适合文档化的存储及查询。
  **MongoDB的行业应用场景**
* 游戏场景：使用MongoDB存储游戏用户信息，用户的装备、积分等直接以内嵌文档的形式存储，方便直接查询、更新。
* 物流场景：使用MongoDB存储订单信息，订单状态在运行过程中会不断更新，以MongoDB内嵌数据的形式来存储，一次查询就能将订单所有的变更信息读出来。
* 
* 社交场景：使用MongoDB存储用户信息，以及用户发布的朋友圈信息，通过地理位置索引实现附近的人、地点等功能。
  物联网场景：使用MongoDB存储所有接入的智能设备，以及设备汇报的日志信息，并对这些信息进行多维度的分析。
* 直播：使用MongoDB存储用户信息、礼物信息等。

