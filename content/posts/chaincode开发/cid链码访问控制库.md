---
title: "链码的访问控制(cid库)"
date: 2020-08-29T13:27:15+08:00
tags:
- 区块链
- golang
Categories:
- Hyperledger Fabric--chincode开发
---

# Client Identity Chaincode Library（客户端标识链代码库）

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


## Using the client identity chaincode library（使用方法）

This section describes how to use the client identity chaincode library.

All code samples below assume two things:

1. The type of the `stub` variable is `ChaincodeStubInterface` as passed
   to your chaincode. 将stub作为`ChaincodeStubInterface`接口传入chaincode

2. You have added the following import statement to your chaincode.  导入相关的包

   ```
   import "github.com/hyperledger/fabric/core/chaincode/lib/cid"
   ```

#### Getting the client's ID（获取客户端id）

The following demonstrates how to get an ID for the client which is guaranteed
to be unique within the MSP:  下面演示了如何为客户端获取一个ID，它保证在MSP中是唯一的:

```
id, err := cid.GetID(stub)
```

#### Getting the MSP ID（获取mspid）

The following demonstrates how to get the MSP ID of the client's identity: 下面演示如何获取客户端身份的MSP ID:

```
mspid, err := cid.GetMSPID(stub)
```

#### Getting an attribute value（获取属性值）

The following demonstrates how to get the value of the *attr1* attribute:下面演示了如何获取attr1属性的值:

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

#### Asserting an attribute value 断言属性值（判断Attr的key是否为对应的value）

Often all you want to do is to make an access control decision based on the value
of an attribute, i.e. to assert the value of an attribute.  For example, the following
will return an error if the client does not have the `myapp.admin` attribute
with a value of `true`:p

通常，您想做的就是根据属性的值做出访问控制决策，即断言属性的值。例如，如果客户端myapp.admin的属性值不为true，下面的代码将返回一个错误。

```
err := cid.AssertAttributeValue(stub, "myapp.admin", "true")
if err != nil {
   // Return an error
}
```

This is effectively using attributes to implement role-based access control,
or RBAC for short.

这是有效地使用属性来实现基于角色的访问控制，简称RBAC。

#### Getting the client's X509 certificate            获取客户端的X509证书

The following demonstrates how to get the X509 certificate of the client, or
nil if the client's identity was not based on an X509 certificate:

下面演示了如何获得客户端的X509证书，如果客户端的身份不是基于X509证书，则为nil:

```
cert, err := cid.GetX509Certificate(stub)
```

Note that both `cert` and `err` may be nil as will be the case if the identity
is not using an X509 certificate.

请注意，cert和err都可能为nil，如果客户端身份没有使用X509证书，则将出现这种情况。

#### Performing multiple operations more efficiently（更有效地执行多个操作）

Sometimes you may need to perform multiple operations in order to make an access
decision.  For example, the following demonstrates how to grant access to
identities with MSP *org1MSP* and *attr1* OR with MSP *org1MSP* and *attr2*.

有时您可能需要执行多个操作才能做出访问决策。例如，下面演示了如何使用MSP org1MSP和attr1或MSP org1MSP和attr2授予身份的访问权。

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

## Adding Attributes to Identities（向身份添加属性）

This section describes how to add custom attributes to certificates when
using Hyperledger Fabric CA as well as when using an external CA.

本节描述如何在使用Hyperledger Fabric CA时以及在使用外部CA时向证书添加自定义属性。

#### Managing attributes with Fabric CA（使用Fabric CA管理属性）

There are two methods of adding attributes to an enrollment certificate
with fabric-ca:

（使用fabric-ca向注册证书添加属性有两种方法:）

  1. When you register an identity, you can specify that an enrollment certificate
     issued for the identity should by default contain an attribute.  This behavior
     can be overridden at enrollment time, but this is useful for establishing
     default behavior and, assuming registration occurs outside of your application,
     does not require any application change.

     当您注册一个身份时，您可以指定为该身份颁发的登记证书（e-cert）在缺省情况下应该包含一个属性(attribute)。可以在注册时覆盖此行为，但这对于建立默认行为非常有用，并且假设注册发生在应用程序之外，不需要对应用程序进行任何更改。

     The following shows how to register *user1* with two attributes:
     *app1Admin* and *email*.
     The ":ecert" suffix causes the *appAdmin* attribute to be inserted into user1's
     enrollment certificate by default.  The *email* attribute is not added
     to the enrollment certificate by default.

     下面展示了如何使用两个属性 app1Admin和email注册user1。“:ecert”后缀在默认情况下会将appAdmin属性插入user1的登记证书（e-cert）中。默认情况下，未将email属性添加到登记证书(e-cert)中

     ```
     fabric-ca-client register --id.name user1 --id.secret user1pw --id.type user --id.affiliation org1 --id.attrs 'app1Admin=true:ecert,email=user1@gmail.com'
     ```

  2. When you enroll an identity, you may request that one or more attributes
     be added to the certificate.
     For each attribute requested, you may specify whether the attribute is
     optional or not.  If it is not optional but does not exist for the identity,
     enrollment fails.

     当您登记一个身份时，您可以请求将一个或多个属性添加到证书中。对于请求的每个属性，您可以指定该属性是否是可选的。如果该身份的属性是必选的，但不存在，则登记失败。

     The following shows how to enroll *user1* with the *email* attribute,
     without the *app1Admin* attribute and optionally with the *phone* attribute
     (if the user possesses *phone* attribute).

     下面展示了如何使用email属性注册user1，不使用app1Admin属性，也可以使用phone属性(如果用户拥有phone属性)。

     ```
     fabric-ca-client enroll -u http://user1:user1pw@localhost:7054 --enrollment.attrs "email,phone:opt"
     ```

