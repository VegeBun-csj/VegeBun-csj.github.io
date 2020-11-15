---
title: "Fabric交易流程的详细分析"
date: 2020-10-11T21:14:08+08:00
tags:
- 区块链
Categories:
- Hyperledger Fabric--基本概念
---

# Fabric交易流程的详细分析

> 总体来讲，一个交易流程分为3个阶段：交易的模拟执行、交易打包生成区块、账本的更新

交易流程图：

![image-20201011092109711](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/image-20201011092109711.png)

## 第一阶段：	

##### 客户端创建交易提案并提交(称为`SignedProposal`，是被客户端签名过的提案)

`SignedProposal`的结构如下：

```json
SignedProposal:{
    ProposalBytes(Proposal):{
        Header:{
            ChannelHeader:{
                Type:"HeaderType_ENDORSER_TRANSACTION",
                TxID:TxId,
                Timestamp:Timestamp,
                ChannelId:ChannelId,
                Extension(ChaincodeHeaderExtension):{
                    PayloadVisibility:PayloadVisibility,
                    ChaincodeId:{
                        Path:Path,
                        Name:Name,
                        Version:Version
                    }
                },
                Epoch:Epoch
            },
            SignatureHeader:{
                Creator:Creator,
                Nonce:Nonce
            }
        },
        Payload:{
            ChaincodeProposalPayload:{
                Input(ChaincodeInvocationSpec):{
                    ChaincodeSpec:{
                        Type:Type,
                        ChaincodeId:{
                            Name:Name
                        },
                        Input(ChaincodeInput):{
                            Args:[]
                        }
                    }
                },
                TransientMap:TransientMap
            }
        }
    },
    Signature:Signature
}
```

所谓的`SignedProposal`顾名思义就是`proposal + Signature`

> proposal就是要发送的提案，Signature是客户端的签名，两个加在一起就叫客户端签名的提案！！

`Proposal`由`Header+Payload`，

- `Header = ChannelHeader + SignatureHeader`(Header里面的内容主要和提案创建者身份)

`ChannelHeader`: 包含了`channel`和`chaincode`调用相关的信息, 如在哪个channel上调用哪个`chaincode`; `TxID`是客户端生成的交易号, 跟客户端的身份证书相关, 可以`避免交易号的冲突`, 背书节点和提交节点都会验证是否重复交易

`SignatureHeader`: 包含客户端的身份证书和一个随机数, 用于验证消息的有效性.

- `payload`：其中主要是链码调用传入的参数

> 客户端构造好签名提案交易消息, 选择背书节点并进行背书签名(背书节点在背书策略中指定),通过调用背书节点的`ProposalProcess()`接口，提交签名提案消息到背书节点，请求进行模拟执行

---

---

##### 背书节点模拟执行交易并进行背书签名

> 背书节点收到签名提案消息之后，其实也会分两个阶段进行处理，即 验证+模拟执行

背书节点对签名提案消息进行验证

1. 签名提案的格式是否正确

   > 主要就是检查签名提案中的Payload的格式，包括channelHeader、SignatureHeader、交易ID、以及消息扩展域中的PayloadVisibility可见性模式和ChannelId(路径、名称、版本)

2. 签名提案的签名的合法性，即是否有效(通过MSP)

   > 主要就是检查签名提案中的Signature域，是否是合法的MSP成员身份

3. 签名提案的提交者是否满足channel的ACL(即在当前channel是否已被授权有写权限/Channel/Application/Writers，这是在通道配置文件中定义)

   > 

4. 交易是否提交过

   > 通过签名提案中的随机数

背书节点模拟执行

- 背书节点会启动链码容器以模拟执行交易提案，并将模拟执行的结果暂存在`交易模拟器`中，等待orderer节点的排序和最后的committer peer的交易验证，而不是直接更新账本；其中的`交易模拟执行的结果`是以一种状态数据`读写集(读数据的键和版本，以及写数据的键)`来记录交易造成的结果变更。
- 调用ESCC(背书系统链码)对提案消息的模拟执行的结果的`读写集进行签名背书`

