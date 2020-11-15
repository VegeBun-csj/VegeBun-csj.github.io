---
title: "链码常见的API"
date: 2020-10-18T21:24:00+08:00
tags:
- 区块链
- golang
Categories:
- Hyperledger Fabric--chaincode开发
---

### levelDB和CouchDB都支持的API

```go
//键的写入
PutState(key string, value []byte) error

//单个键的查询 
GetState(key string) ([]byte, error)

//键的范围查询(左闭右开，并且它的查询是最长匹配的原则！！！！！)
 GetStateByRange(startKey, endKey string)(StateQueryIteratorInterface, error)


//获取键的历史数据
GetHistoryForKey

//组合键的查询
 GetStateByPartialCompositeKey(objectType string, keys []string) (StateQueryIteratorInterface, error)
```

```
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarbles", "{\"selector\":{\"docType\":\"marble\"}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
```



```go
//couchdb的查询

//selector查询语句
GetQueryResult(queryString)

//分页查询
getQueryResultForQueryStringWithPagination(stub, queryString, int32(pageSize), bookmark)
```



