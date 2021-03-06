---
title: '[fabric架构原理]'
date: 2018-11-01 19:15:44
tags:
 - Hyperledger-Fabric
toc: ture
---

### 总览
* 推荐博客和书籍
* 架构
* 事务处理
* 读写集
* 总结

<!--more-->

### 推荐博客

[李佶澳](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/06/25/hyperledger-fabric-main-point.html)

书籍:随便一本就行

### 架构

{% asset_img 1.png 架构 %}

单一的peer节点在1.0中进行了拆分，分为peer（背书节点和提交节点）和orderer（排序节点）。membership也就是CA节点

{% asset_img 2.png 账本 %}

Fabric 1.0中的账本分为3种：
	1.区块链数据，这是用文件系统存储在Committer节点上的。区块链中存储了Transaction的读写集。
	2.为了检索区块链的方便，所以用LevelDB对其中的Transaction进行了索引。
	3.ChainCode操作的实际数据存储在State Database中，这是一个Key Value的数据库，默认采用的LevelDB，1.0也支持使用CouchDB作为State Database。


### 事务处理

{% asset_img 3.png 事物处理 %}

### 读写集

在背书人交易模拟期间，会为交易准备一个读写数据集。读集包含一份唯一键的列表和该键的提交版本，用于交易模拟期间读取。写集包含一份唯一键的列表（键值可能与读集有重叠）和交易写入的对应的新键值。如果交易更新是删除该键，那么会对该键做一个删除标记。 

进一步的，如果交易对一个键对应的值进行多次写操作，该键只会保留最后一次写入的值。同样，如果交易读取一个键对应的值，会返回处于已提交状态的值。即使在进行读取操作之前该交易已经更新了该键的值。换句话说就是，不支持在交易中读取该交易写入的数据。

提交者使用读写集的读集部分校验一笔交易的有效性，使用写集更新相关键的版本和键值。 

在验证阶段，如果一笔交易中每个读集中的键与世界状态中对应键的版本号一致，那么该笔交易被认为是有效的–假设先前的有效交易都已经被提交了，包括同一区块中的先前交易。 

### 总结

提交peer上区块同步存储账本全部信息，状态数据库保存当前世界状态，建立索引，快速进行查询
读写操作，读只需peer地址，而写需要排序节点。

