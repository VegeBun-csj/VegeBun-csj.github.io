---
title: "动态添加peer节点"
date: 2020-10-05T08:53:09+08:00
tags:
- 区块链
Categories:
- Hyperledger Fabric--网络部署
---



### 大致流程

- 生成节点证书文件(依然是用过crypto-config.yaml文件)
- 启动新节点的docker容器
- 节点加入通道
- 节点安装链码
- 节点端执行交易



### 以first-network网络为示例

- 修改**crypto-config.yaml**文件，如下(我们在org1组织中添加一个节点Peer2og1)


> 将Name为Org1的Template的数量更改为3（原来是2）

```yaml
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

# ---------------------------------------------------------------------------
# "OrdererOrgs" - Definition of organizations managing orderer nodes
# ---------------------------------------------------------------------------
OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer
  # ---------------------------------------------------------------------------
  - Name: Orderer
    Domain: example.com
    EnableNodeOUs: true
    Specs:
      - Hostname: orderer
      - Hostname: orderer2
      - Hostname: orderer3
      - Hostname: orderer4
      - Hostname: orderer5

PeerOrgs:
  # ---------------------------------------------------------------------------
  # Org1
  # ---------------------------------------------------------------------------
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true


    Template:
      Count: 3
    Users:
      Count: 1
  # ---------------------------------------------------------------------------
  # Org2: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org2
    Domain: org2.example.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
```

通过cryptogen工具生成peer2org1的证书文件

```shell
cryptogen extend --config=./crypto-config.yaml
```

可以在下面的目录中看到已经生成了新的peer节点(peer2.org1.example.com)的证书

![新增peer节点](/image/fabric/peer_dynamic.png)

- 编写新加入的节点peer2org1的yaml文件，以后序启动docker容器（以下是名为newpeer.yaml文件）

```yaml
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

volumes:
  peer2.org1.example.com:

networks:
  byfn:

services:
  peer2.org1.example.com:
    container_name: peer2.org1.example.com
    image: hyperledger/fabric-tools:latest
    environment:
      - CORE_PEER_ID=peer2.org1.example.com
      - CORE_PEER_ADDRESS=peer2.org1.example.com:12051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:12051
      - CORE_PEER_CHAINCODEADDRESS=peer2.org1.example.com:12052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:12052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer2.org1.example.com:12051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      # the following setting starts chaincode containers on the same
      # bridge network as the peers
      # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_byfn
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer2.org1.example.com/msp:/etc/hyperledger/fabric/msp
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer2.org1.example.com/tls:/etc/hyperledger/fabric/tls
        - peer2.org1.example.com:/var/hyperledger/production
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 12051:12051
    networks:
      - byfn

  cli_new_peer:
    container_name: cli_new_peer
    image: hyperledger/fabric-tools:latest
    tty: true
    stdin_open: true
    environment:
      - SYS_CHANNEL=$SYS_CHANNEL
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      #- FABRIC_LOGGING_SPEC=DEBUG
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer2.org1.example.com:12051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer2.org1.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer2.org1.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer2.org1.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - peer2.org1.example.com
    networks:
      - byfn
```

> 注意节点不要写错，以及端口号等，路径设置，该文件我是直接放在first-network目录下的

启动节点容器(peer2org1，以及对应的一个cli容器用于进行链码的安装和调用)

```shell
docker-compose -f newpeer.yaml up -d
```

- 新节点加入通道（注意此时的cli_new_peer的容器内并没有mychannel.block应用通道的区块，所以需要将之前first-network网络创建的cli容器中的mychannel.block拷贝到cli_new_peer容器中）

```shell
#将cli容器中的mychannel.block拷贝到宿主机
docker cp cli容器id::/opt/gopath/src/github.com/hyperledger/fabric/peer/mychannel.block ./

#将宿主机中mychannel.block拷贝到cli_new_peer容器中
docker cp mychannel.block cli_new_peer容器id: :/opt/gopath/src/github.com/hyperledger/fabric/peer

#进入cli容器，准备加入通道
docker exec -it cli_new_peer bash

ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

#新节点加入通道
peer channel join -b mychannel.block

#新节点安装链码
peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/

#新节点查询交易(此时是第一次调用链码，所以会创建链码容器，需要等待一会)
peer chaincode query -C mychannel -n mycc -c '{"function":"query","Args":["a"]}'
#查询结果为90

#新节点调用交易
peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","40"]}'

#再次查询
peer chaincode query -C mychannel -n mycc -c '{"function":"query","Args":["a"]}'
#查询结果为50
peer chaincode query -C mychannel -n mycc -c '{"function":"query","Args":["b"]}'
#查询结果为250
```



> 可以看出来动态加入peer节点的操作还是比较简单的，主要就是生成节点的证书文件，然后编写新节点的容器文件，然后新节点加入通道，再安装链码，调用交易，整个流程还是比较清晰
>
> 但是有几个注意点：
> 1.新节点的yaml文件编写需要特别注意，我在实际编写过程中盲目参考了网上的一些文章，出现了比较难以解决的问题，主要是yaml文件中的networks需要和之前的first-network中的networks字段保持一致，之前一直出现节点中的gossip通信问题，一直未得到解决，后来发现是因为docker的yaml文件中的networks字段不一致导致的。
>
> ```shell
> 2020-10-03 07:25:31.119 UTC [gossip.discovery] func1 -> WARN 04d Could not connect to Endpoint: peer0.org2.example.com:9051, InternalEndpoint: peer0.org2.example.com:9051, PKI-ID: <nil>, Metadata:  : context deadline exceeded
> 2020-10-03 07:25:36.152 UTC [ConnProducer] NewConnection -> ERRO 04e Failed connecting to {orderer.example.com:7050 [OrdererMSP]} , error: context deadline exceeded
> 2020-10-03 07:25:36.153 UTC [ConnProducer] NewConnection -> ERRO 04f Could not connect to any of the endpoints: [{orderer.example.com:7050 [OrdererMSP]}]
> 2020-10-03 07:25:36.153 UTC [deliveryClient] connect -> ERRO 050 Failed obtaining connection: could not connect to any of the endpoints: [{orderer.example.com:7050 [OrdererMSP]}]
> 2020-10-03 07:25:36.153 UTC [deliveryClient] try -> WARN 051 Got error: could not connect to any of the endpoints: [{orderer.example.com:7050 [OrdererMSP]}] , at 5 attempt. Retrying in 16s
> 
> ```
>
> 
>
> volume字段的一些本地文件映射一定要解和自己的实际目录填写，切不可照搬！！！
>
> 注意：
>
> CORE_PEER_GOSSIP_EXTERNALENDPOINT是写自己的域名以及端口
>
> CORE_PEER_GOSSIP_BOOTSTRAP是写同组织中的节点(不要搞混淆了)
>
>       - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051
>       - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer2.org1.example.com:12051
>
> 2.加入通道的时候，因为新创建的cli_new_peer容器中并没有应用通道mychannel.block文件，所以需要从cli容器中拷贝过来！！！
>
> 当然也可以不进行拷贝！！！！！（放大招！！！！）：
>
> ```shell
> ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
> 
> peer channel fetch 0 mychannel.block -o orderer.example.com:7050 -c mychannel --tls --cafile $ORDERER_CA
> ```
>
> 在cli_new_peer容器中执行以上命令，向orderer进行获取通道的第0个区块，也就是应用通道的创世区块，一样也能获得mychannel.block