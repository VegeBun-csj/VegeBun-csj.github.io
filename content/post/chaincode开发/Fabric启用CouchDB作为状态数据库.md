---
title: "Fabric启用CouchDB作为状态数据库"
date: 2020-10-18T21:22:31+08:00
tags:
- 区块链
- CouchDB
Categories:
- Hyperledger Fabric--chaincode开发
---

# Fabric中启用CouchDB

### 使用CouchDB的注意点

1. 当chaincode的数据模型被构建为`Json`格式的时候，可以使用CouchDB，因为它支持丰富的查询，通过`建立索引`，基于内容查询而不仅仅是基于Key的查询

   > couchDB是一种以json文档存储的数据库，而不是单一的键值数据库

2. 如果要启用CouchDB，那么就要在启动网络之前就配置完成，不能将已经启动levelDB的peer转为启用CouchDB，如果这样就会存在数据的不一致，所有的节点，要么全部使用CouchDB,要么全部使用LevelDB，还有就是如果启用CouchDB，那么一个peer节点使用一个CouchDB，同时这两个最好在同一个服务器上！！！

3. 如果同时有json数据和二进制数据，仍然可以启用CouchDB，但是二进制的数据智能通过单一的key、key的范围查询，以及组合键的查询



### CouchDB的配置以及知识点

如果要启用CouchDB，需要设置peer容器的yaml上加几行配置即可，同时需要有`docker-compose-couchdb.yaml`类似的用于启动couchdb容器的文件

```yaml
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

networks:
  byfn:

services:
  couchdb0:
    container_name: couchdb0
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    # Comment/Uncomment the port mapping if you want to hide/expose the CouchDB service,
    # for example map it to utilize Fauxton User Interface in dev environments.
    ports:
      - "5984:5984"
    networks:
      - byfn

  peer0.org1.example.com:
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    depends_on:
      - couchdb0

  couchdb1:
    container_name: couchdb1
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    # Comment/Uncomment the port mapping if you want to hide/expose the CouchDB service,
    # for example map it to utilize Fauxton User Interface in dev environments.
    ports:
      - "6984:5984"
    networks:
      - byfn

  peer1.org1.example.com:
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    depends_on:
      - couchdb1

  couchdb2:
    container_name: couchdb2
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    # Comment/Uncomment the port mapping if you want to hide/expose the CouchDB service,
    # for example map it to utilize Fauxton User Interface in dev environments.
    ports:
      - "7984:5984"
    networks:
      - byfn

  peer0.org2.example.com:
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb2:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    depends_on:
      - couchdb2

  couchdb3:
    container_name: couchdb3
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    # Comment/Uncomment the port mapping if you want to hide/expose the CouchDB service,
    # for example map it to utilize Fauxton User Interface in dev environments.
    ports:
      - "8984:5984"
    networks:
      - byfn

  peer1.org2.example.com:
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb3:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    depends_on:
      - couchdb3

```

可以看到，核心的就是几行：

```shell
  - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
  - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb2:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
  - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
  - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
```



### 创建索引

通常建立index是非常方便的，如果一个查询需要进行排序，那么在CouchDB中需要有一个排序的索引，不然会报错

一般情况下，**一个chaincode中对应一个`doctype`(通常这个字段是必须的！！！！),用于标识一个chaincode(标识一类状态数据)**，一个chaincode代表一个couchdb文档，通过`docType`来区分不同的chaincode数据（比如：有的chaincode描述的是marble数据，有的是描述的dog数据，有的是描述car的数据，这样的话它们的`docType`就要不一样，以便进行区分）

```go
type marble struct {
         ObjectType string `json:"docType"` //docType is used to distinguish the various types of objects in state database
         Name       string `json:"name"`    //the field tags are needed to keep case from bouncing around
         Color      string `json:"color"`
         Size       int    `json:"size"`
         Owner      string `json:"owner"`
}
```

当建立索引以便chaincode进行查询的时候，要以couchDB的json文件格式建立索引

基本的格式就是：

