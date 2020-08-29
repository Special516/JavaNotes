## 一、Zookeeper 入门

Zookeeper 从设计模式的角度来理解：是一个基于观察者模式的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper 就将负责通知已经在 Zookeeper 上注册的那些观察者做出相应的反应。

#### 1、Zookeeper 的特点：

- 一个领导者（Leader），多个跟随者（Follower）组成的集群
- 集群中只要有半数以上的节点存活，Zookeeper 集群就能正常服务
- 全局数据一致，每个 Server 保存一份相同的数据副本，Client 无论连接到哪个 Server ，数据都是一致的
- 更新请求顺序进行，来自同一个 Client 的更新请求按其发送顺序依次执行
- 数据更新的原子性，一次数据更新要么成功，要么失败
- 实时性，在一定时间范围内，Client 能读到最新数据

#### 2、Zookeeper 的数据结构：

Zookeeper 数据模型的结构与 Unix 文件系统很相似，整体上可以看做是一棵树，每个节点称作一个 ZNode。每个 ZNode 默认能够存储 1MB 的数据，每个 ZNode 都可以通过其路径唯一标识。

#### 3、应用场景：

- **统一命名服务：**

    在分布式环境下，经常需要对应用/服务进行统一命名，便于识别

- **统一配置管理：** 

    分布式环境下，一般要求一个集群中所有节点的配置信息是一致的；对配置文件修改后，希望能够快速同步到各个节点上。

    配置管理可交由 Zookeeper 实现；可将配置信息写入 Zookeeper 上的一个 ZNode，各客户端服务器监听这个 ZNode，一旦 Zookeeper 中的数据修改，Zookeeper 将通知各个客户端服务器。

- **统一集群管理：** 

    Zookeeper 可以实现实时监控节点的状态变化，可将节点信息写入 Zookeeper 上的一个 ZNode ，监听这个 ZNode 可获取它的实时状态变化。

- **服务器动态上下线：** 

    客户端能实时洞察到服务器上下线的变化

- **软负载均衡：** 

    在 Zookeeper 中记录每台服务器的访问数，让访问数量最少的服务器去处理最新的客户端请求

## 二、Zookeeper 安装

#### 1、本地模式安装（测试）：

安装 JDK 并拷贝 zookeeper 安装包，然后解压到指定目录。

#### 2、修改配置文件：

- 将 /usr/local/zookeeper/zookeeper-3.4.10/conf 路径下的 zoo_sample.cfg 修改为 zoo.cfg：

    ```shell
    mv zoo_sample.cfg zoo.cfg
    ```

- 打开 zoo.cfg 文件，修改 dataDir 路径：

    ```shell
    dataDir=/usr/local/zookeeper/zookeeper-3.4.10/data
    ```

#### 3、启动 zookeeper 服务：

```shell
[root@localhost bin]# ./zkServer.sh start
```

#### 4、启动 zookeeper 客户端：

```shell
[root@localhost bin]# ./zkCli.sh
```

#### 5、退出客户端：

```shell
[zk: localhost:2181(CONNECTED) 1] quit
```

#### 6、退出服务：

```shell
[root@localhost bin]# ./zkServer.sh stop
```

#### 7、zoo.cfg 文件中配置参数的含义：

```shell
# 通信心跳数，Zookeeper服务器与客户端心跳时间，单位毫秒，它用于心跳机制，并且设置最小的session超时时间为两倍心
# 跳时间 2*tickTime
tickTime=2000

# LF初始通信时限。集群中的Follower跟随者与Leader领导者之间初始连接时容忍的最多心跳数（tickTime的数量）
# 用来限定集群中的Zookeeper服务器连接到Leader的时限
initLimit=10

# LF同步通信时限。集群中Leader与Follower之间的最大响应时间单位，假如响应时间超过 syncLimit*tickTime，
# Leader认为Follower死掉，从服务器列表中删除Follower
syncLimit=5

# 数据文件目录 + 数据持久化路径，主要用于保存 Zookeeper 中的数据
dataDir=/usr/local/zookeeper/zookeeper-3.4.10/data

# 客户端连接端口
clientPort=2181
```

## 三、Zookeeper 的内部原理

#### 1、选举机制（重点）：

**半数机制：** 集群中半数以上机器存活，集群可用。所以 Zookeeper 适合安装奇数台服务器。

Zookeeper 虽然没有在配置文件中指定 Master 和 Slave。但是，Zookeeper 工作时是有一个节点为 Leader，其它则为 Follower，Leader 是通过内部的选举机制临时产生的。

#### 2、节点的类型：

- **持久（Persistent）：** 客户端和服务器断开连接后，创建的节点不删除
- **短暂（Ephemeral）：** 客户端和服务器断开连接后，创建的节点自己删除

## 四、分布式安装部署

