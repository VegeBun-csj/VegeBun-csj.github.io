---
title: "fabric1.4.x 网络部署的一些坑"
date: 2020-08-28T17:18:13+08:00
tags:
- 区块链
Categories:
- Hyperledger Fabric--网络部署
---

### 下面的搭建主要是基于fabric1.4.x出现错误

#### 1.创建通道出现错误(注意如果出现重复创建通道建议去清理网络)

```bash
config update for existing channel did not pass initial checks: implicit policy evaluation failed - 0 sub-policies were satisfied, but this policy requires 1 of the 'Writers' sub-policies to be satisfied: permission denied
```

这里是将在创建orderer的创世区块的channelID删除  使用默认值   注意这个**创世区块的channelID不能与下面的应用通道的channelID相同**，不然也会出现这个错误

#### 2.创建通道的错误

```bash
Error: failed to create deliver client: orderer client failed to connect to orderer.example.com:7050: failed to create new connection: context deadline exceeded
```

查看docker ps  发现order没有启动后挂了     

查看一下orderer的日志

```bash
2020-03-20 09:31:03.652 UTC [orderer.common.server] extractSysChanLastConfig -> INFO 003 Not bootstrapping because of 3 existing channels
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x10 pc=0xfa2509]

goroutine 1 [running]:
github.com/hyperledger/fabric/protos/utils.GetMetadataFromBlock(0x0, 0x1, 0x0, 0x194, 0x2140990)
        /opt/gopath/src/github.com/hyperledger/fabric/protos/utils/blockutils.go:55 +0x39
github.com/hyperledger/fabric/protos/utils.GetLastConfigIndexFromBlock(0x0, 0xc0001566c0, 0xffffffffffffffff, 0x0)
        /opt/gopath/src/github.com/hyperledger/fabric/protos/utils/blockutils.go:75 +0x37
github.com/hyperledger/fabric/orderer/common/multichannel.ConfigBlock(0x7fe3315d28d0, 0xc0001566c0, 0xc0001566c0)
        /opt/gopath/src/github.com/hyperledger/fabric/orderer/common/multichannel/registrar.go:112 +0x74
github.com/hyperledger/fabric/orderer/common/server.extractSysChanLastConfig(0x16c5420, 0xc000156320, 0xc0000c5fc0, 0x16c5420)
        /opt/gopath/src/github.com/hyperledger/fabric/orderer/common/server/main.go:222 +0x2cf
github.com/hyperledger/fabric/orderer/common/server.Start(0x15309fe, 0x5, 0xc000418900)
        /opt/gopath/src/github.com/hyperledger/fabric/orderer/common/server/main.go:110 +0x30a
github.com/hyperledger/fabric/orderer/common/server.Main()
        /opt/gopath/src/github.com/hyperledger/fabric/orderer/common/server/main.go:91 +0x208
main.main()
        /opt/gopath/src/github.com/hyperledger/fabric/orderer/main.go:15 +0x20
songjian@fabric-1:~/go/src/github.com/hyperledger/MedRec/MedNet$ docker exec -it Not bootstrapping because of 3 existing channels
Error: No such container: Not
songjian@fabric-1:~/go/src/github.com/hyperledger/MedRec/MedNet$ panic: runtime error: invalid memory address or nil pointer dereference
```

解决办法：重新配置order的docker的yaml文件中的volums映射 （20行错误显示因为已经存在3个通道，所以是通道残留，所以需要修改路径，或者删除你的本地路径中的文件）

```bash
  orderer.medrec.com:
    container_name: orderer.medrec.com
    extends:
      file: peer-base.yaml
      service: orderer-base
    volumes:
        - ../channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
        - ../crypto-config/ordererOrganizations/medrec.com/orderers/orderer.medrec.com/msp:/var/hyperledger/orderer/msp
        - ../crypto-config/ordererOrganizations/medrec.com/orderers/orderer.medrec.com/tls/:/var/hyperledger/orderer/tls
        - orderer.medrec.com:/var/hyperledger/production/orderer
```

在最后一行的映射，将本地映射重新换一个,我这里改成下面，重新部署即可

```ba
  - /var/hyperledger/order_data/orderer/chains/:/var/hyperledger/production/orderer
```

所以出现这种错误，是做了账本的本地持久化，下次启动时会同步，如果不这样做，可以在每次启动的时候将本地的映射删除

#### 3.peer加入通道错误

```bash
Error: proposal failed (err: rpc error: code = Unknown desc = chaincode error (status: 500, message: Cannot create ledger from genesis block, due to LedgerID already exists))
```

说明账本在本地已经存在，所以想一下，肯定是docker的yaml文件中做了本地地址的映射，所以每次启动容器时，上一次的账本同步到容器中去了，所以解决办法是，在启动网络是，不做创建通道和加入通道操作，或者，干脆不做本地与容器的映射。也就是将docker的yaml文件中的peer的volum删除

```bash
        - peer0.org1.medrec.com:/var/hyperledger/production
```

4.LOCALMSPID不匹配的错误

（注意检查configtx.yaml文件和docker的yaml文件中的localMSPID是否一致，一定要匹配，不然可能出现无法加入通道或者无法更新锚节点的问题，两者只有一个能成功）

4.创建应用通道时

```bash
Error: got unexpected status: BAD_REQUEST -- initializing configtx manager failed: bad channel ID: channel ID 'channelPA' contains illegal characters
```

原因：通道名不能大写！！！

#### 5.安装链码的时候出现路径不存在的错误（这是一个需要特别注意的地方，很容易出错）

```bash
Error: error getting chaincode code recordpa: path to chaincode does not exist: /opt/gopath/src/github.com/hyperledger/MedRec/chaincode/recordPA/go
```

注意：这种情况需要关注两个地方

（1）如果是使用命令行，部署链码需要关注docker-compose的yaml文件中cli容器的volumes的设置，链码的安装，实例化都是走cli的方式安装的，比如，我的cli的配置是这样

```bash
volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
```

看一下chancode的路径设置，前面是本地的chaincode路径，后面是容器中的chaincde路径，这里的容器中的路径是后面设置的关键

（2）关注一下我们安装链码的路径

```bash
	InfoCC_SRC_PATH=github.com/chaincode/info
```

注意书写方式，这里的路径是相对于容器中gopath的路径，所以是从github.com开始写链码的路径，同时注意，该路径和本地的chaincode路径中链码位置是否一致！！！！



