---
title: 'Hyperledger-Fabric建立第一个网络'
date: 2018-10-08 17:10:31
tags:
- Hyperledger-Fabric
toc: ture
---

### 总览

* 环境安装
* Hyperledger Fabric Samples 下载安装
* 建立网络
<!--more-->

## 环境安装
### 环境

本人系统使用的deepin，大多数所需要的命令环境等软件库直接安装即可。

```bash
$ sudo apt install ...
```

### 安装curl并配置https

Update the apt package index

```bash
$ sudo apt-get update
```

Install packages to allow apt to use a repository over HTTPS:

```bash
$ sudo apt-get install \apt-transport-https \ca-certificates \curl \software-properties-common

Add Docker’s official GPG key:

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Use the following command to set up the stable repository.

```bash
sudo add-apt-repository \"deb [arch=amd64] https://download.docker.com/linux/ubuntu \$(lsb_release -cs) \stable"
```

### 安装docker ce

有旧版本docker需要卸载旧版本

Update the apt package index.

```bash
$ sudo apt-get update
```

Install the latest version of Docker CE

```bash
$ sudo apt-get install docker-ce
```

install docker-ce=17.09.0~ce-0~ubuntu

```bash
$ sudo apt-get install docker-ce=17.09.0~ce-0~debian #看自己的linux发行版基于什么版本
```

run hello-world

```bash
$ sudo docker run hello-world
```

### 安装go

* 直接sudo apt install

* 解压下载的 go1.9.2.linux-amd64.tar.gz

```bash
sudo tar -xzf go1.9.2.linux-amd64.tar.gz -C /usr/local
```

配置全局变量

```bash
$ sudo gedit ~/.bashrc
```

添加如下代码

```bash
export GOPATH=/usr/local/go
export PATH=$GOPATH/bin:$PATH
```

重启配置文件

```bash
source ~/.bashrc
```

### 安装node

* 直接sudo apt install

* or
```bash
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 切换root

先设置 root 到密码:sudo passwd root ;
在控制台直接输入:su root ,并输入密码;

## Hyperledger Fabric Samples 下载安装
### 源码

```bash
$ git clone https://github.com/hyperledger/fabric-samples.git
```

### 二进制文件

下载文件解压至生成的fabric-samples中，1.0版本会生成一个bin文件，1.1及以上会在生成一个config文件

```bash
curl -sSL https://goo.gl/byy2Qj | bash -s 1.0.5 #需fq
```

[离线下载的网址](https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/)

将 hyperledger-fabric-linux-amd64-1.0.5.tar.gz 解压到fabric-samples
将[sh文件](https://goo.gl/byy2Qj)拷贝到samples根目录,注释掉 Downloading platform binaries模块，因为二进制文件已经手动下载完毕。

```bash
$ chmod -R 777 init.sh
$ ./init.sh 1.0.5
```

### docker加速
docker image 下载速度慢,配置代理，DaoCloud配置 docker 加速器，在 [DaoCloud](https://www.daocloud.io/) 注册并登录。登录到后台点击加速器

## 建立网络
### 更改samples版本

samples版本和二进制文件版本不同，要先修改版本

```bash
$ git tag      #查看可更改版本
$ git checkout v1.0.2  
```

### Generate Network Artifacts

首先切换root用户

```bash
$ cd first-network
$ ./byfn.sh -m generate
```

{% asset_img 1.png artifacts01 %}

{% asset_img 2.png artifacts02 %}

结果：

{% asset_img 3.png artifacts03 %}

{% asset_img 4.png artifacts04 %}

### 生生成初始区块

```bash
$ ../bin/cryptogen generate --config=./crypto-config.yaml
$ export FABRIC_CFG_PATH=$PWD
$ ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```

### 生生成应用用通道的配置信息

```bash
$ export CHANNEL_NAME=mychannel  #mychannel是链的名字
$ ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```

### 生生成锚节点配置更更新文文件

```bash
$ ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
$ ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
```

### 操作网络

编辑 docker-compose-cli.yaml ,注释到 command 命令

```bash
working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer 
#command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT' volumes
```

```bash
$ CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=600 docker-compose -f docker-compose-cli.yaml up -d
```

{% asset_img 5.png artifacts05 %}

## 创建和加入通道
### 进入 docker 容器

```bash
$ docker exec -it cli bash
```

### 创建通道

```bash
$ export CHANNEL_NAME=mychannel
$ peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

### 加入入通道

```bash
$ peer channel join -b mychannel.block
```

## 链上代码 (链码)
### 安装链码

```bash
$ peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
```

### 实例例化链码

```bash
$ peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR('Org1MSP.member','Org2MSP.member')"
```

### 查询

```bash
$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```
查询结果:

```bash
Query Result: 100
```

### 转账

```bash
$ peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'
```
查询 a 账户的金额:

```bash
$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```
结果:

```bash
Query Result: 90
```

### 问题记录
docker 运行时出错,可以查询 docker 正在运行的容器,删除运行的容器
查询正在运行的容器:

```bash
docker ps -a
```

删除运行的容器:

```bash
docker rm -f $(docker ps -aq)
```

清理网络：

```bash
./byfn.sh -m down
```

### 过程截图

{% asset_img 6.png guocheng01 %}

{% asset_img 7.png guocheng02 %}

{% asset_img 8.png guocheng03 %}

{% asset_img 9.png guocheng04 %}

{% asset_img 10.png guocheng05 %}

{% asset_img 11.png guocheng06 %}

{% asset_img 12.png guocheng07 %}

{% asset_img 13.png guocheng08 %}


