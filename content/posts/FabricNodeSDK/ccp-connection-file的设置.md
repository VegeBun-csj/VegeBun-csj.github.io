---
title: "ccp文件的设置"
date: 2020-08-29T13:03:02+08:00
tags:
- 区块链
Categories:
- Hyperledger Fabric--NodeSDK开发
---

> 1.实验环境时，我们使用cli容器来进行链码的部署，实例化，以及进行简单的交易操作，但是在Fabric进行开发时，为了避免命令行的繁琐行，采用fabric官方的SDK来进行开发， 尤其是进行链码的调用(我在自己的使用中都是使用cli进行链码的部署，实例化的，然后在cli中测试一下基本的链码功能是否正常，最后在SDK这头进行调用交易即可，不太喜欢SDK进行链码的部署这些，因为感觉是运维的做的事情哈哈哈，放在shell中一套命令安装完不香嘛，哈哈哈:smile:)
>
> 2.在使用SDK之前需要一个配置文件，设置一些节点的证书，以及访问点，因为SDK毕竟最后是充当一个客户端来和网络进行交互的，就涉及到通过哪个peer节点与网络进行交互的问题，CCP文件就是做的这个事情(类似于一个user的登入点)

在使用node-sdk的high-level(fabric1.4.x版本开始升级的新的API，它的写法更加简洁，相比于1.4.x之前的版本的NodeSDK，进行了更进一步的封装，使得链码的调用更加方便)的API时，碰到了sdk中query时好时坏的问题

> #### 1.问题描述：sdk中使用的CCP文件中定义了所有的peer，导致sdk端在扫描该文件时，出现一会链接peer0org1一会链接peer1org1的问题

```json
{
    "name": "first-network-org1",
    "version": "1.0.0",
    "client": {
        "organization": "Org1",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300"
                }
            }
        }
    },
    "organizations": {
        "Org1": {
            "mspid": "Org1MSP",
            "peers": [
                "peer0.org1.example.com",
                "peer1.org1.example.com"
            ],
            "certificateAuthorities": [
                "ca.org1.example.com"
            ]
        }
    },
    "peers": {
        "peer0.org1.example.com": {
            "url": "grpcs://localhost:7051",
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICVzCCAf6gAwIBAgIRAOeRcny12K3xMMPY0BrYJEIwCgYIKoZIzj0EAwIwdjEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHzAdBgNVBAMTFnRs\nc2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMjAwMzE1MDkxNDAwWhcNMzAwMzEzMDkx\nNDAwWjB2MQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UE\nBxMNU2FuIEZyYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEfMB0G\nA1UEAxMWdGxzY2Eub3JnMS5leGFtcGxlLmNvbTBZMBMGByqGSM49AgEGCCqGSM49\nAwEHA0IABMDyCkDy7DP80rm7L0w+OLFyCoN2irvBOU4jww2XLBpsU4+0o+nAgJqC\nCOL/rt/GHwmmX3ka2ioTWwtumtei4/ejbTBrMA4GA1UdDwEB/wQEAwIBpjAdBgNV\nHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEwDwYDVR0TAQH/BAUwAwEB/zApBgNV\nHQ4EIgQgRkvXAREOeeZzEzyAnb99rqWIzAW2RqFVriuhZoZSBpUwCgYIKoZIzj0E\nAwIDRwAwRAIgCcY4vir60ukkLNNHcQ9G22q1i2EsnFdOxH7VfdRASq8CIG2BL/7k\n98ewzous+ZiTv10pAQDdV4yvHY1w4OrTfDgP\n-----END CERTIFICATE-----\n"
            },
            "grpcOptions": {
                "ssl-target-name-override": "peer0.org1.example.com",
                "hostnameOverride": "peer0.org1.example.com"
            }
        },
        "peer1.org1.example.com": {
            "url": "grpcs://localhost:8051",
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICVzCCAf6gAwIBAgIRAOeRcny12K3xMMPY0BrYJEIwCgYIKoZIzj0EAwIwdjEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHzAdBgNVBAMTFnRs\nc2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMjAwMzE1MDkxNDAwWhcNMzAwMzEzMDkx\nNDAwWjB2MQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UE\nBxMNU2FuIEZyYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEfMB0G\nA1UEAxMWdGxzY2Eub3JnMS5leGFtcGxlLmNvbTBZMBMGByqGSM49AgEGCCqGSM49\nAwEHA0IABMDyCkDy7DP80rm7L0w+OLFyCoN2irvBOU4jww2XLBpsU4+0o+nAgJqC\nCOL/rt/GHwmmX3ka2ioTWwtumtei4/ejbTBrMA4GA1UdDwEB/wQEAwIBpjAdBgNV\nHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEwDwYDVR0TAQH/BAUwAwEB/zApBgNV\nHQ4EIgQgRkvXAREOeeZzEzyAnb99rqWIzAW2RqFVriuhZoZSBpUwCgYIKoZIzj0E\nAwIDRwAwRAIgCcY4vir60ukkLNNHcQ9G22q1i2EsnFdOxH7VfdRASq8CIG2BL/7k\n98ewzous+ZiTv10pAQDdV4yvHY1w4OrTfDgP\n-----END CERTIFICATE-----\n"
            },
            "grpcOptions": {
                "ssl-target-name-override": "peer1.org1.example.com",
                "hostnameOverride": "peer1.org1.example.com"
            }
        }
    },
    "certificateAuthorities": {
        "ca.org1.example.com": {
            "url": "https://127.0.0.1:7054",
            "caName": "ca-org1",
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICUTCCAfegAwIBAgIQQCigCctKgodE1Q7gcR5LXDAKBggqhkjOPQQDAjBzMQsw\nCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy\nYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEcMBoGA1UEAxMTY2Eu\nb3JnMS5leGFtcGxlLmNvbTAeFw0yMDAzMTUwOTE0MDBaFw0zMDAzMTMwOTE0MDBa\nMHMxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T\nYW4gRnJhbmNpc2NvMRkwFwYDVQQKExBvcmcxLmV4YW1wbGUuY29tMRwwGgYDVQQD\nExNjYS5vcmcxLmV4YW1wbGUuY29tMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nkhODBxDZsbsm+4roHRwjPhOavnvmGRNnktsKB1X9IncaDDu4jSduwjOu9zdZm/B4\nixe1pE/NhpjMhpXmYQBDk6NtMGswDgYDVR0PAQH/BAQDAgGmMB0GA1UdJQQWMBQG\nCCsGAQUFBwMCBggrBgEFBQcDATAPBgNVHRMBAf8EBTADAQH/MCkGA1UdDgQiBCAS\nCZsC0MsmOf9PukUYg+mCYx0qZC3bIBiB4O2Y+ORVTzAKBggqhkjOPQQDAgNIADBF\nAiEAigmuFPeCY59LcV0KNU27y/v6Tgl3p8NlCBQv7vgXIigCID5vB6P+v9eQnQPj\nyn4alwVpNaO1qnAXZH+nekhnoTEs\n-----END CERTIFICATE-----\n"
            },
            "httpOptions": {
                "verify": false
            }
        }
    }
}

```