背书节点基于背书信息和模拟执行的结果构造`提案响应消息（ProposalResponse）`

`ProposalResponse`的结构如下：

```json
ProposalResponse:{
    Version:Version,
    Timestamp:Timestamp,
    Response:{
        Status:Status,
        Message:Message,
        Payload:Payload
    },
    Payload(ProposalResponsePayload):{
        ProposalHash:ProposalHash,
        Extension(ChaincodeAction):{
            Results(TxRwSet):{
                NsRwSets(NsRwSets):[
                    NameSpace:NameSpace,
                    KvRwSet:{
                        Reads(KVRead):[
                            Key:Key,
                            Version:{
                                BlockNum:BlockNum,
                                TxNum:TxNum
                            }
                        ],
                        RangeQueriesInfo(RangeQueriesInfo):[
                            StartKey:StartKey,
                            EndKey:EndKey,
                            ItrExhausted:ItrExhausted,
                            ReadsInfo:ReadsInfo
                        ],
                        Writes(KVWrite):[
                            Key:Key,
                            IsDelete:IsDelete,
                            Value:Value
                        ]
                    }
                ]
            },
            Events(ChaincodeEvent):{
                ChaincodeId:ChaincodeId,
                TxId:TxId,
                EventName:EventName,
                Payload:Payload
            }
            Response:{
                Status:Status,
                Message:Message,
                Payload:Payload
            },
            ChaincodeId:ChaincodeId
        }
    },
    Endorsement:{
        Endorser:Endorser,
        Signature:Signature
    }
}
```

`ProposalResponse = Version + Timestamp + Response + Payload + Endorsement`

其中主要的就是`payload`中的读写集以及`Endorsement`中的背书节点签名



##### 客户端收集背书节点回复的提案响应消息

客户端收集到`ProposalResponse`后

- 对其中的`背书节点签名进行验证`，验证该提案响应消息的合法性
- 判断自己是否收集到足够多的符合要求的背书签名提案响应消息

> 如果`chaincode`只进行账本query查询, 客户端只检查查询响应, 而不将交易提交给Orderer节点; 
>
> 如果`chaincode`对账本进行Invoke调用, 则客户端先判断是否满足背书策略(如果客户端未收集到足够的背书就提交了交易, `Committer`节点会在提交验证阶段发现交易不能满足背书策略, 将会标记为无效交易), 然后将交易提交给Orderer节点排序打包出块, 等待后序的账本更新.



##### 客户端构造交易请求，发送给Orderer节点进行排序打包出块

在收集到足够多的背书签名提案响应之后，确认所有的背书节点执行结果一致之后，客户端会基于`交易提案, 提案响应(包含了背书签名)`来构造合法的`签名交易消息(Envelope类型)`,通过Orderer节点提供的Broadcast()服务接口将该消息发送给Orderer节点，请求交易的排序处理（注意：配置交易不需要经过背书节点的处理！！！）

`Envelope`签名交易消息的数据结构：

