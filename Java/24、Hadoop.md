## 一、大数据概论

#### 1、大数据概念：

大数据（Big Data）：指无法在一定时间范围内用常规软件工具进行捕捉、管理和处理的数据集合，是需要新处理模式才能具有更强的决策力、洞察发现力和流程优化能力的海量、高增长率和多样化的信息资产。主要解决，海量数据的存储和海量数据的分析计算问题。

#### 2、大数据特点（4V）：

- **Volume（大量）** 
- **Velocity（高速）** 
- **Variety（多样）** 
- **Value（低价值密度）** 

## 二、Hadoop 框架

#### 1、什么是 Hadoop：

Hadoop 是由 Apache 基金会所开发的分布式系统基础架构，主要解决海量数据的存储和海量数据的分析计算问题。

#### 2、Hadoop 的优势：

- 高可靠性：Hadoop底层维护多个数据副本，所以即使Hadoop某个计算元素或存储出现故障，也不会导致数据的丢失。
- 高扩展性：在集群间分配任务数据，可方便的扩展数以千计的节点。
- 高效性：在MapReduce的思想下，Hadoop是并行工作的，以加快任务处理速度。
- 高容错性：能够自动将失败的任务重新分配。

#### 3、Hadoop1.x 和 Hadoop2.x 的区别：

> Hadoop2.x的组成：MapReduce（计算）、Yarn（资源调度）、HDFS（数据存储）、Common（辅助工具）

在Hadoop1.x时代，Hadoop中的MapReduce同时处理业务逻辑运算和资源的调度，耦合性较大，在Hadoop2.x时代，增加了Yarn。Yarn只负责资源的调度，MapReduce只负责运算

#### 4、HDFS 架构概述：

- **NameNode：** 存储文件的元数据，如文件名、文件目录结构、文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的 DataNode 等。
- **DataNode：** 在本地文件系统存储文件块数据、以及块数据的校验和。
- **Secondary NameNode：** 用来监控 HDFS 状态的辅助后台程序，没隔一段时间获取 HDFS 元数据的快照。

#### 5、YARN 架构：

- **ResourceManager（RM）：** 处理客户端请求、监控NodeManager、启动或监控ApplicationMaster、资源的分配与调度
- **NodeManager（NM）：** 管理单个节点上的资源、处理来自 ResourceManager 的命令、处理来自 ApplicationMaster 的命令。
- **ApplicationMaster（AM）：** 负责数据的切分、为应用程序申请资源并分配给内部的任务、任务的监控与容错
- **Container：** Container是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等

#### 6、MapReduce 架构：

MapReduce将计算过程分为两个阶段：Map和Reduce

- **Map：** Map阶段并行处理输入数据
- **Reduce：** Reduce阶段对Map结果进行汇总

#### 7、大数据技术生态体系：

![](G:\云汇科技\笔记\浙思云汇\assets\大数据.png)

## 三、Hadoop 运行环境搭建

#### 1、环境准备：

- 克隆虚拟机

- 修改虚拟机的静态IP

    ```shell
    vi /etc/udev/rules.d/70-persistent-net.rules
    # 删除 eth0 网卡配置，修改 eth1 网卡为 eth0 网卡，并复制其 MAC 地址
    
    vi /etc/sysconfig/network-scripts/ifcfg-eth0
    # 修改网卡的 （HWADDR）MAC 地址，并修改 IP 地址
    IPADDR=
    GATEWAY=
    DNS1=
    
    vi /etc/sysconfig/network
    # 修改主机名称
    ```

- 在 /opt 目录下创建 module 目录和 software 目录

    ```shell
    [root@hadoop101 opt]# mkdir module
    [root@hadoop101 opt]# mkdir software
    ```

- 卸载现有JDK

    ```shell
    # 查询是否安装Java软件
    rpm -qa | grep java
    
    # 如果安装的版本低于1.7，卸载该JDK
    rpm -e 软件包
    
    #查看JDK安装路径
    which java
    ```

- 解压 jdk 到 module 目录

    ```shell
    tar -zxvf jdk1.8.0_191 -C /opt/module
    ```

- 配置环境变量

    ```shell
    vi /etc/profile
    
    ## JAVA_HOME
    export JAVA_HOME=/opt/module/jdk1.8.0_191
    export PATH=$JAVA_HOME/bin:$PATH
    export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ```

#### 2、Hadoop安装

- 解压 Hadoop 到指定目录

    ```shell
    tar -zxvf hadoop-2.7.2.tar.gz -C /opt/module/
    ```

- 配置环境变量

    ```shell
    ## HADOOP_HOME
    export HADOOP_HOME=/opt/module/hadoop-2.7.2
    export PATH=$PATH:$HADOOP_HOME/bin
    export PATH=$PATH:$HADOOP_HOME/sbin
    ```

- Hadoop 的目录结构

    - bin： hadoop、dhfs、yarn等命令
    - etc： hadoop 的配置信息
    - lib： 本地库
    - sbin： hadoop 启动/停止、集群启动/停止等命令

## 四、Hadoop 运行模式