peer0org1安装了chaincode,但是peer1org1没有安装chaincode，所以当client端将peer1org1作为endpoint时就会报下面找不到链码的错误，原因就是没有指定好client端连接哪个peer

```bash
2020-03-15T10:22:30.253Z - warn: [Query]: evaluate: Query ID "[object Object]" of peer "peer1.org1.example.com:8051" failed: message=cannot retrieve package for chaincode emrscc/1.0, error open /var/hyperledger/production/chaincodes/emrscc.1.0: no such file or directory, stack=Error: cannot retrieve package for chaincode emrscc/1.0, error open /var/hyperledger/production/chaincodes/emrscc.1.0: no such file or directory
    at self._endorserClient.processProposal (/home/songjian/go/src/github.com/hyperledger/fabric-samples/fabcar/javascript/node_modules/_fabric-client@1.4.7@fabric-client/lib/Peer.js:144:36)
    at Object.onReceiveStatus (/home/songjian/go/src/github.com/hyperledger/fabric-samples/fabcar/javascript/node_modules/_grpc@1.23.3@grpc/src/client_interceptors.js:1207:9)
    at InterceptingListener._callNext (/home/songjian/go/src/github.com/hyperledger/fabric-samples/fabcar/javascript/node_modules/_grpc@1.23.3@grpc/src/client_interceptors.js:568:42)
    at InterceptingListener.onReceiveStatus (/home/songjian/go/src/github.com/hyperledger/fabric-samples/fabcar/javascript/node_modules/_grpc@1.23.3@grpc/src/client_interceptors.js:618:8)
    at callback (/home/songjian/go/src/github.com/hyperledger/fabric-samples/fabcar/javascript/node_modules/_grpc@1.23.3@grpc/src/client_interceptors.js:845:24), status=500, , url=grpcs://localhost:8051, name=peer1.org1.example.com:8051, grpc.max_receive_message_length=-1, grpc.max_send_message_length=-1, grpc.keepalive_time_ms=120000, grpc.http2.min_time_between_pings_ms=120000, grpc.keepalive_timeout_ms=20000, grpc.http2.max_pings_without_data=0, grpc.keepalive_permit_without_calls=1, name=peer1.org1.example.com:8051, grpc.ssl_target_name_override=peer1.org1.example.com, grpc.default_authority=peer1.org1.example.com, isProposalResponse=true
Failed to evaluate transaction: Error: cannot retrieve package for chaincode emrscc/1.0, error open /var/hyperledger/production/chaincodes/emrscc.1.0: no such file or directory
```

