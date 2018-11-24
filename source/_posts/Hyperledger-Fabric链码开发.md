---
title: '[Hyperledger-Fabric链码开发]'
date: 2018-10-17 16:40:06
tags:
 - Hyperledger-Fabric
toc: ture
---

### 总览

* chaincode相关问题
* Hyperledger Fabric 链码的开发环境

<!--more-->

## chaincode相关问题
### 下载相应的包
需要引用[github项目](https://github.com/hyperledger/fabric.git)，需要下载到GOPATH中创建相关目录
从软件库中安装golang的GOPATH一般在/home/go/src/github.com

[chaincode002源码](https://github.com/SONGSONGBOBO/Hyperledger-Fabric/tree/master/chaincode/chaincode002s)

## Hyperledger Fabric 链码的开发环境
### 链码存放
将 chaincode002 复制到 fabric-samples/chaincode

### 开启 3 个终端
**终端 1-启动网网络**

```bash
$ cd chaincode-docker-devmode
$ docker-compose -f docker-compose-simple.yaml up
```

docker 启动出错:
删除所有的 docker 容器

```bash
$ docker rm -f $(docker ps -aq)
```

**终端 2- 编译且启动链码**

```bash
$ docker exec -it chaincode bash
$ cd chaincode002
$ go build
$ CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=mycc:0 ./chaincode002
```

**终端 3-操作链码**

```bash
$ docker exec -it cli bash
```

### 安装链码

```bash
$ peer chaincode install -p chaincodedev/chaincode/chaincode002 -n mycc -v 0
```

### 实例化链码

```bash
$ peer chaincode instantiate -n mycc -v 0 -c '{"Args":["str","helloworld"]}' -C myc
```

查询链码:

```bash
$ peer chaincode query -n mycc -c '{"Args":["get","str"]}' -C myc
```

运行结果:

```bash
Query Result: helloworld
```

调用链码:

```bash
peer chaincode invoke -n mycc -c '{"Args":["set", "str", "helloworld111"]}' -C myc
```

查询链码:

```bash
$ peer chaincode query -n mycc -c '{"Args":["get","str"]}' -C myc
```

运行结果:

```bash
Query Result: helloworld111
```

### 过程截图
**终端1**
{% asset_img 1.png environment01 %}
**终端2**
{% asset_img 2.png environment02 %}
**终端3**
{% asset_img 3.png environment03 %}

{% asset_img 4.png environment04 %}