#### Attribute format in a certificate （证书中的属性格式）

Attributes are stored inside an X509 certificate as an extension with an
ASN.1 OID (Abstract Syntax Notation Object IDentifier)
of `1.2.3.4.5.6.7.8.1`.  The value of the extension is a JSON string of the
form `{"attrs":{<attrName>:<attrValue}}`.  The following is a sample of a
certificate which contains the `attr1` attribute with a value of `val1`.
See the final entry in the *X509v3 extensions* section.  Note also that the JSON
entry could contain multiple attributes, though this sample shows only one.

属性作为扩展名存储在X509证书中，其ASN.1 OID(抽象语法符号对象标识符)为1.2.3.4.5.6.7.8.1。该扩展的值是以{"attrs":{<attrName>:<attrValue}}（属性名，属性值这种键值对）为形式的JSON字符串。下面是一个证书示例，其中包含值为val1的attr1属性。请参阅X509v3扩展部分中的最后一项。还要注意，JSON条目可以包含多个属性，但本示例只显示了一个属性。

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

If you want to use the client identity library to extract or assert attribute
values as described previously but you are not using Hyperledger Fabric CA,
then you must ensure that the certificates which are issued by your external CA
contain attributes of the form shown above.  In particular, the certificates
must contain the `1.2.3.4.5.6.7.8.1` X509v3 extension with a JSON value
containing the attribute names and values for the identity.

如果您想使用前面描述的客户端身份库来提取或断言属性值，但是您没有使用Hyperledger Fabric CA，那么您必须确保由您的外部CA颁发的证书包含上面所示表单的属性。特别是，证书必须包含1.2.3.4.5.6.7.8.1 X509v3扩展，其JSON值包含标识的属性名和值。



---

下面列举一个实际使用的例子：**(注意在合约中引入相关的依赖，即cid库文件，这里可以借鉴fabric-samples/chaincode/abac中的库的引用)**

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

链码我们是以官方的fabcar链码进行了小小的修改，地址如下：

> https://github.com/VegeBun-csj/learning-chaincode/blob/master/access-control/go/fabcar.go
>

```go
//根据name获取用户的attr的值
//这里就是根据name--usertype来获取我们注册时设置的value，如果使用上面注册时的身份user进行调用就会获取到usertype对应的value是storeshop
func (s *SmartContract)getAttr(APIstub shim.ChaincodeStubInterface) sc.Response {
  res, found, err := cid.GetAttributeValue(APIstub, "usertype")
  if found == false {
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



演示一个在Java-SDK中通过`CID`库进行链码访问控制的例子：

链码地址如下：

fabric-SDK-Java的客户端代码为:

```java
/*
SPDX-License-Identifier: Apache-2.0
*/

package org.example;

import java.nio.file.Paths;
import java.util.Properties;

import org.example.Common.User;
import org.hyperledger.fabric.gateway.Wallet;
import org.hyperledger.fabric.gateway.Wallet.Identity;
import org.hyperledger.fabric.sdk.Enrollment;
import org.hyperledger.fabric.sdk.security.CryptoSuite;
import org.hyperledger.fabric.sdk.security.CryptoSuiteFactory;
import org.hyperledger.fabric_ca.sdk.Attribute;
import org.hyperledger.fabric_ca.sdk.EnrollmentRequest;
import org.hyperledger.fabric_ca.sdk.HFCAClient;
import org.hyperledger.fabric_ca.sdk.RegistrationRequest;

public class RegisterUser {

    static {
        System.setProperty("org.hyperledger.fabric.sdk.service_discovery.as_localhost", "true");
    }

