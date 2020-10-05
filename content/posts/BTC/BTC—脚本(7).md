---
title: "BTC—脚本(7)"
date: 2020-08-23T20:09:50+08:00
tags:
- 区块链
Categories:
- 比特币
---

# 比特币的交易

![image-20200728192656381](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200728192656381.png)



# 交易结构

![image-20200728193228573](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200728193228573.png)



![image-20200728193557698](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200728193557698.png)

![image-20200728194144832](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200728194144832.png)



![image-20200728194802246](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200728194802246.png)

实际中为了安全起见脚本都是单独执行的！！！



# 输入输出脚本的形式

### （1）P2PK(最简单):

![](D:\项目资料\study_notes\区块链公开课——北大肖臻\p2pk.png)

![image-20200801172815056](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801172815056.png)

从上往下一次入栈，CHRECKSIG是将前两个入栈的弹出，如果正确就返回TRUE,说明验证通过。



![image-20200801173158962](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801173158962.png)



### （2）P2PKH(最常用的)：

![](D:\项目资料\study_notes\区块链公开课——北大肖臻\p2pkh.png)

这个与P2PK的不同在于，输出脚本中并没有给出公钥，而是给出的公钥的hash，公钥放在了输入脚本中。

![image-20200801173930129](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801173930129.png)

执行细节如下：（注意：**前两个脚本是输入脚本，后面的脚本是输出脚本**）

![image-20200801174548528](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801174548528.png)

上述7个脚本从上往下依次执行， 首先1和2脚本入栈，然后DUP的作用是复制一份PubKey然后压栈，HASH160作用是将栈顶的PubKey取hash，然后压栈，第五个脚本是将公钥的hash（收款人公钥的hash）入栈，第5个脚本是对栈顶的两个PubKeyHash进行判断，如果两个Hash值相等，那么这两个Hash值就消失了（**目的是防止有人冒名顶替，用他自己的公钥冒充收款人的公钥**），然后执行最后一步，用公钥来判断签名，是否正确，如果正确就返回TRUE。

![image-20200801175758383](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801175758383.png)



### （3）P2SH:

输出脚本给出的不是收款人公钥的hash，而是**收款人提供的脚本(redeem script)的hash**

![image-20200801180437621](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801180437621.png)

![image-20200801180637566](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801180637566.png)

![image-20200801180733938](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801180733938.png)

![image-20200801181050100](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801181050100.png)

**执行的详细过程如下：（最上面两个是输入脚本，下面的都是输出脚本的内容）**

![image-20200801181417735](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801181417735.png)

首先执行最上面两个脚本，先入栈，然后HASH160，对栈顶脚本取hash，压栈，第四步就是将收款人给出的赎回脚本压栈，然后执行EQUAL操作，比较两个hash是否相等，如果不相等就结束，如果相等就消失，最后剩下Sig签名。

![image-20200801181907620](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801181907620.png)

在第一阶段的基础上，执行赎回脚本（redeem script），也就是将公钥的hash入栈，然后执行CHECKS，使用公钥来验证签名，正确无误就返回TRUE。

![image-20200801182230825](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801182230825.png)

#### 为什么要用P2SH?

![image-20200801183241108](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801183241108.png)

![image-20200801183341826](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801183341826.png)

最后执行CHECKMULTISIG验证栈中是不是含有三个签名中的两个，如果是，那么验证通过。

注意：**这个过程中并没有用到Hash，就是用原生的多重签名实现的。但是这也带来了复杂性，带来了不方便**

例如：某个用户取网上购物，某个电商使用多重签名，要求必须要有5个合伙人中的三个人进行签名才能把钱取出来，这就要求购物的用户在进行网上购物时，给出这5个合伙人的公钥，同时还要给出N和M（分别代表几个公钥中至少有几个公钥才能合法，例如N=5,M=3代表5个合伙人中必须三个人进行签名，电商需要在网上公布出来，用户可以看到然后在进行转账交易的时候，就把这些信息填进去，不同的电商采用的多重签名的规则是不一样的，即N和M不一样，这就给用户生成转账交易带来不方便的地方，因为这些复杂性都暴露给用户了）

**所以就需用P2SH实现多重签名（本质是把复杂度从输出脚本转移到了输入脚本）**

![image-20200801192559444](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801192559444.png)

输出脚本中只需要提供赎回脚本的hash，输入脚本中需要提供赎回脚本(其中包含N和公钥和N与M的值)，这个赎回脚本是由收款人提供的（前面提到的）

再回到前面的例子：收款人是电商，他只要在网上公布赎回脚本的hash值,然后用户生成转账交易的时候，只需要把这个赎回脚本的hash值包含在交易的输出脚本中即可，至于这个电商用什么样的多重签名规则，对于用户来说是不可见的，用户没必要知道。采用这种方式与之前说的P2PKH没有什么区别（只不过把公钥的hash值改成了赎回脚本的hash值）

![image-20200801193332388](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801193332388.png)

![image-20200801193750202](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801193750202.png)

以上是执行的第一阶段，然后进行第二阶段。

![image-20200801193840212](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801193840212.png)

最后验证多重签名的有效性，正确返回TRUE。

![image-20200801194050454](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801194050454.png)

**现在的多重签名基本都是采用的这种P2SH的形式**



### （4）Proof of Burn

![image-20200801194207991](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801194207991.png)

这种销毁比特币的特性适用于以下几种场景：

（1）为了得到altcoin（alternative coin）这种小币种，有的币要求销毁一个BTC就能获得其一部分币，这样确保是付出代价的。

（2）向区块链中永久写入内容。可以内容写入RETURN语句的后面，类似于coinbase域写入内容（两者虽然都可以写入一部分内容，但是有些不一样：RETURN语句不论是哪个节点都可以写入，但是coinbase域只有获得记账权的矿工才能写入信息）

下面是是一个**Coinbase交易**：

![image-20200801195558212](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801195558212.png)



下面是一个普通的交易：

![image-20200801195757733](C:\Users\Bun\AppData\Roaming\Typora\typora-user-images\image-20200801195757733.png)