```json
{
    "index":{
        "fields":["...","..."]	#建立索引的字段
    },
    "ddoc":"...DOC"	#可选的，但是建议加上，而且看官方的写法都是使用“索引名+DOC”
    "name":"索引名",
    "type":"json"#注意这个type是固定的，在Fabric中只支持json类型的couchdb索引
    
}
```

例如下面:

```json
  "index":{
      "fields":["owner"] // Names of the fields to be queried
  },
  "ddoc":"index1Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index1",
  "type":"json"
}

{
  "index":{
      "fields":["owner", "color"] // Names of the fields to be queried
  },
  "ddoc":"index2Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index2",
  "type":"json"
}

{
  "index":{
      "fields":["owner", "color", "size"] // Names of the fields to be queried
  },
  "ddoc":"index3Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index3",
  "type":"json"
}
```

> ddoc属性（design document，设计文档），可以清除地辨别用的那个索引进行的查询，如果没有指定名称，会被自动创建
>
> 从上面的索引可以看出：
>
> **一个Index可以由1个或者多个字段进行指定，一个字段可以出现在多个Index中**

总体来说，index的fields字段应该匹配一些字段，可以用于查询或者排序；Fabric将索引数据文档称为"`index warming`"



### 将索引添加到链码文件夹

- 当时用SDK安装和实例化链码的时候，可以自定义索引的目录

- 当使用CLI命令行安装和实例化链码的时候，必须将索引的json文件放在链码的同级目录下，并且目录结构必须为`META-INF/statedb/couchdb/indexes`

  ![image-20201011205454539](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20201011205454539.png)

在marbles的例子中，使用了索引，如下：

```json
{"index":{"fields":["docType","owner"]},"ddoc":"indexOwnerDoc", "name":"indexOwner","type":"json"}
```



### 启动网络

使用first-network示例：

```shell
./byfn.sh up -c mychannel -s couchdb
```



### 安装并实例化链码

```shell
docker exec -it cli bash
#安装
peer chaincode install -n marbles -v 1.0 -p github.com/chaincode/marbles02/go
#实例化
export CHANNEL_NAME=mychannel
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.peer','Org1MSP.peer')"
```



### 验证索引是否部署

可以通过peer节点的日志查看

```shell
docker logs peer0.org1.example.com  2>&1 | grep "CouchDB index"
```

输出如下即代表索引部署成功：

```shell
[couchdb] CreateIndex -> INFO 0be Created CouchDB index [indexOwner] in state database [mychannel_marbles] using design document [_design/indexOwnerDoc]
```



### 查询CouchDB状态数据库

建议在查询的时候指定参数，使用`use_index`字段指定`index name`，不然查询会很慢

因为index在chaincode部署的时候被部署，当使用chaincode进行查询的时候，索引会自动使用，也就是说如果不指定那个索引，CouchDB也会在索引文件中寻找使用哪个索引，但是这个自动检测也是需要时间的，所以建议指定索引的名称进行查询



### 在chaincode上构建一个查询

> selector查询语法相关文档
>
> https://docs.couchdb.org/en/stable/api/database/find.html

首先在账本中创建一个owner为`tom`的marble，以供后面查询

```shell
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
```

然后使用couchDB进行查询

```shell
peer chaincode query -C $CHANNEL_NAME -n marbles -c 
'{"Args":["queryMarbles", 
"{\"selector\":{\"docType\":\"marble\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}"
]}'
```

其中有**三个参数**：

- `queryMarbles`：这是chaincode中的方法
- `{"selector":{"docType":"marble","owner":"tom"}`：这是查询地querystring,查询owner为tom的所有的marble
- `"use_index":["_design/indexOwnerDoc", "indexOwner"]`:这里就是指定使用哪种索引，是一个数组，其中第一个是ddoc,第二个是index的name

查询结果为：

```shell
[{"Key":"marble1", "Record":{"color":"blue","docType":"marble","name":"marble1","owner":"tom","size":35}}]
```



### 查询和索引(queries and indexes)的最佳实践

使用索引的查询非常快，不需要扫描整个CouchDB数据库，了解索引将使您能够编写查询以提高性能，并帮助您的应用程序处理网络上的大量数据或数据块。