    public static void main(String[] args) throws Exception {

        // Create a CA client for interacting with the CA.
        Properties props = new Properties();
        props.put("pemFile", EnrollAdmin.CERTIFICATION_PATH);
        props.put("allowAllHostNames", "true");
        HFCAClient caClient = HFCAClient.createNewInstance(EnrollAdmin.PATH_AND_PORT, props);
        CryptoSuite cryptoSuite = CryptoSuiteFactory.getDefault().getCryptoSuite();
        caClient.setCryptoSuite(cryptoSuite);

        // Create a wallet for managing identities
        Wallet wallet = Wallet.createFileSystemWallet(Paths.get("src/main/resources/wallet"));

        // Check to see if we've already enrolled the user.
        boolean userExists = wallet.exists("songjian");
        if (userExists) {
            System.out.println("An identity for the user \"songjian\" already exists in the wallet");
            return;
        }

        userExists = wallet.exists("admin");
        if (!userExists) {
            System.out.println("\"admin\" needs to be enrolled and added to the wallet first");
            return;
        }

        Identity adminIdentity = wallet.get("admin");
        org.hyperledger.fabric.sdk.User admin = new User.Builder()
                .name("admin")
                .affiliation("org1.department1")
                .mspId("Org1MSP")
                .enrollment(adminIdentity)
                .build();

        // Register the user, enroll the user, and import the new identity into the wallet.
        RegistrationRequest registrationRequest = new RegistrationRequest("songjian");
        registrationRequest.setAffiliation("org1.department1");
        registrationRequest.setEnrollmentID("songjian");

        //register的时候设置用户自定义的属性类型
        registrationRequest.addAttribute(new Attribute("usertype","worker"));
        String enrollmentSecret = caClient.register(registrationRequest, admin);
        //enroll的时候，指定之前register的用户中的哪些属性要加入到证书中，如果不加，后买你在链码中就读取不到
        EnrollmentRequest enrollmentRequest = new EnrollmentRequest();
        enrollmentRequest.addAttrReq("hf.Affiliation");		//default attribute
        enrollmentRequest.addAttrReq("hf.EnrollmentID");	//default attribute
        enrollmentRequest.addAttrReq("hf.Type");			//default attribute
        enrollmentRequest.addAttrReq("usertype");			//user-defined attribute

        Enrollment enrollment = caClient.enroll("songjian", enrollmentSecret,enrollmentRequest);
        Identity user = Identity.createIdentity("Org1MSP", enrollment.getCert(), enrollment.getKey());
        wallet.put("songjian", user);
        System.out.println("Successfully enrolled user \"songjian\" and imported it into the wallet");
    }
}
```

> 注意：在register中自定义了属性值之后，需要在最后进行Enroll的时候，将相关的属性放入`EnrollmentRequest`中，否则就会出现在链码中获取不到attribution的属性值，我这边就是Enroll的时候没加，导致我一度怀疑自己的链码出现了问题，最后看到网上大佬的才发现是这步不对，之前使用NodeSDK的时候好像没这个问题，估计是SDK不同pa。。。。也算是学到了hh

这个例子想实现的就是，fabcar中的汽车，只有用户属性为`leader_car_department(汽车部门主管)`类型的才可以查看所有的汽车并且更改汽车的所有者,其他属性值的是看不了的，比如普通的工人`worker`

用户属性`Attr`我们定义为`usertype`，但是`value`的值就可以有很多了，比如worker(员工).....还有其他的。。。

上面是注册了一个`Attr`为`usertype`，`value`为`worker`的用户`songjian`，所以这个用户是不能调用`queryAllCars`和`changeCarOwner`的

下面我们再注册一个用户`yfhuang`，他的类型为`leader_car_department(汽车部门主管)`,所以就可以使用`queryAllCars`和`changeCarOwner`

```java
/*
SPDX-License-Identifier: Apache-2.0
*/

package org.example;

import java.nio.file.Paths;
import java.util.Properties;

import org.example.Common.User;
import org.hyperledger.fabric.gateway.Wallet;
import org.hyperledger.fabric.gateway.Wallet.Identity;
import org.hyperledger.fabric.sdk.Enrollment;
import org.hyperledger.fabric.sdk.security.CryptoSuite;
import org.hyperledger.fabric.sdk.security.CryptoSuiteFactory;
import org.hyperledger.fabric_ca.sdk.Attribute;
import org.hyperledger.fabric_ca.sdk.EnrollmentRequest;
import org.hyperledger.fabric_ca.sdk.HFCAClient;
import org.hyperledger.fabric_ca.sdk.RegistrationRequest;

public class RegisterUser {

    static {
        System.setProperty("org.hyperledger.fabric.sdk.service_discovery.as_localhost", "true");
    }

    public static void main(String[] args) throws Exception {

        // Create a CA client for interacting with the CA.
        Properties props = new Properties();
        props.put("pemFile", EnrollAdmin.CERTIFICATION_PATH);
        props.put("allowAllHostNames", "true");
        HFCAClient caClient = HFCAClient.createNewInstance(EnrollAdmin.PATH_AND_PORT, props);
        CryptoSuite cryptoSuite = CryptoSuiteFactory.getDefault().getCryptoSuite();
        caClient.setCryptoSuite(cryptoSuite);

        // Create a wallet for managing identities
        Wallet wallet = Wallet.createFileSystemWallet(Paths.get("src/main/resources/wallet"));

        // Check to see if we've already enrolled the user.
        boolean userExists = wallet.exists("yfhuang");
        if (userExists) {
            System.out.println("An identity for the user \"yfhuang\" already exists in the wallet");
            return;
        }

        userExists = wallet.exists("admin");
        if (!userExists) {
            System.out.println("\"admin\" needs to be enrolled and added to the wallet first");
            return;
        }

        Identity adminIdentity = wallet.get("admin");
        org.hyperledger.fabric.sdk.User admin = new User.Builder()
                .name("admin")
                .affiliation("org1.department1")
                .mspId("Org1MSP")
                .enrollment(adminIdentity)
                .build();

        // Register the user, enroll the user, and import the new identity into the wallet.
        RegistrationRequest registrationRequest = new RegistrationRequest("yfhuang");
        registrationRequest.setAffiliation("org1.department1");
        registrationRequest.setEnrollmentID("yfhuang");

        //register的时候设置用户自定义的属性类型
        registrationRequest.addAttribute(new Attribute("usertype","leader_car_department"));

        String enrollmentSecret = caClient.register(registrationRequest, admin);

        //enroll的时候，指定之前register的用户中的哪些属性要加入到证书中，如果不加，后买你在链码中就读取不到
        EnrollmentRequest enrollmentRequest = new EnrollmentRequest();
        enrollmentRequest.addAttrReq("hf.Affiliation");		//default attribute
        enrollmentRequest.addAttrReq("hf.EnrollmentID");	//default attribute
        enrollmentRequest.addAttrReq("hf.Type");			//default attribute
        enrollmentRequest.addAttrReq("usertype");				//user-defined attribute

        Enrollment enrollment = caClient.enroll("yfhuang", enrollmentSecret,enrollmentRequest);
        Identity user = Identity.createIdentity("Org1MSP", enrollment.getCert(), enrollment.getKey());
        wallet.put("yfhuang", user);
        System.out.println("Successfully enrolled user \"yfhuang\" and imported it into the wallet");
    }

}

