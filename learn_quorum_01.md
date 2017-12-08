# 零起点玩转基于以太坊的联盟链Quorum系列01 #

----------

## 简介 ##

- Quorum是由美国的金融机构摩根大通推出的企业级区块链平台
- Quorum是一个联盟链
- Quorum基于以太坊, 在基础结构上保持与以太坊的同步, 增强的部分集中在隐私控制、权限控制、共识机制，以及提高整个链的交易性能

## 特点 ##

### 逻辑架构 ###
![逻辑架构](https://raw.githubusercontent.com/jpmorganchase/quorum-docs/master/images/Quorum%20Architecture.JPG)

从上面的逻辑架构图可以看出一个Quorum结点由两部分组成:1.基于以太坊的QuorumNode 2.Quorum增加的Constellation.
Quorum最初设计的实现就是对以太坊geth的一个minimal fork, 官方的一个实践准则就是保持对以太坊geth的持续跟进.
在共识方法上, Quorum用QuorumChain取代了以太坊的POW, 最新推出的2.0.0又增加了Raft的共识方法, 最终的目标就是形成一个插件化的共识体系.

Quorum作为企业级联盟链, 强调隐私控制与权限管理, Constellation就是用于这一个目标的组件.Constellation中的Transaction Manager用于管理私有交易在联盟内部成员之间的同步, Enclave的职责就是配合Transaction Manager来完成私有交易的密码学相关的处理工作.

### 交易流程 ###
![交易流程](https://github.com/jpmorganchase/quorum-docs/raw/master/images/QuorumTransactionProcessing.JPG)

结合上面的交易流程示意图, 一次Quorum的交易流程分成两种情况:

1. 如果这是一个公开我交易, 那么这个交易直接走以太坊的交易流程, 交易不会经由Transaction Manager来处理.
2. 如果这是一个私有交易, 那么这个交易就会被QuorumNode发送到Transaction Manager, 进行私有交易的处理(具体细节会在以后的系统中结合实例代码与演示加以剖析).

## **实验** - 跑起本地结点 ##


*实验环境:Deepin Linux, go 1.9.1*

#### 生成Quorum各个组件 ####

1. **获取Quorum**

    `git clone --recursive https://github.com/jpmorganchase/quorum.git`

2. **构建Quorum**

    `cd quorum`

	`make all`

3. **设置编译生成文件**

	第2步执行成功之后, quorum/build/bin目录下会生成如下文件列表

    abigen	bootnode  evm  examples  faucet  geth  p2psim  puppeth	rlpdump  swarm	wnode
	
	将上述编译生成的文件所在目录的完整路径添加到系统PATH中去

4. **获取constellation-node可执行文件**

	wget https://github.com/jpmorganchase/constellation/releases/download/v0.2.0/constellation-0.2.0-ubuntu1604.tar.xz

	解压之后解压得到的文件constellation-node, 执行第3步同样的操作将constellation-node添到系统PATH中去

*注: 完成第4步的操作之后, 本地结点所需要的二进制可执行文件已经准备完成, 下面的任务是生成运行本地结点所需要的配置文件.*

#### 生成启动本地结点的配置文件 ####

1. 创建实验的工作目录work

    `mkdir work`

2. 创建创世区块的配置文件

	`nano genesis.json`

	写入以下内容
	
	`{
	  "alloc": {},
	  "coinbase": "0x0000000000000000000000000000000000000000",
	  "config": {
	    "homesteadBlock": 0,
	    "chainId": 1,
	    "eip155Block": null,
	    "eip158Block": null,
	    "isQuorum": true
	  },
	  "difficulty": "0x0",
	  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000",
	  "gasLimit": "0xE0000000",
	  "mixhash": "0x00000000000000000000000000000000000000647572616c65787365646c6578",
	  "nonce": "0x0",
	  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
	  "timestamp": "0x00"
	}`
	
	*注: 对于此配置文件各字段的具体解释将在后续系列中加以剖析*

3. 创建结点运行时存放数据的目录树

    `mkdir -p qdata/logs`

	`mkdir -p qdata/node1/keystore`

	`mkdir -p qdata/node1/geth`

	`mkdir -p qdata/node1/keys`

4. 创建一个account

    `cd qdata/node1/keystore`

	`geth account new --keystore .`

	`重命名生成的以UTC开头的文件为key_file1`

	`keyfetcher key_file1 <这里输入你的生成账号时的password>`

	`在输出中就可以得到账号的raw private_key, nodeid`

	`echo private_key > ../geth/nodekey`
	
	`将nodeid存入static-nodes.json文件中去, 形式如下:`

	["enode://db59ca2bb693ea395e0f7e623e085f37465c85b77b7ff4835ab40b9c63b93177080f260b33636b7dae5212fa7d48cda69bf9f2be75e8b6163d17667b3bb8757a@127.0.0.1:21000?discport=0&raftport=50401",
	"enode://9294c4eeb74271f6a28ae2c8bccf6ba360bf52b6182bda9869e4f54251583a03254f8eab318a2effc3155d73391159a5525961893e5a6d8379df91f71158a0ec@127.0.0.1:21001?discport=0&raftport=50402",
	"enode://4ce71e5cc27b935ea60e12e9bc8d2545491e0c9a1759ea4f8f4aeabd361cd22167559cfe44310ecc94b328181dc85ac901b6abb8d5adc6e592b96c88e231745f@127.0.0.1:21002?discport=0&raftport=50403"]

	`copy static-nodes.json permissoned-nodes.json`

5. 重复3和4创建另外两个结点node2, node3
	
	`geth --datadir qdata/node1 init genesis.json`

	`geth --datadir qdata/node2 init genesis.json`

	`geth --datadir qdata/node3 init genesis.json`

#### 启动本地单结点 ####

1. 创建run_three_nodes.sh

>#!/bin/bash
>set -u
>set -e

>GLOBAL_ARGS="--raft --rpc --rpcaddr 0.0.0.0 --rpcapi >admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum --emitcheckpoints"
>nohup constellation-node tm1.conf 2>> qdata/logs/constellation1.log &
>sleep 2

>nohup constellation-node tm2.conf 2>> qdata/logs/constellation2.log &
sleep 2

>nohup constellation-node tm3.conf 2>> qdata/logs/constellation3.log &
sleep 2

>PRIVATE_CONFIG=tm1.conf nohup geth --datadir qdata/node1 $GLOBAL_ARGS --permissioned --raftport 50401 --rpcport 22000 --port 21000 --unlock 0 --password passwords.txt 2>>qdata/logs/1.log &

>PRIVATE_CONFIG=tm2.conf nohup geth --datadir qdata/node2 $GLOBAL_ARGS --permissioned --raftport 50402 --rpcport 22001 --port 21001 --unlock 0 --password passwords.txt 2>>qdata/logs/2.log &

>PRIVATE_CONFIG=tm3.conf nohup geth --datadir qdata/node3 $GLOBAL_ARGS --permissioned --raftport 50403 --rpcport 22002 --port 21002 --unlock 0 --password passwords.txt 2>>qdata/logs/3.log &

注:以上创建的3个账号使用的是相同的密码
2. 创建stop.sh

`#!/bin/bash

killall geth

killall constellation-node`

3. 运行run_three_nodes.sh

