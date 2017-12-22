# 零起点玩转基于以太坊的联盟链Quorum系列02

**注** : 上一节未完成了本地3结点网络的搭建工作, 基于raft共识模式. 本节主要讲解在3结点网络上的完成智能合约的部署与测试

## 智能合约

### 一个简单的智能合约实现

```
pragma solidity ^0.4.17;

contract SimpleStorage {
    uint public storedData;

    function SimpleStorage(uint initVal) public {
        storedData = initVal;
    }

    function set(uint x) public {
        storedData = x;
    }

    function get() public constant returns (uint retVal) {
        return storedData;
    }
}
```

### Truffle 部署智能合约
1. 新建一个Truffle工程

**注** : truffle 是目前以太坊开发的主要工具链之一, 由CONSENSYS公司开发.

```
mkdir test
cd test
truffle init
```
完成上述操作之后, 目录结构如下所示:

![文件结构](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/21.png)

2. 将上面的智能合约示例保存到test/contracts目录为SimpleContract.sol

3. 编译智能合约
```
truffle compile --all
```

执行之后会生成build目录, 内容如下:

![build目录结构](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/22.png)

4. 编写部署文件migrations/2_deploy_simplestorage.js
```
var simple_storage = artifacts.require("SimpleStorage");

module.exports = function(deployer) {
    deployer.deploy(simple_storage, 42, {privateFor:["3znvmFqqc63FyiW/bhxIlTHfAZ6xnuhMgPaxTchgJlg="]});
};
```
**注** : privateFor表明这是一个私有合约, 不是所有结点可见

5. 修改truffle.js文件
```
module.exports = {
    networks: {
        development: {
            host: "localhost",
            port: 22000,
            network_id: "*",
            gas: 4600000,
            gasPrice: 0
        },
        second_node: {
            host: "localhost",
            port: 22001,
            network_id: "*",
            gas: 4600000,
            gasPrice: 0
        },
        third_node: {
            host: "localhost",
            port: 22002,
            network_id: "*",
            gas: 4600000,
            gasPrice: 0
        }
    }
};
```
**注** : 端口号一定要与你本地实验网络的各结点的端口号对应起来

6. 部署智能合约到链上

```
truffle migrate --reset
```

输出如下所示:

![部署命令输出](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/23.png)

## 验证
### 私有合约只有允许的结点可以看到

```
var abi = 智能合约编译生成的abi
var private = eth.contract(abi).at(智能合约的地址)
```
将上述命令在三个结点上分别执行, 前面我们在node1上通过privateFor字段指定只有node3可以看到智能合约.所以通过private访问智能合约的时候, 也只有node1和node2
可以看到数据状态.执行效果如下:

![private执行效果1](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/25.png)

![private执行效果2](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/26.png)

![private执行效果3](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/27.png)

### 验证在私有合约的基础只有privateFor的结点可以收到
在node1上发布一个privateFor["node2", "node3"]的智能合约, 然后在node1中执行

```
private.set(199, {from:eth.coinbase, privateFor:["3znvmFqqc63FyiW/bhxIlTHfAZ6xnuhMgPaxTchgJlg="]})
```
上面的意思是这个交易只有node3可见, 虽然node2, node3都是可见到智能合约

在node3中执行private.get()结果如下:

![node3 private.get](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/28.png)

在node2中执行private.get()结果如下:
![node2 private.get](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/29.png)

**Next** : 下一篇将完成一个完整的基于这个本地测试网络的示例:积分联盟系统