```

以`yfhuang`用户进行调用

```java
package org.example;

import org.hyperledger.fabric.gateway.Contract;
import org.hyperledger.fabric.gateway.Gateway;
import org.hyperledger.fabric.gateway.Network;
import org.hyperledger.fabric.gateway.Wallet;

import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * @author csj
 * @date 2020/10/16 - 16:49
 */
public class clientCar {
    static {
        System.setProperty("org.hyperledger.fabric.sdk.service_discovery.as_localhost", "false");
    }

    public static void main(String[] args) throws Exception {
        // Load a file system based wallet for managing identities.
        Path walletPath = Paths.get("src/main/resources/wallet");
        Wallet wallet = Wallet.createFileSystemWallet(walletPath);

        // load a CCP
        Path networkConfigPath = Paths.get("src/main/resources/connection-org1.yaml");

        Gateway.Builder builder = Gateway.createBuilder();
        builder.identity(wallet, "yfhuang").networkConfig(networkConfigPath).discovery(false);

        // create a gateway connection
        try (Gateway gateway = builder.connect()) {

            // get the network and contract
            Network network = gateway.getNetwork("mychannel");
            Contract contract = network.getContract("fabcar");
            byte[] result;
           //这个方法，如果不是用户注册属性为leader_car_department，就会抛出没有权限的错误
            result = contract.evaluateTransaction("queryAllCars");
            System.out.println(new String(result));
        }
    }
}
```

可以看到，是可以查看到所有的汽车的：

```java
[{"Key":"CAR0", "Record":{"colour":"blue","make":"Toyota","model":"Prius","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"colour":"red","make":"Ford","model":"Mustang","owner":"Brad"}},{"Key":"CAR2", "Record":{"colour":"green","make":"Hyundai","model":"Tucson","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"colour":"yellow","make":"Volkswagen","model":"Passat","owner":"Max"}},{"Key":"CAR4", "Record":{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}},{"Key":"CAR5", "Record":{"colour":"purple","make":"Peugeot","model":"205","owner":"Michel"}},{"Key":"CAR6", "Record":{"colour":"white","make":"Chery","model":"S22L","owner":"Aarav"}},{"Key":"CAR7", "Record":{"colour":"violet","make":"Fiat","model":"Punto","owner":"Pari"}},{"Key":"CAR8", "Record":{"colour":"indigo","make":"Tata","model":"Nano","owner":"Valeria"}},{"Key":"CAR9", "Record":{"colour":"brown","make":"Holden","model":"Barina","owner":"Shotaro"}}]
```

以`songjian`用户进行调用：

可以看到爆出了权限错误

```java
Exception in thread "main" org.hyperledger.fabric.gateway.ContractException: 没有权限
	at org.hyperledger.fabric.gateway.impl.SingleQueryHandler.evaluate(SingleQueryHandler.java:50)
	at org.hyperledger.fabric.gateway.impl.TransactionImpl.evaluate(TransactionImpl.java:213)
	at org.hyperledger.fabric.gateway.impl.ContractImpl.evaluateTransaction(ContractImpl.java:55)
	at org.example.clientCar.main(clientCar.java:43)

