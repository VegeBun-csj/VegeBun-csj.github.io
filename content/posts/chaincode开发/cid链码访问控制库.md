---
title: "Cid链码访问控制库"
date: 2020-08-29T13:27:15+08:00
tags:
- 区块链
- golang
Categories:
- Hyperledger Fabric--chincode开发
---

### Client Identity Chaincode Library（客户端标识链代码库，简称cid）

The client identity chaincode library enables you to write chaincode which
makes access control decisions based on the identity of the client
(i.e. the invoker of the chaincode).  In particular, you may make access
control decisions based on either or both of the following associated with
the client:

 客户端标识链码库使您能够编写基于客户端标识(即链码的调用者)进行访问控制决策的链码。
特别是，您可以根据与客户端关联的下列任一项或两项进行访问控制决策

* the client identity's MSP (Membership Service Provider) ID（客户端身份的MSP(会员服务提供者)ID）
* an attribute associated with the client identity（与客户端身份关联的属性）

Attributes are simply name and value pairs associated with an identity.
For example, `email=me@gmail.com` indicates an identity has the `email`
attribute with a value of `me@gmail.com`.

属性只是与身份关联的键值对。例如，email=me@gmail.com表示一个有着email属性，值为me@gmail.com的身份


### 使用方法

导入相关的包

```
import "github.com/hyperledger/fabric/core/chaincode/lib/cid"
```

> #### 1.Getting the client's ID（获取客户端id）
>

获取客户端的ID，这个ID必须是唯一的:

```
id, err := cid.GetID(stub)
```

> #### 2.Getting the MSP ID（获取mspid）
>

获取客户端身份的MSP ID:

```
mspid, err := cid.GetMSPID(stub)
```

> #### 3.Getting an attribute value（获取属性值）
>

获取attr1属性的值:

```
val, ok, err := cid.GetAttributeValue(stub, "attr1")
if err != nil {
   // There was an error trying to retrieve the attribute
}
if !ok {
   // The client identity does not possess the attribute
}
// Do something with the value of 'val'
```

> #### 4.Asserting an attribute value 断言属性值（判断Attr的key是否为对应的value）
>

通常，您想做的就是根据属性的值做出访问控制决策，即断言属性的值。例如，如果客户端myapp.admin的属性值不为true，下面的代码将返回一个错误。

```
err := cid.AssertAttributeValue(stub, "myapp.admin", "true")
if err != nil {
   // Return an error
}
```

这是有效地使用属性来实现基于角色的访问控制，简称RBAC。

> #### 5.获取客户端的X509证书
>

如何获得客户端的X509证书，如果客户端的身份不是基于X509证书，则为nil:

```
cert, err := cid.GetX509Certificate(stub)
```

请注意，cert和err都可能为nil，如果客户端身份没有使用X509证书，则将出现这种情况。

> #### 6.更有效地执行多个操作
>

有时可能需要执行多个操作才能做出访问决策。例如，下面就是如何使用MSP org1MSP和attr1或MSP org1MSP和attr2授予身份的访问权。

```
// Get the client ID object
id, err := cid.New(stub)
if err != nil {
   // Handle error
}
mspid, err := id.GetMSPID()
if err != nil {
   // Handle error
}
switch mspid {
   case "org1MSP":
      err = id.AssertAttributeValue("attr1", "true")
   case "org2MSP":
      err = id.AssertAttributeValue("attr2", "true")
   default:
      err = errors.New("Wrong MSP")
}
```

Although it is not required, it is more efficient to make the `cid.New` call
to get the ClientIdentity object if you need to perform multiple operations,
as demonstrated above.

虽然不是必需的。如果需要执行多个操作，用`cid.New`调用来获取ClientIdentity对象是非常有效的，如上所示。

### Adding Attributes to Identities（向身份添加属性）

如何在使用Hyperledger Fabric CA时以及在使用外部CA时向证书添加自定义属性。

> #### 1.Managing attributes with Fabric CA（使用Fabric CA管理属性）

使用fabric-ca向注册证书添加属性有两种方法:

  1. 当注册一个身份时，您可以指定为该身份颁发的登记证书（e-cert）在缺省情况下应该包含一个属性(attribute)。可以在注册时覆盖，但这对于建立默认行为非常有用，并且假设注册发生在应用程序之外，不需要对应用程序进行任何更改。

     下面就是如何使用两个属性 app1Admin和email注册user1。“:ecert”后缀在默认情况下会将appAdmin属性插入user1的登记证书（e-cert）中。默认情况下，是不会将email属性添加到登记证书(e-cert)中

     > 这种方式其实是和在NodeSDK进行注册user时的操作是一样的，在创建新用户的时候，可以指定相应的attributes，而这个属性就可以作为后序在链码中进行getAttr的一个值，通过这个值是否符合预期来进行链码的访问控制

     ```
     fabric-ca-client register --id.name user1 --id.secret user1pw --id.type user --id.affiliation org1 --id.attrs 'app1Admin=true:ecert,email=user1@gmail.com'
     ```

  2. 当您登记一个身份时，您可以请求将一个或多个属性添加到证书中。对于请求的每个属性，您可以指定该属性是否是可选的。如果该身份的属性是必选的，但不存在，则登记失败。

     下面展示了如何使用email属性注册user1，不使用app1Admin属性，也可以使用phone属性(如果用户拥有phone属性)。

     ```
     fabric-ca-client enroll -u http://user1:user1pw@localhost:7054 --enrollment.attrs "email,phone:opt"
     ```