将索引和chaincode代码一起安装非常重要，每个链码只需要安装几个索引就能满足绝大部分的查询需求，`添加太多索引，或在索引中使用过多字段，都会降低网络性能`,这是因为在提交每个区块之后都会更新索引。

通过“index warming”更新的索引越多，完成事务所需的时间就越长。

下面的实例中会演示如何使用查询索引，以及什么样的索引会展现最好的性能，在编写查询语句的时候要记住以下几点：

- 索引中的所有字段也必须在查询的选择器selector或排序部分 sort sections中，才能使用索引。
- 太复杂的索引会降低网络的性能，并且也不大可能使用索引
- 应尽量避免使用全表扫描或全索引扫描的运算符，例如`$or`, `$in` and `$regex`等

之前使用的Marbles中的查询：

```shell
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarbles", "{\"selector\":{\"docType\":\"marble\",\"owner\":\"tom\"}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```

Marbles的链码同时被安装了 `indexOwnerDoc`索引

```shell
{"index":{"fields":["docType","owner"]},"ddoc":"indexOwnerDoc", "name":"indexOwner","type":"json"}
```

可以注意到，上面的索引的查询字段中有两个参数`docType`和`owner`，这就是一个**索引支持的查询(query fully supported by the index)**，表明该查询将能够使用索引中的数据，而不必搜索整个数据库，像这样的fully Supported queries**比链码中的其他查询返回得更快**。

> 所谓的query fully supported by the index，其实可以简单(不恰当地)地理解为查询条件字段和索引中字段一致

如果在上面得查询中增加一个字段，这个查询仍然会使用这个索引，但是这个索引会额外地去查询索引中的其他字段，这就会产生延长的响应时间

举个例子，下面这个查询仍然会使用上面的索引，但是查询会比之前的查询慢一会

```shell
#Example two: query fully supported by the index with additional data

peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarbles", "{\"selector\":{\"docType\":\"marble\",\"owner\":\"tom\",\"color\":\"red\"}, \"use_index\":[\"/indexOwnerDoc\", \"indexOwner\"]}"]}'
```

可以看到这个查询中不光查询了owner为tom的，还要同时查到color为red的，而color字段是上面建立的索引中没有的，所以此时就需要全局扫描数据库

再有一个例子，下面的查询是搜索owner,但是没有指定它拥有的东西的类型，因为我们上面上面建立的索引是包含了`docType`和`owner`两个字段的(也就是我们建立的这个索引是针对的marbles这个产品的索引，至于owner是否还拥有其他的产品就不知道了，所以需要全局扫描)，这样在查找的时候，并不会使用上面的索引，而是采用全局的扫描

```shell
#Example three: query not supported by the index

peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarbles", "{\"selector\":{\"owner\":\"tom\"}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```

总的来说，越复杂的查询需要耗费更多的时间，并且使用索引进行查询的几率会小很多，像`$or`, ``$in` , `$regex`这几个操作符通常都是全局扫描，并不会使用索引，所以当数据量很大的时候，尽量避免使用它们

举个例子，下面的查询会包含一个`$or`操作符来查找每一个`marbles`并且它的拥有者为`tom`

```shell
# Example four: query with $or supported by the index

peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarbles", "{\"selector\":{"\$or\":[{\"docType\:\"marble\"},{\"owner\":\"tom\"}]}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```

这个查询中的查询字段都在索引中，所以会使用索引查询，但是因为`$or`操作符的原因，要求全局扫描进行查询，从而导致了更长的查询时间。

下面再举一个例子，这是一个复杂的查询，并没有使用索引进行查询

```shell
# Example five: Query with $or not supported by the index

peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarbles", "{\"selector\":{"\$or\":[{\"docType\":\"marble\",\"owner\":\"tom\"},{"\color\":"\yellow\"}]}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```

这个查询会寻找所有owner为tom的Marbles的或者所有的黄色物品(不仅是marbles)，这个查询不会使用索引，因为它会扫描整个表来达成`$or`的条件，如果说账本上有非常多的数据，这种情况下可能会消耗大量的时间，甚至是超时

尽管上面介绍了几个查询的注意点，但是索引并不是大数据量查询的解决方式，区块链的数据结构是为了优化验证和确认交易，并不适合直接用来进行数据分析，如果想要使用一个仪表盘来对应用程序中的数据进行分析，比较好的方式是采用链下数据库(链上数据的副本)，其实这也是从另一个角度来思考区块链的性能问题，可以数据上链，然后查询本地数据库

> 这里抛出一个疑问：既然到了链下，那链下数据被篡改了怎么办？？？？虽然性能提升了，但是安全系数降低了

通常可以在应用程序中使用`区块/链码的事件`，将交易数据写到链下的数据库中，对于每一个被提交的区块，区块监听器需要遍历区块中的所有的交易，读取它们的读写集，并通过键值对的方式中构建一个数据存储，SDK的事件机制提供了可重放的事件，确保数据存储的完整性，所以可以通过事件的监听，将区块链上的数据写入到外部的数据库中，可以在fabric-samples中的[Off chain data sample](https://github.com/hyperledger/fabric-samples/tree/master/off_chain_data)中找到实例



### 利用CouchDB进行分页查询

当查询的结果数据集合比较大的时候，需要对查询结果进行分页，通过分页机制可以将查询的结果通过特定的页面大小（`pagesize`，一页可以展示的数据条数）和书签（`bookmark`）来进行显示(类似于书的页数和一页可以展示的内容)，客户端程序会重复调用chaincode进行查询，直到没有数据可以返回，更详细的信息，可以参考 [topic on pagination with CouchDB](http://hyperledger-fabric.readthedocs.io/en/master/couchdb_as_state_database.html#couchdb-pagination).

使用marbles案例中的`queryMarblesWithPagination`来展示分页功能，

首先需要好几个数据才能展示，下面添加几个数据

```shell
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","25","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","yellow","35","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble4","green","20","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble5","purple","20","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble6","blue","40","tom"]}'
```

除了之前的例子中的查询参数以外，这个分页的函数`queryMarblesWithPagination`，还需要传入`pagesize`和`bookmark`；`pagesize`指定每次(页)查询返回的数据量，`bookmark`是一个锚点，告诉couchDB从哪里开始新的一页（每一页的结果都会返回一个`bookmark`）

```go
getQueryResultForQueryStringWithPagination(stub, queryString, int32(pageSize), bookmark)
```

查询分页的函数中调用的是`getQueryResultForQueryStringWithPagination`方法，同时传入了一页的大小和书签标记。

下面的一个例子演示调用这个分页的函数，指定`pagesize`大小为`3`，不指定`bookmark`

> 如果不指定`bookmark`,那么查询会默认从第一页的记录开始

```shell
# Rich Query with index name explicitly specified and a page size of 3:

peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesWithPagination", "{\"selector\":{\"docType\":\"marble\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3",""]}'
```

下面就是返回的数据

```shell
[{"Key":"marble1", "Record":{"color":"blue","docType":"marble","name":"marble1","owner":"tom","size":35}},
{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"tom","size":25}},
{"Key":"marble3", "Record":{"color":"yellow","docType":"marble","name":"marble3","owner":"tom","size":35}}]

[{"ResponseMetadata":{"RecordsCount":"3", "Bookmark":"g1AAAABLeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqz5yYWJeWkGoOkOWDSOSANIFk2iCyIyVySn5uVBQAGEhRz"}}]
```

可以看到查询到三条数据（但是我们前面的弹珠并不止3个）

> 注意：查询返回的不止查询的数据，couchdb还会为每一次查询生成一个唯一的`bookmark`的hash值，它也包含在结果集中，通过传递返回的这个`bookmark`的值来实现下一页的查询

下面的查询就是通过调用分页查询，`pagesize`置为3，同时`bookmark`是从上一次返回的那个hash值开始的

```shell
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesWithPagination", "{\"selector\":{\"docType\":\"marble\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3","g1AAAABLeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqz5yYWJeWkGoOkOWDSOSANIFk2iCyIyVySn5uVBQAGEhRz"]}'
```

查询的就是剩下来的Marbles数据：

```shell
[{"Key":"marble4", "Record":{"color":"green","docType":"marble","name":"marble4","owner":"tom","size":20}},
{"Key":"marble5", "Record":{"color":"purple","docType":"marble","name":"marble5","owner":"tom","size":20}},
{"Key":"marble6", "Record":{"color":"blue","docType":"marble","name":"marble6","owner":"tom","size":40}}]

[{"ResponseMetadata":{"RecordsCount":"3", "Bookmark":"g1AAAABLeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqz5yYWJeWkmoGkOWDSOSANIFk2iCyIyVySn5uVBQAGihR2"}}]
```

下面再继续调用分页查询，`pagesize`依然为`3`,`bookmark`才能过上一次开始

```shell
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesWithPagination", "{\"selector\":{\"docType\":\"marble\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3","g1AAAABLeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqz5yYWJeWkmoGkOWDSOSANIFk2iCyIyVySn5uVBQAGihR2"]}'
```

查询结果为：

```shell
[]
[{"ResponseMetadata":{"RecordsCount":"0", "Bookmark":"g1AAAABLeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqz5yYWJeWkmoGkOWDSOSANIFk2iCyIyVySn5uVBQAGihR2"}}]
```

可以看到已经没有数据可以查询了

> 上面的例子就是演示了客户端如何通过结果集来进行分页



### 更新索引

随着时间的推移，很有必要更新索引，在安装链码的后续版本中，可能会存在相同的索引，为了更新索引，原来的索引定义中必须包含设计文档`ddoc`属性和`索引名称index name`。

如果要更新索引，还是使用原来的索引名称，但是需要对内容定义进行修改，编辑索引的Json文件，从索引中增加或者删除一些字段

> 注意：Fabric中只支持Json类型的索引，其他类型不支持

更新的索引定义会在chaincode被安装和实例化的时候部署，改变`索引的名称index name`或者`ddoc`会创建出新的索引，而原来的索引仍会保留，直到被删除

> 如果状态数据库中有大量数据，则重建索引将花费一些时间，在此期间，链码调用会导致查询失败或超时。



### 重复定义索引

在开发环境中，可以尝试不同的索引来支持链码查询，但是，对链码的任何更改都需要重新部署。可以使用 [CouchDB Fauxton interface](http://docs.couchdb.org/en/latest/fauxton/index.html) 接口来自定义创建和更新索引

> The Fauxton interface提供了一个web界面来创建、更新、部署CouchDB的索引，可以在浏览器中访问`http://localhost:5984/_utils`查看，当然也可以通过curl命令



### 删除索引

索引的定义并不受fabric的工具管理，可以通过curl命令或者 the Fauxton interface图形化界面删除索引

删除索引的curl命令格式为：

```shell
curl -X DELETE http://localhost:5984/{database_name}/_index/{design_doc}/json/{index_name} -H  "accept: */*" -H  "Host: localhost:5984"
```

例如删除本示例中的索引就是：

```shell
curl -X DELETE http://localhost:5984/mychannel_marbles/_index/indexOwnerDoc/json/indexOwner -H  "accept: */*" -H  "Host: localhost:5984"
```



下面来动手实践一下，体验一下CouchDB的查询快感！！！！

我们增加一个Color的索引，在chaincode/marbles02/go/META-INF/statedb/couchdb/indexes目录下面

![image-20201011200034730](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20201011200034730.png)

我们仿照原来的`indexOwner.json`写一个`indexColor.json`

```json
{
    "index":{
      "fields":[
        "docType",
        "color"]
    },
    "ddoc":"indexColorDoc",
    "name":"indexColor",
    "type":"json"
  }
```

这样就直接在容器中产生映射了，可以在命令行中进行调用

我们来查询所有的颜色为green的marbles

```
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarbles", "{\"selector\":{\"docType\":\"marble\",\"color\":\"green\"}, \"use_index\":[\"_design/indexColorDoc\", \"indexColor\"]}"]}'
```

结果为

```shell
[{"Key":"marble4", "Record":{"color":"green","docType":"marble","name":"marble4","owner":"tom","size":20}}]
```

和预期的一样！！！这样我们就可以不局限于Key进行查找了，可以进行很多形式的查询！

> 注意：一个索引一个json文件