Process finished with exit code 1
```

所以，基于链码的访问控制就完成了

---

最后一个小tips：

> 我们可以使用openssl工具来查看我们的证书文件，在我们使用SDK进行注册用户之后，会在相关的文件夹中生成两个文件，其中一个是私钥文件，还有一个是json类型的一个文件(其中有用户的证书certification),将证书粘贴到新建的pem文件中，然后使用spenssl工具将两个用户的证书转换成文本形式
>
> - `songjian`用户的证书
>
> ```
> -----BEGIN CERTIFICATE-----
> MIICqTCCAk+gAwIBAgIUe7kl1iz7AC+JiI4V+GQSFMXh79gwCgYIKoZIzj0EAwIw
> czELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh
> biBGcmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMT
> E2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMjAxMDE2MTAwMzAwWhcNMjExMDE2MTAw
> ODAwWjBFMTAwDQYDVQQLEwZjbGllbnQwCwYDVQQLEwRvcmcxMBIGA1UECxMLZGVw
> YXJ0bWVudDExETAPBgNVBAMTCHNvbmdqaWFuMFkwEwYHKoZIzj0CAQYIKoZIzj0D
> AQcDQgAEwbOP5V6+LAQN7FLLeR0sNXIWmyG09TfE434e/7bbi/mJs4c1n6/8h3Ix
> KCE3DOQwiH5EbHNuSa3j8XaFPIWE46OB7jCB6zAOBgNVHQ8BAf8EBAMCB4AwDAYD
> VR0TAQH/BAIwADAdBgNVHQ4EFgQUixJa/Mionju24JKEGYwytn0rasIwKwYDVR0j
> BCQwIoAgKIGbpvymakg5F6IFyeRvFQKplozygNldEr5VdR3Kc1QwfwYIKgMEBQYH
> CAEEc3siYXR0cnMiOnsiaGYuQWZmaWxpYXRpb24iOiJvcmcxLmRlcGFydG1lbnQx
> IiwiaGYuRW5yb2xsbWVudElEIjoic29uZ2ppYW4iLCJoZi5UeXBlIjoiY2xpZW50
> IiwidXNlcnR5cGUiOiJ3b3JrZXIifX0wCgYIKoZIzj0EAwIDSAAwRQIhAKr6NRaL
> aGC+NC8AWiVR6KqIbL3+A2y0tMPYP/NvsRFcAiAEB8nCmpRjYXTvekctqTtKtykL
> utw3qJfW7wCFhnp8VQ==
> -----END CERTIFICATE-----
> ```
>
> 在命令行执行：
>
> ```shell
> openssl x509 -in songjian.pem -noout -text
> ```
>
> 输出如下：
>
> ```text
> Certificate:
>     Data:
>         Version: 3 (0x2)
>         Serial Number:
>             7b:b9:25:d6:2c:fb:00:2f:89:88:8e:15:f8:64:12:14:c5:e1:ef:d8
>         Signature Algorithm: ecdsa-with-SHA256
>         Issuer: C = US, ST = California, L = San Francisco, O = org1.example.com, CN = ca.org1.example.com
>         Validity
>             Not Before: Oct 16 10:03:00 2020 GMT
>             Not After : Oct 16 10:08:00 2021 GMT
>         Subject: OU = client + OU = org1 + OU = department1, CN = songjian
>         Subject Public Key Info:
>             Public Key Algorithm: id-ecPublicKey
>                 Public-Key: (256 bit)
>                 pub:
>                     04:c1:b3:8f:e5:5e:be:2c:04:0d:ec:52:cb:79:1d:
>                     2c:35:72:16:9b:21:b4:f5:37:c4:e3:7e:1e:ff:b6:
>                     db:8b:f9:89:b3:87:35:9f:af:fc:87:72:31:28:21:
>                     37:0c:e4:30:88:7e:44:6c:73:6e:49:ad:e3:f1:76:
>                     85:3c:85:84:e3
>                 ASN1 OID: prime256v1
>                 NIST CURVE: P-256
>         X509v3 extensions:
>             X509v3 Key Usage: critical
>                 Digital Signature
>             X509v3 Basic Constraints: critical
>                 CA:FALSE
>             X509v3 Subject Key Identifier:
>                 8B:12:5A:FC:C8:A8:9E:3B:B6:E0:92:84:19:8C:32:B6:7D:2B:6A:C2
>             X509v3 Authority Key Identifier:
>                 keyid:28:81:9B:A6:FC:A6:6A:48:39:17:A2:05:C9:E4:6F:15:02:A9:96:8C:F2:80:D9:5D:12:BE:55:75:1D:CA:73:54
> 
>             1.2.3.4.5.6.7.8.1:
>                 {"attrs":{"hf.Affiliation":"org1.department1","hf.EnrollmentID":"songjian","hf.Type":"client","usertype":"worker"}}
>     Signature Algorithm: ecdsa-with-SHA256
>          30:45:02:21:00:aa:fa:35:16:8b:68:60:be:34:2f:00:5a:25:
>          51:e8:aa:88:6c:bd:fe:03:6c:b4:b4:c3:d8:3f:f3:6f:b1:11:
>          5c:02:20:04:07:c9:c2:9a:94:63:61:74:ef:7a:47:2d:a9:3b:
>          4a:b7:29:0b:ba:dc:37:a8:97:d6:ef:00:85:86:7a:7c:55
> ```
>
> - `yfhuang`的证书
>
> ```
> -----BEGIN CERTIFICATE-----
> MIICuDCCAl6gAwIBAgIULUajiYT7HTD8cJT7HHMKC9Vm7yswCgYIKoZIzj0EAwIw
> czELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh
> biBGcmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMT
> E2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMjAxMDE2MTAwMzAwWhcNMjExMDE2MTAw
> ODAwWjBEMTAwDQYDVQQLEwZjbGllbnQwCwYDVQQLEwRvcmcxMBIGA1UECxMLZGVw
> YXJ0bWVudDExEDAOBgNVBAMTB3lmaHVhbmcwWTATBgcqhkjOPQIBBggqhkjOPQMB
> BwNCAATrR6GhF0ZUBqvM2suH8GN1qVnFCCXq9gZV4iPkcTLufwKij0zVDdw1/uM7
> RsPcEYtBGv4ofRW9UErmST58XynKo4H+MIH7MA4GA1UdDwEB/wQEAwIHgDAMBgNV
> HRMBAf8EAjAAMB0GA1UdDgQWBBSglzSxtWdYHuCT/DWwmBaF1i+phjArBgNVHSME
> JDAigCAogZum/KZqSDkXogXJ5G8VAqmWjPKA2V0SvlV1HcpzVDCBjgYIKgMEBQYH
> CAEEgYF7ImF0dHJzIjp7ImhmLkFmZmlsaWF0aW9uIjoib3JnMS5kZXBhcnRtZW50
> MSIsImhmLkVucm9sbG1lbnRJRCI6InlmaHVhbmciLCJoZi5UeXBlIjoiY2xpZW50
> IiwidXNlcnR5cGUiOiJsZWFkZXJfY2FyX2RlcGFydG1lbnQifX0wCgYIKoZIzj0E
> AwIDSAAwRQIhAKUSaUMnkbvZOaPCPnzW1FxVVJMJMRepFTgCQ9Eog80bAiBKoPHL
> IJzGv2uVMmt47P+ydhm1Sw+nLHUJ1CjB7PKVgQ==
> -----END CERTIFICATE-----
> ```
>
> 在命令行执行：
>
> ```shell
> openssl x509 -in yfhuang.pem -noout -text
> ```
>
> 输出如下：
>
> ```text
> Certificate:
>     Data:
>         Version: 3 (0x2)
>         Serial Number:
>             2d:46:a3:89:84:fb:1d:30:fc:70:94:fb:1c:73:0a:0b:d5:66:ef:2b
>         Signature Algorithm: ecdsa-with-SHA256
>         Issuer: C = US, ST = California, L = San Francisco, O = org1.example.com, CN = ca.org1.example.com
>         Validity
>             Not Before: Oct 16 10:03:00 2020 GMT
>             Not After : Oct 16 10:08:00 2021 GMT
>         Subject: OU = client + OU = org1 + OU = department1, CN = yfhuang
>         Subject Public Key Info:
>             Public Key Algorithm: id-ecPublicKey
>                 Public-Key: (256 bit)
>                 pub:
>                     04:eb:47:a1:a1:17:46:54:06:ab:cc:da:cb:87:f0:
>                     63:75:a9:59:c5:08:25:ea:f6:06:55:e2:23:e4:71:
>                     32:ee:7f:02:a2:8f:4c:d5:0d:dc:35:fe:e3:3b:46:
>                     c3:dc:11:8b:41:1a:fe:28:7d:15:bd:50:4a:e6:49:
>                     3e:7c:5f:29:ca
>                 ASN1 OID: prime256v1
>                 NIST CURVE: P-256
>         X509v3 extensions:
>             X509v3 Key Usage: critical
>                 Digital Signature
>             X509v3 Basic Constraints: critical
>                 CA:FALSE
>             X509v3 Subject Key Identifier:
>                 A0:97:34:B1:B5:67:58:1E:E0:93:FC:35:B0:98:16:85:D6:2F:A9:86
>             X509v3 Authority Key Identifier:
>                 keyid:28:81:9B:A6:FC:A6:6A:48:39:17:A2:05:C9:E4:6F:15:02:A9:96:8C:F2:80:D9:5D:12:BE:55:75:1D:CA:73:54
> 
>             1.2.3.4.5.6.7.8.1:
>                 {"attrs":{"hf.Affiliation":"org1.department1","hf.EnrollmentID":"yfhuang","hf.Type":"client","usertype":"leader_car_department"}}
>     Signature Algorithm: ecdsa-with-SHA256
>          30:45:02:21:00:a5:12:69:43:27:91:bb:d9:39:a3:c2:3e:7c:
>          d6:d4:5c:55:54:93:09:31:17:a9:15:38:02:43:d1:28:83:cd:
>          1b:02:20:4a:a0:f1:cb:20:9c:c6:bf:6b:95:32:6b:78:ec:ff:
>          b2:76:19:b5:4b:0f:a7:2c:75:09:d4:28:c1:ec:f2:95:81
> PS C:\Users\Bun\Desktop> openssl x509 -in songjian.pem -noout -text
> Certificate:
>     Data:
>         Version: 3 (0x2)
>         Serial Number:
>             7b:b9:25:d6:2c:fb:00:2f:89:88:8e:15:f8:64:12:14:c5:e1:ef:d8
>         Signature Algorithm: ecdsa-with-SHA256
>         Issuer: C = US, ST = California, L = San Francisco, O = org1.example.com, CN = ca.org1.example.com
>         Validity
>             Not Before: Oct 16 10:03:00 2020 GMT
>         Subject: OU = client + OU = org1 + OU = department1, CN = songjian
>         Subject Public Key Info:
>             Public Key Algorithm: id-ecPublicKey
>                 Public-Key: (256 bit)
>                 pub:
>                     04:c1:b3:8f:e5:5e:be:2c:04:0d:ec:52:cb:79:1d:
>                     2c:35:72:16:9b:21:b4:f5:37:c4:e3:7e:1e:ff:b6:
>                     db:8b:f9:89:b3:87:35:9f:af:fc:87:72:31:28:21:
>                     37:0c:e4:30:88:7e:44:6c:73:6e:49:ad:e3:f1:76:
>                     85:3c:85:84:e3
>                 ASN1 OID: prime256v1
>                 NIST CURVE: P-256
>         X509v3 extensions:
>             X509v3 Key Usage: critical
>                 Digital Signature
>             X509v3 Basic Constraints: critical
>                 CA:FALSE
>             X509v3 Subject Key Identifier:
>                 8B:12:5A:FC:C8:A8:9E:3B:B6:E0:92:84:19:8C:32:B6:7D:2B:6A:C2
>             X509v3 Authority Key Identifier:
>                 keyid:28:81:9B:A6:FC:A6:6A:48:39:17:A2:05:C9:E4:6F:15:02:A9:96:8C:F2:80:D9:5D:12:BE:55:75:1D:CA:73:54
> 
>             1.2.3.4.5.6.7.8.1:
>                 {"attrs":{"hf.Affiliation":"org1.department1","hf.EnrollmentID":"songjian","hf.Type":"client","usertype":"worker"}}
>     Signature Algorithm: ecdsa-with-SHA256
>          30:45:02:21:00:aa:fa:35:16:8b:68:60:be:34:2f:00:5a:25:
>          51:e8:aa:88:6c:bd:fe:03:6c:b4:b4:c3:d8:3f:f3:6f:b1:11:
>          5c:02:20:04:07:c9:c2:9a:94:63:61:74:ef:7a:47:2d:a9:3b:
>          4a:b7:29:0b:ba:dc:37:a8:97:d6:ef:00:85:86:7a:7c:55
> PS C:\Users\Bun\Desktop> openssl x509 -in yfhuang.pem -noout -text
> Certificate:
>     Data:
>         Version: 3 (0x2)
>         Serial Number:
>             2d:46:a3:89:84:fb:1d:30:fc:70:94:fb:1c:73:0a:0b:d5:66:ef:2b
>         Signature Algorithm: ecdsa-with-SHA256
>         Issuer: C = US, ST = California, L = San Francisco, O = org1.example.com, CN = ca.org1.example.com
>         Validity
>             Not Before: Oct 16 10:03:00 2020 GMT
>             Not After : Oct 16 10:08:00 2021 GMT
>         Subject: OU = client + OU = org1 + OU = department1, CN = yfhuang
>         Subject Public Key Info:
>             Public Key Algorithm: id-ecPublicKey
>                 Public-Key: (256 bit)
>                 pub:
>                     04:eb:47:a1:a1:17:46:54:06:ab:cc:da:cb:87:f0:
>                     63:75:a9:59:c5:08:25:ea:f6:06:55:e2:23:e4:71:
>                     32:ee:7f:02:a2:8f:4c:d5:0d:dc:35:fe:e3:3b:46:
>                     c3:dc:11:8b:41:1a:fe:28:7d:15:bd:50:4a:e6:49:
>                     3e:7c:5f:29:ca
>                 ASN1 OID: prime256v1
>                 NIST CURVE: P-256
>         X509v3 extensions:
>             X509v3 Key Usage: critical
>                 Digital Signature
>             X509v3 Basic Constraints: critical
>                 CA:FALSE
>             X509v3 Subject Key Identifier:
>                 A0:97:34:B1:B5:67:58:1E:E0:93:FC:35:B0:98:16:85:D6:2F:A9:86
>             X509v3 Authority Key Identifier:
>                 keyid:28:81:9B:A6:FC:A6:6A:48:39:17:A2:05:C9:E4:6F:15:02:A9:96:8C:F2:80:D9:5D:12:BE:55:75:1D:CA:73:54
> 
>             1.2.3.4.5.6.7.8.1:
>                 {"attrs":{"hf.Affiliation":"org1.department1","hf.EnrollmentID":"yfhuang","hf.Type":"client","usertype":"leader_car_department"}}
>     Signature Algorithm: ecdsa-with-SHA256
>          30:45:02:21:00:a5:12:69:43:27:91:bb:d9:39:a3:c2:3e:7c:
>          d6:d4:5c:55:54:93:09:31:17:a9:15:38:02:43:d1:28:83:cd:
>          1b:02:20:4a:a0:f1:cb:20:9c:c6:bf:6b:95:32:6b:78:ec:ff:
>          b2:76:19:b5:4b:0f:a7:2c:75:09:d4:28:c1:ec:f2:95:81
> ```
>
> 可以观察到上面两个证书转换之后的文本中最后的attrs中的usertype是不一样的，也就是说明，我们的usertype已经enroll到证书中了，也就能在链码中读取了
>
> 我们再来看一个在注册用户的时候Enroll的时候没有指定将自定义的属性加入证书的例子(注意一个haha用户)
>
> ```java
> /*
> SPDX-License-Identifier: Apache-2.0
> */
> 
> package org.example;
> 
> import java.nio.file.Paths;
> import java.util.Properties;
> 
> import org.example.Common.User;
> import org.hyperledger.fabric.gateway.Wallet;
> import org.hyperledger.fabric.gateway.Wallet.Identity;
> import org.hyperledger.fabric.sdk.Enrollment;
> import org.hyperledger.fabric.sdk.security.CryptoSuite;
> import org.hyperledger.fabric.sdk.security.CryptoSuiteFactory;
> import org.hyperledger.fabric_ca.sdk.Attribute;
> import org.hyperledger.fabric_ca.sdk.EnrollmentRequest;
> import org.hyperledger.fabric_ca.sdk.HFCAClient;
> import org.hyperledger.fabric_ca.sdk.RegistrationRequest;
> 
> public class RegisterUser {
> 
>     static {
>         System.setProperty("org.hyperledger.fabric.sdk.service_discovery.as_localhost", "true");
>     }
>     public static void main(String[] args) throws Exception {
>         // Create a CA client for interacting with the CA.
>         Properties props = new Properties();
>         props.put("pemFile", EnrollAdmin.CERTIFICATION_PATH);
>         props.put("allowAllHostNames", "true");
>         HFCAClient caClient = HFCAClient.createNewInstance(EnrollAdmin.PATH_AND_PORT, props);
>         CryptoSuite cryptoSuite = CryptoSuiteFactory.getDefault().getCryptoSuite();
>         caClient.setCryptoSuite(cryptoSuite);
> 
>         // Create a wallet for managing identities
>         Wallet wallet = Wallet.createFileSystemWallet(Paths.get("src/main/resources/wallet"));
> 
>         // Check to see if we've already enrolled the user.
>         boolean userExists = wallet.exists("haha");
>         if (userExists) {
>             System.out.println("An identity for the user \"haha\" already exists in the wallet");
>             return;
>         }
> 
>         userExists = wallet.exists("admin");
>         if (!userExists) {
>             System.out.println("\"admin\" needs to be enrolled and added to the wallet first");
>             return;
>         }
>         Identity adminIdentity = wallet.get("admin");
>         org.hyperledger.fabric.sdk.User admin = new User.Builder()
>                 .name("admin")
>                 .affiliation("org1.department1")
>                 .mspId("Org1MSP")
>                 .enrollment(adminIdentity)
>                 .build();
>         RegistrationRequest registrationRequest = new RegistrationRequest("haha");
>         registrationRequest.setAffiliation("org1.department1");
>         registrationRequest.setEnrollmentID("haha");
>         //register的时候设置用户自定义的属性类型
>         registrationRequest.addAttribute(new Attribute("usertype","leader_car_department"));
>         String enrollmentSecret = caClient.register(registrationRequest, admin);
>         Enrollment enrollment = caClient.enroll("haha", enrollmentSecret);
>         Identity user = Identity.createIdentity("Org1MSP", enrollment.getCert(), enrollment.getKey());
>         wallet.put("haha", user);
>         System.out.println("Successfully enrolled user \"haha\" and imported it into the wallet");
>     }
> }
> ```
>
> 下面是这个用户的证书：
>
> ```
> -----BEGIN CERTIFICATE-----
> MIICjDCCAjOgAwIBAgIUJrH6/14VpN4DZAG/b3H7TwFxm0wwCgYIKoZIzj0EAwIw
> czELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh
> biBGcmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMT
> E2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMjAxMDE2MTEwMzAwWhcNMjExMDE2MTEw
> ODAwWjBBMTAwDQYDVQQLEwZjbGllbnQwCwYDVQQLEwRvcmcxMBIGA1UECxMLZGVw
> YXJ0bWVudDExDTALBgNVBAMTBGhhaGEwWTATBgcqhkjOPQIBBggqhkjOPQMBBwNC
> AATSOOKZ0tB7IRD97e+G8Rbka058hcLr+euj1WHNLipOOKeBvLFocCKMML2dL8hP
> 2mkndM1zj00YAkVP8yTwO97zo4HWMIHTMA4GA1UdDwEB/wQEAwIHgDAMBgNVHRMB
> Af8EAjAAMB0GA1UdDgQWBBSuIp0UQPrfvmgbJ5iNhOZUh2e/1zArBgNVHSMEJDAi
> gCAogZum/KZqSDkXogXJ5G8VAqmWjPKA2V0SvlV1HcpzVDBnBggqAwQFBgcIAQRb
> eyJhdHRycyI6eyJoZi5BZmZpbGlhdGlvbiI6Im9yZzEuZGVwYXJ0bWVudDEiLCJo
> Zi5FbnJvbGxtZW50SUQiOiJoYWhhIiwiaGYuVHlwZSI6ImNsaWVudCJ9fTAKBggq
> hkjOPQQDAgNHADBEAiAYderD2wOsojvdK5k1zYyDLtK73KIYLS5dim02Oz3+BQIg
> DjZ1y7kctNRyWJ3uXPJlZWISnwD3fpzp8QccIKE5bUY=
> -----END CERTIFICATE-----
> ```
>
> 使用openssl工具，执行：
>
> ```shell
> openssl x509 -in haha.pem -noout -text
> ```
>
> 转换得到文本
>
> ```
> Certificate:
>     Data:
>         Version: 3 (0x2)
>         Serial Number:
>             26:b1:fa:ff:5e:15:a4:de:03:64:01:bf:6f:71:fb:4f:01:71:9b:4c
>         Signature Algorithm: ecdsa-with-SHA256
>         Issuer: C = US, ST = California, L = San Francisco, O = org1.example.com, CN = ca.org1.example.com
>         Validity
>             Not Before: Oct 16 11:03:00 2020 GMT
>             Not After : Oct 16 11:08:00 2021 GMT
>         Subject: OU = client + OU = org1 + OU = department1, CN = haha
>         Subject Public Key Info:
>             Public Key Algorithm: id-ecPublicKey
>                 Public-Key: (256 bit)
>                 pub:
>                     04:d2:38:e2:99:d2:d0:7b:21:10:fd:ed:ef:86:f1:
>                     16:e4:6b:4e:7c:85:c2:eb:f9:eb:a3:d5:61:cd:2e:
>                     2a:4e:38:a7:81:bc:b1:68:70:22:8c:30:bd:9d:2f:
>                     c8:4f:da:69:27:74:cd:73:8f:4d:18:02:45:4f:f3:
>                     24:f0:3b:de:f3
>                 ASN1 OID: prime256v1
>                 NIST CURVE: P-256
>         X509v3 extensions:
>             X509v3 Key Usage: critical
>                 Digital Signature
>             X509v3 Basic Constraints: critical
>                 CA:FALSE
>             X509v3 Subject Key Identifier:
>                 AE:22:9D:14:40:FA:DF:BE:68:1B:27:98:8D:84:E6:54:87:67:BF:D7
>             X509v3 Authority Key Identifier:
>                 keyid:28:81:9B:A6:FC:A6:6A:48:39:17:A2:05:C9:E4:6F:15:02:A9:96:8C:F2:80:D9:5D:12:BE:55:75:1D:CA:73:54
> 
>             1.2.3.4.5.6.7.8.1:
>                 {"attrs":{"hf.Affiliation":"org1.department1","hf.EnrollmentID":"haha","hf.Type":"client"}}
>     Signature Algorithm: ecdsa-with-SHA256
>          30:44:02:20:18:75:ea:c3:db:03:ac:a2:3b:dd:2b:99:35:cd:
>          8c:83:2e:d2:bb:dc:a2:18:2d:2e:5d:8a:6d:36:3b:3d:fe:05:
>          02:20:0e:36:75:cb:b9:1c:b4:d4:72:58:9d:ee:5c:f2:65:65:
>          62:12:9f:00:f7:7e:9c:e9:f1:07:1c:20:a1:39:6d:46
> ```
>
> 可以看到attrs中并没有加入usertype

当然链码的控制还可以使用断言的方式进行

好了，大功告成！！！！