> #### 2.Attribute format in a certificate （证书中的属性格式）
>

属性作为扩展名存储在X509证书中，其ASN.1 OID(抽象语法符号对象标识符)为1.2.3.4.5.6.7.8.1。该扩展的值是以{"attrs":{<attrName>:<attrValue}}（属性名，属性值这种键值对）为形式的JSON字符串。下面是一个证书示例，其中包含值为val1的attr1属性。还要注意，JSON条目可以包含多个属性，但本示例只显示了一个属性。

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            1e:49:98:e9:f4:4f:d0:03:53:bf:36:81:c0:a0:a4:31:96:4f:52:75
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN=fabric-ca-server
        Validity
            Not Before: Sep  8 03:42:00 2017 GMT
            Not After : Sep  8 03:42:00 2018 GMT
        Subject: CN=MyTestUserWithAttrs
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
            EC Public Key:
                pub:
                    04:e6:07:5a:f7:09:d5:af:38:e3:f7:a2:90:77:0e:
                    32:67:5b:70:a7:37:ca:b5:c9:d8:91:77:39:ae:03:
                    a0:36:ad:72:b3:3c:89:6d:1e:f6:1b:6d:2a:88:49:
                    92:6e:6e:cc:bc:81:52:fa:19:88:18:5c:d7:6e:eb:
                    d4:73:cc:51:79
                ASN1 OID: prime256v1
        X509v3 extensions:
            X509v3 Key Usage: critical
                Certificate Sign
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier:
                D8:28:B4:C0:BC:92:4A:D3:C3:8C:54:6C:08:86:33:10:A6:8D:83:AE
            X509v3 Authority Key Identifier:
                keyid:C4:B3:FE:76:0D:E2:DE:3C:FC:75:FB:AE:55:86:04:F0:BB:7F:F6:01

            X509v3 Subject Alternative Name:
                DNS:Anils-MacBook-Pro.local
            1.2.3.4.5.6.7.8.1:
                {"attrs":{"attr1":"val1"}}
    Signature Algorithm: ecdsa-with-SHA256
        30:45:02:21:00:fb:84:a9:65:29:b2:f4:d3:bc:1a:8b:47:92:
        5e:41:27:2d:26:ec:f7:cd:aa:86:46:a4:ac:da:25:be:40:1d:
        c5:02:20:08:3f:49:86:58:a7:20:48:64:4c:30:1b:da:a9:a2:
        f2:b4:16:28:f6:fd:e1:46:dd:6b:f2:3f:2f:37:4a:4c:72
```

如果想使用前面描述的客户端身份库来提取或断言属性值，但是没有使用Hyperledger Fabric CA，那么必须确保由自己的外部CA颁发的证书包含上面所示表单的属性。特别是，证书必须包含1.2.3.4.5.6.7.8.1 X509v3扩展，其JSON值包含标识的属性名和值。

---

---

---

说了这么多，来句一个栗子：**(注意在合约中引入相关的依赖，即cid库文件)**

此链码库cid是用来进行链码操作权限的控制的，那么，这个attrs的name-value该如何获取呢，从上面的解释中可以看到跟CA有关，所以是在我们注册用户身份的时候进行设置的，下面列举使用Node sdk进行初始化设置attrs的例子

```javascript
 const secret = await ca.register(
     {
       affiliation: 'org1.department1',
       enrollmentID: 'cai',
       role: 'client',
       attrs: [					//在register的传入属性中有attrs这个参数，这是一个Array数组
       {						//其中传入三个
         name: 'usertype',		//attrs的name，也就是key
         value: 'storeshop',	//attrs的value，也就是key对应的值
         ecert: true			//ecert为true表示默认情况下应将此属性包括在注册证书中(默认)
      }]
     },
    adminIdentity
   );
```

上面就是设置了attrs的name是usertype,value是storeshop

在链码中我们这样写

```go
//根据name获取用户的attr的值
//这里就是根据name--usertype来获取我们注册时设置的value，如果使用上面注册时的身份user进行调用就会获取到usertype对应的value是storeshop
func (s *SmartContract)getAttr(APIstub shim.ChaincodeStubInterface) sc.Response {
  res, found, err := cid.GetAttributeValue(APIstub, "usertype")
  if found==true {
  	 shim.Error("未查询到属性值")
  }else if err != nil{
  	 shim.Error("查询错误")
  }
  return shim.Success([]byte(res))
}

//类型断言 -----用来判断是否当前的操作用户的attr的key与value是否与这里指定的值匹配
func (s *SmartContract)AssertAttr(APIstub shim.ChaincodeStubInterface) sc.Response {
  err := cid.AssertAttributeValue(APIstub, "usertype", "storeshop")
  if err != nil {
    return shim.Error("类型不是storeshop")
  }
  return shim.Success([]byte("类型是storeshop"))
}
```