---
layout: post
title: geth & Solidity合约的部署和调用
categories: [学习笔记]
---



## 1. 碎碎念

最近区块链课上布置了个实验，大概就是用Solidity编写一个投票智能合约然后部署到geth私有链。之前没怎么学以太坊，借这次实验记录一下学到的知识和实验过程，以及尚未解决的问题。

## 2. 以太坊客户端geth常用命令

这次实验做下来发现有那么几个命令是十分常用的：

* **启动私有链**

```cmd
> geth --rpc --nodiscover --datadir data0  --port 30303 --rpc --rpcport 8545 --rpcapi "db,eth,net,web3" --rpccorsdomain "*" --networkid 1108 --ipcdisable console --allow-insecure-unlock
```



| 命令                    | 作用                                                         |
| ----------------------- | ------------------------------------------------------------ |
| rpc                     | 启用rpc服务，默认端口号8545                                  |
| nodiscover              | 关闭p2p网络的自动发现，需要手动添加节点，有利于隐藏私有网络  |
| datadir                 | 设置当前区块链网络数据存放的位置                             |
| port                    | 设置网络监听端口                                             |
| rpcapi                  | 可以通过rpc调用的对象                                        |
| rpccorsdomain           | 指定可以访问APi的域名地址，设置为“*”则任何地址都可以访问，这样做不安全 |
| networkid               | 网络标识，私有链取一个大于4的任意值                          |
| --ipcdisable            | 禁用IPC-RPC服务器（默认是打开的）                            |
| --allow-insecure-unlock | 允许不安全解锁                                               |

* 创建新账户

```cmd
> personal.newAccount() //接下来设置密码
> personal.newAccount("passwd")//双引号中设置密码
```

* 解锁账户

```cmd
> personal.unlockAccount(eth.accounts[0])
```

* 查看账户

```cmd
> eth.accounts //查看所有账户
> eth.accounts[0] //查看某一个账户
```

* 查看账户余额

```cmd
eth.getBalance("address")
eth.getBalance(eth.accounts[0])
```

* 转账三个比特币

```cmd
eth.sendTransaction({from:"addressfrom",to:"addressto",value:web3.toWei(3,"ether")})
```

* 挖矿&停止

```cmd
miner.start()
miner.stop()
```

## 3. Solidity智能合约编写

这里主要是看的《精通以太坊智能合约开发》以及*study by example*。其实solidity和javascript还是很像的，比较好上手。

## 4. 合约部署

我使用Remix在线IDE编写的合约，在IDE里编译后可以拿到这个程序的ABI接口和比特码。用这两样就可以把合约部署在以太坊私有链上啦~

首先拿到ABI接口：

```cmd
> abi=[{},{},......,{}]
```

拿到ABI接口后就可以定义一个合约类：

```cmd
> mycontract=eth.contract(abi)
```

部署合约还需要使用合约的十六进制码，但是它太长了，直接用太太太不方便了，就先定义一个变量myhex来存储它：

```cmd
> myhex="0x........."
```

准备工作完成以后就可以开始部署合约啦。用账户0来部署合约，首先要解锁一下账户0，然后部署：

```cmd
> votecontract=mycontract.new({from:eth.accounts[0],data:Myhex,gas:3000000})
```

这一步之后会得到这个交易的hash, *TransactionHash*

合约部署其实是一个交易，所以要挖矿确认一下。可以用`txpool.status`查看交易池里是否有待确认的交易。

然后通过交易哈希获得合约的地址*ContractAddress*，用地址给合约命名：

```cmd
> recpt=eth.getTransactionReceipt("TransactionHash")
> myvote=mycontract.at("ContractAddress")
```

到此合约的部署就结束了，后面可以通过合约名称调用合约。

## 5. 调用合约

### 以太坊中主要有三种合约调用方法

* myvote.function.sendTransaction(parameter1,parameter2,...,{from:" "});

  > 这种调用方法会创建一个交易，返回一个交易哈希，会广播到网络等待矿工打包。创建交易会消耗gas。

* myvote.function.call(parameter1,parameter2,...,{from:" "});

  > 这种调用是一个本地调用，不向区块链广播，返回值取决于方法的代码逻辑，不消耗gas。

* myvote.function(parameter1,parameter2,...,{from:" "});

  > 这个调用根据方法的constant标识来执行，有constant标识说明不会修改状态变量，web3.js会执行call()操作；反之，没有constant表示说明会修改状态变量，就会执行sendTransacton()操作