所以需要修改CCP文件（一开始是将原来的包含所有peer的CCP文件剔除掉一些peer，变成下面这样，但是还是报上面的错）原因就是没有关掉gateway.connect()时的discoveryOption参数中的enable,应该设置为false,不让其扫描本地。由于discovery service的原因，导致连接peer不确定，所以将discovery service关掉指定某个peer为访问点即可

```json
{
    "name": "first-network-org1",
    "version": "1.0.0",
    "client": {
        "organization": "Org1",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300"
                }
            }
        }
    },
    "organizations": {
        "Org1": {
            "mspid": "Org1MSP",
            "peers": [
                "peer0.org1.example.com"
            ],
            "certificateAuthorities": [
                "ca.org1.example.com"
            ]
        }
    },
    "peers": {
        "peer0.org1.example.com": {
            "url": "grpcs://localhost:7051",
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICVzCCAf6gAwIBAgIRAOeRcny12K3xMMPY0BrYJEIwCgYIKoZIzj0EAwIwdjEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHzAdBgNVBAMTFnRs\nc2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMjAwMzE1MDkxNDAwWhcNMzAwMzEzMDkx\nNDAwWjB2MQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UE\nBxMNU2FuIEZyYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEfMB0G\nA1UEAxMWdGxzY2Eub3JnMS5leGFtcGxlLmNvbTBZMBMGByqGSM49AgEGCCqGSM49\nAwEHA0IABMDyCkDy7DP80rm7L0w+OLFyCoN2irvBOU4jww2XLBpsU4+0o+nAgJqC\nCOL/rt/GHwmmX3ka2ioTWwtumtei4/ejbTBrMA4GA1UdDwEB/wQEAwIBpjAdBgNV\nHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEwDwYDVR0TAQH/BAUwAwEB/zApBgNV\nHQ4EIgQgRkvXAREOeeZzEzyAnb99rqWIzAW2RqFVriuhZoZSBpUwCgYIKoZIzj0E\nAwIDRwAwRAIgCcY4vir60ukkLNNHcQ9G22q1i2EsnFdOxH7VfdRASq8CIG2BL/7k\n98ewzous+ZiTv10pAQDdV4yvHY1w4OrTfDgP\n-----END CERTIFICATE-----\n"
            },
            "grpcOptions": {
                "ssl-target-name-override": "peer0.org1.example.com",
                "hostnameOverride": "peer0.org1.example.com"
            }
        }
    },
    "certificateAuthorities": {
        "ca.org1.example.com": {
            "url": "https://127.0.0.1:7054",
            "caName": "ca-org1",
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICUTCCAfegAwIBAgIQQCigCctKgodE1Q7gcR5LXDAKBggqhkjOPQQDAjBzMQsw\nCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy\nYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEcMBoGA1UEAxMTY2Eu\nb3JnMS5leGFtcGxlLmNvbTAeFw0yMDAzMTUwOTE0MDBaFw0zMDAzMTMwOTE0MDBa\nMHMxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T\nYW4gRnJhbmNpc2NvMRkwFwYDVQQKExBvcmcxLmV4YW1wbGUuY29tMRwwGgYDVQQD\nExNjYS5vcmcxLmV4YW1wbGUuY29tMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nkhODBxDZsbsm+4roHRwjPhOavnvmGRNnktsKB1X9IncaDDu4jSduwjOu9zdZm/B4\nixe1pE/NhpjMhpXmYQBDk6NtMGswDgYDVR0PAQH/BAQDAgGmMB0GA1UdJQQWMBQG\nCCsGAQUFBwMCBggrBgEFBQcDATAPBgNVHRMBAf8EBTADAQH/MCkGA1UdDgQiBCAS\nCZsC0MsmOf9PukUYg+mCYx0qZC3bIBiB4O2Y+ORVTzAKBggqhkjOPQQDAgNIADBF\nAiEAigmuFPeCY59LcV0KNU27y/v6Tgl3p8NlCBQv7vgXIigCID5vB6P+v9eQnQPj\nyn4alwVpNaO1qnAXZH+nekhnoTEs\n-----END CERTIFICATE-----\n"
            },
            "httpOptions": {
                "verify": false
            }
        }
    }
}

```

但是又出现了新的错误，如下：

```bash
2020-03-15T10:52:10.058Z - error: [Network]: _initializeInternalChannel: No peers defined in channel that have the ledger query role
Failed to evaluate transaction: Error: No peers defined in channel that have the ledger query role
```

这是因为CCP中没有设置channels的配置,并将discovery的enable置为false,这才大功告成！！

**我在实验过程中做过测试，只有query的操作采用false,但对于invoke操作是需要true的，这一点需要注意**

