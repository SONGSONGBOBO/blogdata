---
title: '[couchdb]'
date: 2018-11-02 19:30:53
tags:
- Hyperledger-Fabric
toc: ture
---

### 总览
* NoSQL
* fbric中的couchdb


<!--more-->

### NoSQL

两种类型的数据库:
	1.关系数据库
	2.NoSQL
特点:NoSQL数据库比RDBMS更快，因为它与关系数据库相比使用不同的数据结构。 NoSQL数据库可以存储结构化和非结构化数据，如音频文件，视频文件，文档等。
NoSQL数据库可以根据其数据存储性质分为三种类型：
	1.键-值存储
	2.列存储
	3.文件存储

### fbric中的couchdb

历史数据和区块链的索引是LevelDB
State Database 由于和业务相关，所以提供了替换数据库，目前支持默认的LevelDB和用户可选择的CouchDB
“富查询”，是对Value的内容进行查询，如果是LevelDB，那么是不支持，只有CouchDB时才能用这个方法。