```json
Envelope:{
    Payload:{
        Header:{
            ChannelHeader:{
                Type:"HeaderType_ENDORSER_TRANSACTION",
                TxId:TxId,
                Timestamp:Timestamp,
                ChannelId:ChannelId,
                Extension(ChaincodeHeaderExtension):{
                    PayloadVisibility:PayloadVisibility,
                    ChaincodeId:{
                        Path:Path,
                        Name:Name,
                        Version:Version
                    }
                },
                Epoch:Epoch
            },
            SignatureHeader:{
                Creator:Creator,
                Nonce:Nonce
            }
        },
        Data(Transaction):{
            TransactionAction:[
                Header(SignatureHeader):{
                    Creator:Creator,
                    Nonce:Nonce
                },
                Payload(ChaincodeActionPayload):{
                    ChaincodeProposalPayload:{
                        Input(ChaincodeInvocationSpec):{
                            ChaincodeSpec:{
                                Type:Type,
                                ChaincodeId:{
                                    Name:Name
                                },
                                Input(ChaincodeInput):{
                                    Args:[]
                                }
                            }
                        },
                        TransientMap:nil
                    },
                    Action(ChaincodeEndorsedAction):{
                        Payload(ProposalResponsePayload):{
                            ProposalHash:ProposalHash,
                            Extension(ChaincodeAction):{
                                Results(TxRwSet):{
                                    NsRwSets(NsRwSet):[
                                        NameSpace:NameSpace,
                                        KvRwSet:{
                                            Reads(KVRead):[
                                                Key:Key,
                                                Version:{
                                                    BlockNum:BlockNum,
                                                    TxNum:TxNum
                                                }
                                            ],
                                            RangeQueriesInfo(RangeQueryInfo):[
                                                StartKey:StartKey,
                                                EndKey:EndKey,
                                                ItrExhausted:ItrExhausted,
                                                ReadsInfo:ReadsInfo
                                            ],
                                            Writes(KVWrite):[
                                                Key:Key,
                                                IsDelete:IsDelete,
                                                Value:Value
                                            ]
                                        }
                                    ]
                                },
                                Events(ChaincodeEvent):{
                                    ChaincodeId:ChaincodeId,
                                    TxId:TxId,
                                    EventName:EventName,
                                    Payload:Payload
                                }
                                Response:{
                                    Status:Status,
                                    Message:Message,
                                    Payload:Payload
                                },
                                ChaincodeId:ChaincodeId
                            }
                        },
                        Endorsement:[
                            Endorser:Endorser,
                            Signature:Signature
                        ]
                    }
                }
            ]
        }
    },
    Signature:Signature
}
```

可以看到这个签名交易消息Envelope也是顾名思义，即签名过的交易消息

`Envelope = payload + Signature`

> payload就是交易消息的主体内容，Signature是客户端对payload的签名

```shell
payload = Header + Data 

Header = ChannelHeader + SignatureHeader 

Data = TransactionAction数组

TransactionAction = Header(SignatureHeader) + payload(ChaincodeActionPayload)

Payload(ChaincodeActionPayload) = ChaincodeProposalPayload + Action(ChaincodeEndorsedAction)

Action = Payload(ProposalResponsePayload) + Endorsement
```

> Envelope.Payload.Header == SignedProposal.Proposal.Header; (和签名提案消息的头部信息相同).
>
> Envelope.Payload.Data.TransactionAction.Header 是签名交易消息的`提交者的身份信息`, 与 Envelope.Payload.Header.SignatureHeader、 SignedProposal.Proposal.Header.SignatureHeader 是相同的.
>
> Envelope.Payload.Data.TransactionAction.Payload.ChaincodeProposalPayload 与 SignedProposal.Proposal.Payload.ChaincodeProposalPayload 类似, 但是前者的TransientMap设置成了nil, 目的是避免在区块中出现敏感信息.
>
> Envelope.Payload.Data.TransactionAction.Payload.Action.Payload 和提案响应消息中的ProposalResponse.Payload相同
>
> Envelope.Payload.Data.Transaction.Payload.Action.Endorsement 是数组, 将多个背书节点的背书签名放置在这里.
>
> 所以也就能理解为什么是基于`签名提案、提案响应(背书签名)`来构建`Envelope签名交易消息`了，因为其中有很多都是相同的

### 第二阶段

##### Orderer节点收到交易进行打包排序，产生区块

> Orderer不读取交易的内容, 所以如果在交易(背书前称为提案, 背书后将提案打包在一起, 称为交易)里面伪造了交易模拟执行的结果, Orderer并不会发现.，或者orderer节点自己作恶一些交易，最终都会在Committer节点进行交易验证阶段会被校验出来, 并标记为无效交易.

Orderer的职责：