```json
{
    "name": "first-network-org1",
    "version": "1.0.0",
    "client": {
        "organization": "Org1",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300"
                }
            }
        }
    },
    "channels": {
        "mychannel": {
            "orderers": [
                "orderer.example.com"
            ],
            "peers": {
                "peer0.org1.example.com": {}
            }
        }
    },
    "organizations": {
        "Org1": {
            "mspid": "Org1MSP",
            "peers": [
                "peer0.org1.example.com"
            ],
            "certificateAuthorities": [
                "ca.org1.example.com"
            ]
        }
    },
    "peers": {
        "peer0.org1.example.com": {
            "url": "grpcs://localhost:7051",
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICVzCCAf6gAwIBAgIRAOeRcny12K3xMMPY0BrYJEIwCgYIKoZIzj0EAwIwdjEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHzAdBgNVBAMTFnRs\nc2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMjAwMzE1MDkxNDAwWhcNMzAwMzEzMDkx\nNDAwWjB2MQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UE\nBxMNU2FuIEZyYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEfMB0G\nA1UEAxMWdGxzY2Eub3JnMS5leGFtcGxlLmNvbTBZMBMGByqGSM49AgEGCCqGSM49\nAwEHA0IABMDyCkDy7DP80rm7L0w+OLFyCoN2irvBOU4jww2XLBpsU4+0o+nAgJqC\nCOL/rt/GHwmmX3ka2ioTWwtumtei4/ejbTBrMA4GA1UdDwEB/wQEAwIBpjAdBgNV\nHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEwDwYDVR0TAQH/BAUwAwEB/zApBgNV\nHQ4EIgQgRkvXAREOeeZzEzyAnb99rqWIzAW2RqFVriuhZoZSBpUwCgYIKoZIzj0E\nAwIDRwAwRAIgCcY4vir60ukkLNNHcQ9G22q1i2EsnFdOxH7VfdRASq8CIG2BL/7k\n98ewzous+ZiTv10pAQDdV4yvHY1w4OrTfDgP\n-----END CERTIFICATE-----\n"
            },
            "grpcOptions": {
                "ssl-target-name-override": "peer0.org1.example.com",
                "hostnameOverride": "peer0.org1.example.com"
            }
        }
    },
    "certificateAuthorities": {
        "ca.org1.example.com": {
            "url": "https://127.0.0.1:7054",
            "caName": "ca-org1",
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----\nMIICUTCCAfegAwIBAgIQQCigCctKgodE1Q7gcR5LXDAKBggqhkjOPQQDAjBzMQsw\nCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy\nYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEcMBoGA1UEAxMTY2Eu\nb3JnMS5leGFtcGxlLmNvbTAeFw0yMDAzMTUwOTE0MDBaFw0zMDAzMTMwOTE0MDBa\nMHMxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T\nYW4gRnJhbmNpc2NvMRkwFwYDVQQKExBvcmcxLmV4YW1wbGUuY29tMRwwGgYDVQQD\nExNjYS5vcmcxLmV4YW1wbGUuY29tMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nkhODBxDZsbsm+4roHRwjPhOavnvmGRNnktsKB1X9IncaDDu4jSduwjOu9zdZm/B4\nixe1pE/NhpjMhpXmYQBDk6NtMGswDgYDVR0PAQH/BAQDAgGmMB0GA1UdJQQWMBQG\nCCsGAQUFBwMCBggrBgEFBQcDATAPBgNVHRMBAf8EBTADAQH/MCkGA1UdDgQiBCAS\nCZsC0MsmOf9PukUYg+mCYx0qZC3bIBiB4O2Y+ORVTzAKBggqhkjOPQQDAgNIADBF\nAiEAigmuFPeCY59LcV0KNU27y/v6Tgl3p8NlCBQv7vgXIigCID5vB6P+v9eQnQPj\nyn4alwVpNaO1qnAXZH+nekhnoTEs\n-----END CERTIFICATE-----\n"
            },
            "httpOptions": {
                "verify": false
            }
        }
    }
}

```



> #### 2.问题描述：当我需要在org2的节点peer0上进行操作时，由于我没有注册org2的admin和对应的user，然后又在CCP文件中错误的赋值的org1peer1的配置,所才出现了以下的错误

```bash
2020-03-15T13:42:06.069Z - error: [Network]: _initializeInternalChannel: No peers defined for MSP 'Org1MSP' to discover from
Failed to submit transaction: Error: No peers defined for MSP 'Org1MSP' to discover from
```

**注意：在切换了相应的endpoint（组织节点）时，一定要检查当前使用的client的user是哪个组织的用户，这对于和相应的peer节点的交互尤为重要**





-------------------------------我是更新部分    -------------------------------2020年3月31日

