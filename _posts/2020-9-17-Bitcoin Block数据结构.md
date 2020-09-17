---
layout: post
title: Bitcoin Block数据结构
categories: [区块链]
---

## 0

最近重新看了区块链的相关内容，读了些资料，感觉知识不够系统，所以在这里总结总结，先总结区块的数据结构，小黄鸭debug~

## 1 从一条交易的产生说起

当一条转账交易(Transaction)发生后，它首先s's会被存到网络节点的内存池(Memory Pool/Transaction Pool)中，然后节点从自己的内存池选择一些交易，构造自己的候选块(candidate block)，并且努力通过自己的算力将其成功添加到区块链上，也即挖矿（mining）。**也就是说每个区块中其实是很多条成功添加到区块上的交易记录**。

因此，遵循了**区块结构 > 交易记录结构**的顺序递进记录，先画一个区块框架，再慢慢填满这个块。

## 2 Block Header

首先粗略地把一个区块划分为两大部分：block header和余下的block body。

![2020-9-17-1](\assets\2020-9-17-1.png)

六个字段均为小端存储
#### *1.Version*  (4 bytes) 

> 区块版本


#### *2.Previous Block Hash*  (32 bytes)
> 前一区块的hash

##### Block Hash

通过使用SHA256算法对block header做两次哈希所得，能标识一个区块。为什么不对block body一起做Hash而仅仅Hash block header就行？我个人理解，因为Merkel Root即可表示block body中的transactions。 

#### *3. Merkel Root* （32 bytes）

> 哈希一个块中成对的TX ID，最终得到一个根节点，这个根节点就是Merkel Root。

看图就十分直观啦~可以看到Merkel Tree只有叶子节点是Tx ID，中间的节点都是计算Merkel Root的桥梁。

![2020-9-17-2](\assets\2020-9-17-2.png)

知其然容易，更要知其所以然，那么，**为什么要构造一个树来计算所有Tx ID的Hash，而不是直接Hash呢？**

如果是所有Tx ID直接一起Hash，那么当需要验证某一个Tx ID是不是属于这个Merkel Root的时候，就需要获取所有的Tx ID再一起Hash，和原Hash对比；但采用Merkel Tree的方法，我们只需要沿着这个Tx ID的分支，获取他的相邻节点（Merkel Proof），就可以计算得到Merkel Root。

由此也可以引出**轻节点**和**全节点**概念：

简单来说，只存储块头的节点就是轻节点，所有区块都存储的节点就是全节点。
当轻节点需要查看一个交易是否成功存入区块，就可以通过块头中的Merkel Root字段来验证（其余计算需要的Merkel Prof来自于全节点）

####  *4.Time* (4 bytes)

> 区块生成时的时间戳

#### *5. Bits* (4 bytes)

> 压缩格式的目标Hash(Target)

具体的压缩方法参考[这里](https://learnmeabitcoin.com/technical/bits)

#### *6. Nonce* (4 bytes)

> 一个Candidate Block能够脱颖而出，成功添加到链上的条件就是该block header的hash比target小，而实现这个的方法就是不断调整nonce的值。

## 3 Transaction

block header大概记录完了，再来说说剩下的部分，首先是transaction，每个区块中包含很多transaction，每一条结构都是相同的。

首先要说明的一点是比特币其实不是像数字支付一样，可以刚好给出某个特定金额而不用找零，更形象地说，比特币交易更像是纸币交易，你拿出你手头有的钱给对方，同时要把剩余的钱再找给自己。也就是说，转帐就是把你账户上一笔或者几笔未花过的交易作为输入（input），将其分给别的账户（output）的过程。input要大于output之和，多出来的部分就是矿工成功挖矿所得。

> **note**:如果想将某一部分多余的input“找零”，那么只需建立一个地址是自己的output即可。**即，一个input是不可分的，但你可以再建立一个流向自己的output来解决这个不可分。**

这其中涉及到一个概念[UTXO](https://learnmeabitcoin.com/technical/utxo).

### Transaction ID(Tx ID)

使用SHA256两次Hash transaction data就可以获得Tx ID(与计算block hash的方法一样)

### Transaction Data

一条transaction包含六个字段，示意图如下：

![2020-9-17-3](\assets\2020-9-17-3.png)

#### *1.Version* (4 bytes)

> 指明使用的是哪个版本的数据结构

#### *2. Input Count* ([长度不固定](https://learnmeabitcoin.com/technical/varint))

>指明一共有几个input

#### *3. Input(s)*

一个input包含5个字段，如图：

![2020-9-17-4](\assets\2020-9-17-4.png)

* TXID(32 bytes)：指明input来自哪条交易 。
* Vout (4 bytes)：指明input的序号（是第几个input）。
* ScriptSig Size (长度不固定)：指明ScriptSig代码的长度。
* ScriptSig：解锁脚本。

#### *4. Output Count* （长度不固定)

> 指明一共有借output

#### *5. Output(s)*

一共有三个字段：

![2020-9-17-5](\assets\2020-9-17-5.png)

* Value(8 bytes)：satoshis的数量。
* ScriptPubKey Size：加密脚本的长度。
* ScriptPublicKey：加密脚本

#### *6. Locktime*

> 设置可以包含此事务的最小块高度

## 3 Block

block header和transaction的数据结构都在上面记录了，下面就能补全整个block的数据结构啦~p.s.他俩之间还有一个transaction count（也是长度可变的）。

也就是一个Block =

 Block Header || Tx Count || Transaction1 || Transaction2 ||...

具体结构图参照[这里](https://www.arcblock.io/zh/post/2018/08/16/index-bitcoin)

> **note**:
>
> Block Hash,Block Height,Tx Hash(ID),Address因为计算可得，因此都不显示的存储在block里，计算方法仍然参照上面的[链接](https://www.arcblock.io/zh/post/2018/08/16/index-bitcoin)。

## · References

1. https://learnmeabitcoin.com/

2. https://www.arcblock.io/zh/post/2018/08/16/index-bitcoin
3. https://bitcoin.stackexchange.com/questions/11054/how-do-spv-simple-payment-verification-wallets-learn-about-incoming-transactio
4. 图一图二来自：https://learnmeabitcoin.com/

