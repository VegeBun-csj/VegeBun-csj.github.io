---
title: "BTC—数据结构(2)"
date: 2020-08-23T20:06:04+08:00
tags:
- 区块链
Categories:
- 比特币
---

## 1.hash指针

普通的指针存储的是某个结构体在内存中的起始地址，但是hash地址除了存储这个地址，还存储结构体内容的hash，所以hash指针不仅可以找到地址还能检测出结构体内容是否被篡改



#### 区块链与普通的链表的区别：

区块链使用的是hash指针而不是普通的指针，注意hash指针的hash内容是整个区块的hash（也就是上一个区块中所有的内容，包括其中的hash指针），通过这种方式可以实现tamper-evident log，普通链表是可以改变其中一个区块的元素，对后面的区块没有影响，但是区块链不一样，只要其中一个区块发生变化，后面的区块后会发生变化，牵一发动全身，引发多米诺骨牌效应（因为后一个区块记录了上一个区块的hash）



## 2.Merkel tree

默克尔树与二叉排序树的区别就是用hash指针代替了普通的指针。

![Merkle树结构](/image/BTC/Merkle.png)

最底层是区块数据（上面的都是hash指针），然后将这些区块做hash，成为Merkel树的叶子节点，所以Merkle树也称为Hash Tree。最上面的被称为根hash（root hash）。

Merkle树的这种结构被用来用做Merkle proof。



## 3.区块结构

区块分为区块头，区块体

（1）区块头(Block Header)：存放了区块体中所有交易组成的Merkel Tree的根hash值

（2）区块体(Block body)：其中存放了交易Tx列表

比特币中的节点分为全节点和轻节点，全节点是拥有区块头和区块体的节点，而轻节点只含有区块头，也就是没有交易列表（无法立即验证交易的有效性/存在），所以对于轻节点想要验证交易时，需要向全节点获取一些信息来进行验证，此时就会用到Merkle Tree提供的Merkle proof

![Merkle proof](/image/BTC/Merkle_proof.png)

当要验证图中的待证明的Tx时，需要从全节点获取其到根节点路径上的hash值（红色hash），从而计算出根hash，然后将这个计算出来的根hash与轻节点自己的区块头中保存的hash进行对照，如果相同认定交易时有效的，这就是Merkle proof的过程。复杂度是O(logn)