1. 接收所有channel发出的交易信息
2. 读取交易数据结构中的Envelope.Payload.Header.ChannelHeader.ChannelId 获取channel名称
3. 按各个channel上交易的消息进行排序, 对一段时间内接受到的交易消息按照打包交易的出块规则(出块的时间、出块的大小等)，生成区块



##### Orderer节点广播给Leader节点

Leader主节点会通过Orderer节点的deliver()服务接口代表组织请求Orderer节点产生的区块，并通过Gossip协议将区块分发给同组织内的peer节点

---

### 第三阶段

##### Committer节点验证区块交易并更新账本

验证流程：(对区块的验证是以交易为单位)

![交易验证](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/image-20201011110002997.png)

- 验证交易

  > 格式是否正确，通过签名的合法性来检查交易是否被篡改

  注意：chaincode的交易是隔离的, 每个交易的模拟执行结果读写集TxRwSet都包含了交易所属的chaincode.

  为避免错误地更新chaincode交易数据, 在交易提交给系统chaincode(VSCC)验证前, 还要对chaincode进行校验. 以下交易都是无效的:

  1. chaincode的名称或版本为空
  2. Envelope.Payload.Header.ChannelHeader.Extension.ChaincodeId.Name != Envelope.Payload.Data.TransactionAction.Payload.ChaincodeProposalPayload.Input.ChaincodeSpec.ChaincodeId.Name
  3. chaincode更新chaincode数据时, 生成读写集的chaincode版本不是LSCS(生命周期管理系统chaincode)记录的最新版本
  4. chaincode更新了LSCC的数据
  5. chaincode更新了VSCC的数据

- 验证背书策略(调用VSCC看是否符合背书策略)

  > 要有**有效的、足够的、特定组织角色**的背书

- MVCC多版本并发控制检查(读写集)，标记交易的有效性

  > 交易通过VSCC检查后, 就进入记账流程. 键值账本会对读写集TxRwSet进行MVCC(Multi-Version Concurrency Control)检查
  >
  > 即检查当前节点状态数据库中的读集数据的版本是否和模拟执行的读集数据的版本是否一致，即检查是否存在读写冲突，并将存在冲突的交易标记为无效交易

  键值账本实现的是基于键值对的状态数据模型, 对状态数据的键有3种操作:

  - 读状态数据
  - 写状态数据
  - 删除状态数据

  对状态数据的读操作有两种形式:

  - 基于单一键的读取
  - 基于键范围的读取

  MVCC的检查是对模拟执行时状态数据的版本和提交交易时状态数据的版本进行比较. 如果版本发生变化, 就说明这段时间之内有别的交易改变了状态数据, 当前交易基于原有状态的处理就是有问题的.

  由于交易提交是并行的, 所以在交易未打包生成区块之前, 并不能确定最终的执行顺序. 如果交易执行的顺序存在依赖, 在MVCC检查的时候就会出现依赖的状态发生了变化, 实际上是数据发生了冲突.

  ![交易有效性验证](https://cdn.jsdelivr.net/gh/VegeBun-csj/Images/image-20201011112019227.png)

  

- 提交所有的区块数据（公共数据）到**区块数据文件**中，保存所有的私有数据(隐私数据)到**隐私数据库(leveldb)**中，建立区块索引信息到**区块索引数据库**，同步**有效交易数据**(公共数据，隐私数据读写集，隐私数据读写集的hash值)到**状态数据库**，提交经过Endorser背书签名的**有效交易数据到历史数据库**。

> 除了伪造的交易会导致交易无效外, 正常的交易也可能出现交易无效.
>
> 因为MVCC检查的是背书节点在模拟执行的时候, 环境是否和committer节点提交交易时的环境(key, value, version)一致. 如果正常提交的交易在这个过程中涉及的数据发生了变化, 也会出现检查失败导致交易无效. 这时, 需要调整交易打包的配置, 重新提交失败的交易等
>
> 无效的交易也会放在区块中，只不过会被标记为无效，并且不会更新到状态数据库和历史数据库中

这样一个完整的交易流程就结束了。