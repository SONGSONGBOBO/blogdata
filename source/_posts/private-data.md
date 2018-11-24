---
title: '[private_data]'
date: 2018-11-08 16:11:25
tags:
- Hyperledger-Fabric
- private-data
categories: 
- Hyperledger-Fabric

toc : ture
---

## 总览
* private data collections
* 私有数据模式
* 在Fabric中使用私有数据


<!--more-->

### 官方文档地址
[private data](https://hyperledger-fabric.readthedocs.io/en/release-1.3/private-data-arch.html)


### private data collections

**为什么要隐私数据集合？**
如果在通道上的一群组织中，有几个组织需要将该通道上的数据保持为私有，那么它们可以选择创建一个新通道，仅由需要访问数据的组织组成。然而，在每一种情况下创建单独的通道会产生额外的管理开销(维护链码版本、策略、msp等)，并且，这种方法无法实现，所有加入通道的成员在保持部分数据私密的同时，还能够查看交易。这就是为什么，从v1.2开始，Fabric提供了创建私有数据集合的能力，它允许一个定义的组织子集在通道上支持、提交或查询私有数据，而不需要创建单独的通道。

**隐私数据集合是什么？**
Private Data collections 是在通道成员之间保持数据/事务私密性的一种方法。这种方法可以在不创建通道的情况下，来实现特定组织成员数据之间的共享。

### 案例

考虑一个由5个组织组成的联盟，在一个通道内进行贸易交易： 
* 农场主 
* 分销商 
* 货运商 
* 批发商 
* 零售商

这样，在基于price5个组织之间存在3条各自隐私的私有数据集合（private data collections PDC） 
1. PDC1:分销商、农场主和货运商 
2. PDC2：分销商和批发商 
3. PDC3：批发商、零售商和货运商 

{% asset_img 1.png private-data01 %}

### 私有数据模式

{% asset_img 2.png private-data02 %}

这是要实现新的例子
需要定义两个单独的数据定义
	name, color, size, and owner 将对通道的所有成员可见（Org1和Org2）
	price 仅对Org1的成员可见
此数据到限制其访问的集合策略的映射由链代码API控制。具体地讲，读，写使用集合定义私有数据是通过调用执行GetPrivateData() 而且PutPrivateData()

### 构建策略

```bash
// collections_config.json
[
 {
	 "name": "collectionMarbles",
	 "policy": "OR('Org1MSP.member', 'Org2MSP.member')",
	 "requiredPeerCount": 0,
	 "maxPeerCount": 3,
	 "blockToLive":1000000
},
 {
	 "name": "collectionMarblePrivateDetails",
	 "policy": "OR('Org1MSP.member')",
	 "requiredPeerCount": 0,
	 "maxPeerCount": 3,
	 "blockToLive":3
 }
]
```
这些策略要保护的数据以链代码映射，当使用peer chaincode instantiate命令在通道上实例化其关联的链代码时，此集合定义文件部署在通道上

### 私有数据编写

[api文档](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim)

```bash
// ==== Create marble object and marshal to JSON ====
      objectType := "marble"
      marble := &marble{objectType, marbleName, color, size, owner}
      marbleJSONasBytes, err := json.Marshal(marble)
      if err != nil {
              return shim.Error(err.Error())
      }

      // === Save marble to state ===
      err = stub.PutPrivateData("collectionMarbles", marbleName, marbleJSONasBytes)
      if err != nil {
              return shim.Error(err.Error())
      }

      // ==== Save marble private details ====
      objectType = "marblePrivateDetails"
      marblePrivateDetails := &marblePrivateDetails{objectType, marbleName, price}
      marblePrivateDetailsBytes, err := json.Marshal(marblePrivateDetails)
      if err != nil {
              return shim.Error(err.Error())
      }
      err = stub.PutPrivateData("collectionMarblePrivateDetails", marbleName, marblePrivateDetailsBytes)
      if err != nil {
              return shim.Error(err.Error())
      }
```

### 测试

{% asset_img 3.png private-data03 %}

{% asset_img 4.png private-data04 %}

### 在Fabric中使用私有数据
[官方文档搬运工](https://hyperledger-fabric.readthedocs.io/en/release-1.3/private_data_tutorial.html)s








