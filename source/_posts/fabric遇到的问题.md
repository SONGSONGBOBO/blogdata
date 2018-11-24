---
title: '[fabric遇到的问题]'
date: 2018-10-30 20:51:04
tags:
- Hyperledger-Fabric

toc : ture
---
### 总览
* 

<!--more-->

### 

```bash
ERRO 001 Cannot run peer because cannot init crypto, folder "/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" does not exist
```
下面的目录指向MSP的对等证书所在的位置。这些文件是使用../bin/cryptogen generate --config=./crypto-config.yaml教程中的命令生成的。错误消息是由于该目录不存在或包含所需的证书而导致对等方无法启动。我会确保对等容器在该位置具有那些证书，一种方式是Docker exec进入它。

/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

### Orderer Client failed to connect to orderer.example.com

Orderer Client failed to connect to orderer.example.com

do a docker ps and see if the orderer is up and running if not do a docker logs orderer.example.com and check orderers logs usually the orderer logs give clear clues on what has happened read the logs and fix the issue . that's what i did

and instead of creating a peer first going inside it and creating a channel ,i suggest you create the channel first and join the peer to it afterwards

Create the channel

docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.tracexyz.com/msp" peer0.org1.tracexyz.com peer channel create -o orderer.tracexyz.com:7050 -c cheeseproduction -f /etc/hyperledger/configtx/channel.tx

Add peer 0 to the channel

docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.tracexyz.com/msp" peer0.org1.tracexyz.com peer channel join -b cheeseproduction.block

Fetch from peer 1

docker exec -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.tracexyz.com/msp" peer1.org1.tracexyz.com peer channel fetch config -o orderer.tracexyz.com:7050 -c cheeseproduction

Join peer 1 also to the channel

docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.tracexyz.com/msp" -e "CORE_PEER_ADDRESS=peer1.org1.tracexyz.com:7061" peer0.org1.tracexyz.com peer channel join -b cheeseproduction.block

this is how i connected two peers to a single channel. my org name was tracexyz (tracexyz instead of example ) and my channel name was cheeseproduction

I guesss you can replace those with your own values

after doing these go inside the peer with

docker exec -it cli bash

it will take you inside the default peer which is peer0.org1

then do a peer channel list to see the channels to which peer0 has joined

you will see it will list cheeseproduction

### 测试链码时 Received error from server, ending chaincode stream: rpc error: code = Unimplemented desc = unknown service protos.ChaincodeSupport receive failed

在v1.1中，端口是7052而不是像在v1.0中那样是7051
