# 零起点玩转基于以太坊的联盟链Quorum系列之Quorum docker镜像自动构建系统

## 构建docker镜像的意义
  - 对于开发过程来说, docker镜像使用所有的开者都能够快速启动一个统一的环境, 加速了开发环境搭建的速度.
  同时环境的一致性也可以避免很多因为环境问题而产生的bug, 方便测试与开发的快速迭代, 并最终大大降低成员沟通的成本.
  - 对于最终的交付, docker镜像可以将目标系统的运行环境与资源需求作最清晰的隔离与划分, 方便交付产品, 以及后期产品维护与更新.
  
## 文件结构与关系
![文件组成](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/quorum_docker_build_system.png)

- init.sh是系统的初始化脚本, 用于生成quorum结点工作需要的目标结构与配置文件数据, 运行之后生成的所有配置文件都位于Target目录中
- dockerfile是docker镜像的构建文件, 它以ubuntu官方标准镜像为基础, 生成quorum:v1容器镜像, 镜像里面只有quorum的二进制可执行文件
- docker-compose.yml是容器的启动配置文件, 它负责将Target目录中的配置文件加载到容器中, 并启动quorum单结点

## 具体配置目标结构
![文件组成](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/target_dir_structure.png)

- genesis.json是创世区块的配置文件
- keys目录存放的是结点共识组件的加密码公私钥
- static-nodes.json存放的是用于连接的结点url
- tm1.conf是共识组件的配置文件

## 启动服务脚本start.sh
![文件组成](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/start_geth.png)

**注意:** 这里面有一个非常关键的地方, 就是在docker容器中启动服务的时候, 要保证最后一个启动的服务是持有tty的, 这样容器才能够一直处于服务的状态

启动容器docker-compose up, 如下所示:
![文件组成](https://github.com/Lispre/Learn_Quorum_Blockchain_From_Scratch/blob/master/docker_conatiner_startup.png)
