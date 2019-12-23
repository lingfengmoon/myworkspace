# Hadoop                       

```
大数据的特征：*****
       Volume:  巨大的数据量				
       Variety：数据类型多样化
       Velocity: 数据增长速度快
       Value: 价值密度低
       
谷歌三篇文章：*****
    - 2003年发表的《GFS》
	基于硬盘不够大、数据存储单份的安全隐患问题，提出的分布式文件系统用                                             于存储的理论思想。
    - 2004年发表的《MapReduce》
 	基于分布式文件系统的计算分析的编程框架模型。
 	
    - 2006年发表的《BigTable》
	针对于传统型关系数据库不适合存储非结构化数据的缺点，提出了另一种适										合存储大数据集的解决方案
	
Hadoop重要组成：******
        Hadoop Distributed File System(HDES)：分布式文件系统
        Hadoop YARN：作业调度和资源管理框架
        Hadoop MapReduce：基于YARN的大型数据集并行计算处理框架
	
Hadoop官网：
		 http://hadoop.apache.org/
	 版本：
         Apache Hadoop(社区版): 基础版本，入门学习最好
         Cloudera Hadoop(CDH版): 大型互联网企业中用的较多
         Hortonworks Hadoop(HDP): 文档较好
------------------------------------------------------------
hadoop目录介绍：
bin: Hadoop的脚本管理目录和使用目录,sbin目录下实现脚本的基础
etc: Hadoop配置文件所在目录  core-site.xml；
                          hdfs-site.xml； 
                          mapred-site.xml；
                          yran-site.xml
include: 对外提供的库文件(存储着一些C++的动态链接库,为了有C++程序访问                                                     HDFS使用)
lib: Hadoop环境需要使用的动态库和静态库存在位置
sbin: Hadoop集群启动和停止时需要执行脚本命令
share: hadoop各个模块编译后jar所在的位置
libexec:各个服对应的shell配置文件所在目录
```

## HDFS

```
HDFS伪分布配置格式化的目的：
	1. 生成一个集群唯一标识符
	2. 生成一个块池唯一标识符
	3. 生成namenode进程管理内容的存储路径：
	   默认配置文件属性hadoop.tmp.dir指定的路径下生成dfs/name目录
	4. 生成镜像文件fsimage，记录分布式文件系统根路径的元数据
完全分布式格式化集群的目的：
	- 产生新的clusterid
	- 产生块池id
	- 创建name的存储路径
	- 产生fsimage镜像文件
启动集群目的：
  	1 启动集群中的各个机器节点上的分布式文件系统的守护进程
          namenode 一个
          resourcemanger 一个
          secondnamenode 一个
          datanode 多个
          nodemanager 多个
	2 在namenode守护进程管理内容的目录下生成edit日志文件
	3 在每个datanode所在节点下生成${hadoop.tmp.dir}/dfs/data目录
```

### HDFS概述

**HDFS:分布式文件系统**；HDFS文件在物理介质中的存储方式是**分块存储**

```
HDFS:
	1 Hadoop的核心技术之一
	2 适合存储超大数据集的文件
	3 采用数据冗余的方式提供了数据的安全性，以空间换取安全性****
HDFS优点：*******
       1  高可靠性:Hadoop存储和处理数据的能力强
       2  高扩展性: 有效的分布数据计算,在不同节点上
       3  高效性:动态的移动数据,可以保证各个节点之间的数据平衡
       4  高容错: Hadoop能自动保存多个文件副本,副本丢失会自动恢复
       5  流式数据访问: 一次性写入，多次读取；保证数据一致性
       6  构建成本低：可以构建在廉价机器上
       
HDFS缺点:
        1.无法高效存储大量小文件(因为HDFS文件存储机制问题)
        2.不适合并发写入以及任意修改文件
        3.不适合低延迟数据访问
```

### HDFS体系结构***

```
namenode格式化之后会在/opt/apps/hadoop/tmp/dfs/name/current目录下产生fsimage文件和edits文件
fsimage文件: 
命名空间镜像文件，它是文件系统元数据的一个完整的永久检查点。记录最近检查点的文件系统的所有文件和目录的元数据.
          
edit文件：
编辑日志文件，HDFS系统对datanode的所有的增删改查操作即客户端datanode的操作记录。
------------------------------------------------------------
HDFS采用的是master/slaves这种主从的结构来管理数据，该结构模型主要由四个部分组成：Client,namenode,datanode,secondarynamenode
1 namenode:
    NameNode(基于内内存),不会和磁盘发生交互操作,只在内存中完成
	 1.1 中心服务器，管理HDFS的名称空间
	 1.2 在内存中维护数据块的映射信息
	 1.3 实施副本冗余策略
	 1.4 处理客户端的读写操作
	 
2 secondarynamenode: 帮助NameNode做磁盘操作      
	 2.1 帮助NameNode合并fsimage和edits文件（检查点机制），并将合并						  好的文件推还给NameNode做备份使用。
	 2.2 不能实时同步，不能作为热备份节点
	 合并具体步骤：
	 	secondnamenode请求namenode停止使用editlog文件
	 	secondnamenode通过http协议获取fsimage和edit
	 	secondnamenode读取fsimage到内存中，合并fsimage和edit
	 	将合并的fsimage_x.ckpt通过http发送给namenode
	 	namenode进行更名操作
	 
3 datanode:
	 2.1 存储真正的数据(块进行存储)
	 2.2 执行数据块的读写操作
	 2.3 心跳机制（3秒）
	 
4 client:
	 4.1 HDFS实际上提供了各种语言操作HDFS的接口。
	 4.2 与NameNode进行交互，获取文件的存储位置（读/写两种操作）
	 4.3 与DataNode进行交互，写入数据，或者读取数据
	 4.4 上传时分块进行存储，读取时分片进行读取
```

### HDFS工作机制

```
1 开机启动过程:
	1.1 namenode启动，加载fsimage内容到内存中，执行editlog文件的各  			项操作 保证内存中原数据是最新的。
	1.2 namenode生成一个新的fsimage和edit ，更新操作写入edit

2 安全模式:
	当集群启动的时候 ,先加载NameNode,NameNode节点内部会先加载镜像文件夹到内存中,并执行Edits日志,在进行加载和执行日志的时候,即刚刚启动时,NameNode是不会接受任何对当前集群操作的信息的,此时NameNode处于全模式状态。启动结束后，退出安全模式，进入正常运行状态。
	 	
3 心跳机制:*****
	master启动时会开启一个IPC服务，slave开启后会主动连接这个服务，每隔3s一次（时间可以调整），这种每隔一段时间连接一次的机制就称为心跳机制，
slave通过心跳向master汇报自己的信息，master通过心跳下达命令，master长时间（默认10分30秒）收不到slave的信息就认为slave荡机了。
  	Namenode通过心跳得知datanode状态。
  	Resourcemanager通过心跳得知nodemanager状态

4 检查点机制:
	secondarynamenopde辅助namenode将fsimage和edit合并的周期就称为检查点机制；间隔时间到3600s或edits文件大小到64都会进行合并
```

### HDFS-数据的读写流程***

```
读取流程：
  1 客户端通过FileSystem向NameNode请求下载文件，namenode通过查找元数据，找到文件所存储在datanode上的块信息并返回给客户端
  2 客户端会选择一个datanode进行读取数据（就近原则，然后是随机原则）
  3 datanode开始传输数据给客户端（从磁盘中读取数据输入流，以packet为单位进行验证传输）
  4 客户端以packet为单位接收数据，先在本地缓存（内存）然后写入目标文件
  ps:packet-->数据报包
-------------------------------------------------------------------------------- 
写流程：
  1  客户端通过FileSystem创建连接向Namenode请求上传文件，namenode会检查文件是否存在，父目录是否存在，并且响应客户端的上传请求
  2 客户端上传第一个block块信息，并请求上传到哪个datanode
  3 namenode会返回对应上传的datanode信息，并允许上传数据
  4 客户端通过FSDataOutputStream请求向datanode上传数据，datanode响应请求并建立通道传输数据，以packet为单位接收数据；
  5 客户端先从磁盘读取数据到本地缓冲区，然后以packet为单位上传到datanode中，datanode内部接收数据以后启用副本机制，在不同的datanode进行数据传入
  6 直到所有的数据上传完成之后，datanode会接收信息记录块信息并且通过心跳机制将数据信息上报给namenode.客户端也会通知namenode数据上传成功
```

### HDFS块的设计

```
切块存储的原因：
 	最小化寻址开销时间，节省内存使用率
 块大小为128 
```



### HDFS常用shell命令

```
运行一个文件系统命令：
	hdfs dfs
1 从本地系统复制文件到HDFS文件系统中
	hdfs dfs -put 操作系统中文件所存储的位置 HDFS文件系统路径
2 从hdfs文件系统中将文件复制到本地
    hdfs dfs -get HDFS文件系统路径 本地存储文件的路径
3 查看hdfs文件系统中的文件内容
	hdfs dfs -cat HDFS文件系统路径
4 查看HDFS文件系统下有哪些文件或文件夹
	hdfs dfs -ls HDFS文件系路径
5 HDFS系统中创建文件夹
	hdfs dfs -mkdir HDFS文件系路径
	# 递归创建文件夹
	hdfs dfs -mkdir -p HDFS文件系路径
6 HDFS文件系统中删除文件(文件夹)
    hdfs dfs -rm HDFS文件系统路径(文件夹)
--------------------------------------------------------
 1 格式化集群
 	hadoop namenode -format
 2 启动/关闭dfs脚本
 	start-dfs.sh
 	stop-dfs.sh
 3 启动/关闭yarn脚本
 	start-yarn.sh
 	stop-yarn.sh
 4 启动/关闭hdfs所有进程
 	start-all.sh
 	stop-all.sh
 5  刷新节点
    hadoop dfsadmin -refreshNodes
 6  启动单个datanode/namenode
 	hadoop-daemon.sh start datanode/namenode.
```

### HA（高可用）

```
HA出现的原因：可以解决 单点故障和性能瓶颈
HA基本原理：
	采用n台JN存储editlog每次数据写入操作有一半以上的JN完成就认为写入成功，数据不会丢失；但是此算法最多能容忍一半机器挂掉；
--------------------------------------------------------
active和standby交互原理：
  1 在HA架构里面SecondaryNameNode这个冷备角色已经不存在了，为了保持 standby NN时时的与主Active NN的元数据保持一致，他们之间交互通过一系列守护的轻量级进程JournalNode。
  2 任何修改操作在 Active NN上执行时，JN进程同时也会记录修改log到至少半数以上的JN中，这时Standby NN 监测到JN 里面的同步log发生变化了会读取 JN 里面的修改log，然后同步到自己的的目录镜像树里面
  3 当发生故障时，Active的 NN 挂掉后，Standby NN 会在它成为Active NN 前，读取所有的JN里面的修改日志，这样就能高可靠的保证与挂掉的NN的目录镜像树一致，然后无缝的接替它的职责，维护来自客户端请求，从而达到一个高可用的目的。
----------------------------------------------------
命令：
	  1  启动JN
	  hadoop-daemon.sh start journalnode 
	  2  同步日志到JN
	  hdfs namenode -initializeSharedEdits
	  3 拉取镜像文件
	  hdfs namenode -bootstrapStandby
```

### Zookeeper

**Zookeeper是一个分布式协调服务；采用主从集群模式，是Hadoop和Hbase的重要组件。主要用于解决分布式集群中应用系统的一致性问题。**

```

选举机制：
	集群中只要有半数以上的机器存活集群就可以正常服务，所以zookeeper集群必须是奇数Leader的选举过程是内部选举,因为Zookeeper没有主从模式在,所以每一个节点都认为自己是Leader,所以我们在部署Zookeeper的时候会给每台Zookeeper节点一个ID码,ID码就是决定投票使用；ID值越大选举时候的权值越重，同时满足半数以上的服务器投票支持就可以称为leader
-------------------------------------------------------
监听原理：
    1. 首先要有一个main()线程  
    2. 在main线程中创建Zookeeper客户端， 这时就会创建两个线程， 一个负责网络连接通信(connet),一个负责监听(listener)。  
    3. 通过connect线程将注册的监听事件发送给Zookeeper。  
    4. 在Zookeeper的注册监听器列表中将注册的监听事件添加到列表中。  
    5. Zookeeper监听到有数据或路径变化就会将这个消息发送给listener 
    6. listener线程内部调用了process（） 方法。
--------------------------------------------------------
zookeeper特点：
	1 zookeeper是一个分布式集群，一个leader多个follower
	2 集群中只要有半数以上的节点存活，集群就能正常服务
	3 全局一致性：每个server保存一份相同的数据副本，client无论连接到哪个server，数据都是一致的
	4. 更新请求按顺序进行：来自同一个client的更新请求按其发送顺序依次执行；
	5. 数据更新的原子性：一次数据的更新要么成功，要么失败
	6. 数据的实时性：在一定时间范围内，client能读到最新数据。
----------------------------------------------------------
zookeeper数据模型：
   zookeeper数据模型采用与unix系统类似的树状结构；zookeeper统一使用节点的概念称之为znode,znode既可以作为保存数据的容器，也可以作为保存其他znode的容器；所有的znode构成了一个层次化的命名空间
-------------------------------------------------------------
zookeeper的应用场景：
            1. 统一配置管理
            2. 统一集群管理
            3. 服务器节点动态上下线感知
            4. 软负载均衡等
            5. 分布式锁
            6. 分布式队列
zookeeper的节点类型：
	临时；临时有序；持久；持久有序
zookeeper的节点状态：
	ephemeral(短暂)；持久
--------------------------------------------------
命令：
	1 开启zookeeper （k小写）
		zkServer.sh start
	3 关闭zookeeper
		zkServer.sh stop
	3 使用客户端连接zookeeper
		zkCli.sh -server [host]
	4 客户端退出服务
       [zk: localhost:2181(CONNECTED) 1] quit/close

```

## YARN

**yarn是一个分布式资源管理框架，主要负责mapreduce资源管理分配**

### YARN的基本组成

```
YARN的基本组成：
	ResourceManager、NodeManager、ApplicationMaste，Container
	
1 ResourceManger :
	全局资源管理器，负责整个系统的资源管理和分配，包括处理客户端的请求启动/监控APP master、监控nodemanager、资源的分配与调度。它主要由两个组件构成：调度器（Scheduler）和应用程序管理器（Applications Manager，ASM）
	1.1  调度器（Scheduler）：
		调度器根据容量、队列等限制条件将系统中的资源分配给各个正在运行的应用程序，调度器仅根据各个应用程序的资源需求进行资源分配，限定每个任务使用的资源量。
	1.2  应用程序管理器
		应用程序管理器负责管理整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动AppMaster、监控AppMaster运行状态并在失败时重新启动它等
		
2 applicationMaster :
	管理yarn内运行的应用程序的每个实例；负责 数据的切分，为应用程序申请资源并进一步分配给内部任务。，任务监控与容错，协调来自resourcemanager的资源，并通过nodemanager监视任务的执行和资源使用情况。
    
3 nodeManger:
	负责每个节点上的资源和使用，处理来自于resourcemanager和app master的命令，管理着抽象容器(抽象容器代表着一些特定程序使用针对每个节点的资源),定时地向RM汇报本节点上的资源使用情况和各Container的运行状态.

4 container : 
  yarn为每个任务分配的资源容器，里面包含该任务可以使用的资源，AM向RM申请资源时，RM为AM返回的资源就是用container表示的
-----------------------------------------------------------
核心思想:
	yarn的基本思想是将资源管理和作业调度/监视功能划分为单独的守护进程。其思想是拥有一个全局ResourceManager（RM)和每个应用程序ApplicationMaster (AM)。应用程序可以是单个作业，也可以是一组作业.
ResourceManager和NodeManager是YARN的两个长期运行的守护进程，提供核心服务
------------------------------------------------------------
YARN的三种调度器:
	 1 FIFO调度器:一个队列，按照到达的先后时间来提供服务 	
     2 容量调度器:
     	支持多个任务队列，每个队列都可配置一定的资源容量，使用容量调度器时，一个独立的专门队列保证小作业一提交就可以启动。
	 3  公平调度器:
	 	支持多队列多用户，每个队列中的资源可以分配,同一个队列中的作用是公平共享队列中所有的资源,每个队列中job会按照优先级来分配资源,每个job分配的资源一定是平均的保证公平。
------------------------------------------------------------
YARN启动关闭命令:
     yarn-daemon.sh start ResourceManager
     yarn-daemon.sh start NodeManager
     start-yarn.sh
     stop-yarn.sh
```

### YARN的完整执行流程

```
1 作业提交
 1.1：Client调用job.waitForCompletion方法向集群提MapReduce作业。
 1.2：Client向RM申请一个作业id。
 1.3：RM给Client返回该job资源的提交路径和作业id。
 1.4：Client提交jar包、切片信息和配置文件到指定的资源提交路径。
 1.5：Client提交完资源后，向RM申请运行MrAppMaster。
2 作业初始化
      当RM收到Client的请求后，将该job添加到容量调度器中。某一个空闲的NM领取到该Job，该NM创建
      Container，并产生MRAppmaster。下载Client提交的资源到本地。
3 任务分配
 3.1 ：MRAppMaster向RM申请运行多个MapTask任务资源。
 3.2 ：RM将运行MapTask任务分配给另外两个NodeManager，另两个
       NodeManager分别领取任务并创建容器。
4 任务运行
 4.1 ：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，
 		MapTask对数据分区排序。
 4.2 ：MRAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask.
 4.3 ：ReduceTask向MapTask获取相应分区的数据。	
 4.4 : 程序运行完毕后，MR会向RM申请注销自己。
5 进度和状态更新
    YARN中的任务将其进度和状态返回给应用管理器, 客户端每秒(通过
mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。
6 作业完成
	除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。
```

## MapReduce

**MapReduce是一个分布式运算程序编程框架；**

*在unix上运行mr程序命令：*
hadoop jar jar包路径 类的全限定名 ‘文件输出路径’

```java
优势：处理大规模数据集

MapReduce是什么？
	mapreduce是谷歌提出的分布式并行编程模型mapreduce论文的开源实现，以可靠容错的方式运行在分布式文件系统HDFS上的并行处理数据的编程模型
	MR核心功能：将用户编写业务逻辑代码和自带的默认组件组合成一个完成的分布式运行程序,并运行在一个Hadoop集群上。
	MR的核心思想:  分而治之、移动计算不移动数据
	MR的设计理念是："计算向数据靠拢" 在进行数据处理时MapReduce框架会将Map程序就近的在HDFS数据所在的节点上运行，即将计算节点和存储节点放在一起运行，从而减少节点间的数据移动开销。正因为这样，MapReduce可以并行的进行处理数据，解决计算效率问题
    mapreduce处理数据，首先通过读取文件，进行分片，然后所要处理数据的行偏移量和内容作为<k1 v1 >输入到map函数中，map对数据进行处理会输出<K2 V2 >然后就是shuffer的过程，也就是排序归并，k2 和v2的集合会作为新的<K3,V3>输入到reduce，数据经过reduce端的处理最终筛选出我们想要的数据；
----------------------------------------------
MR优点：
	  1 易于编程
	  2 良好的扩展性
	  3 高容错性
	  4 适合做离线数据处理
MR缺点：
	 不适合做实时计算，不适合做流式计算
--------------------------------------------
分片与分块区别：
1. 分片是逻辑数据，记录的是要处理的物理块的信息而已
2. 块是物理的，是真实存储在文件系统上的原始数据文件
------------------------------------------------------------
  StringTokenizer可以对数据进行拆分 默认是 空格，\t,\n,\r,\f
  拆分完之后得到的是一个类似于迭代器一样的对象
    StringTokenizer st = new StringToken(line);
     while(st.hasMoreTokens()){
         st.nextToken()
     }
   
```

### MR处理数据完整流程**

```
MapReduce处理数据流程：
	map---> shuffer--->reduce
MapTask:
    1.Input Split 对数据块进行分片分配给Map
    
    2.Map过程进行处理,Mapper任务会接收输入的分片,然后不断调用map方法,对记录进行处理,处理完毕之后,转换为新的<key,value>输出(其中每一个分片对应一个map,一个map可以被调用多次来处理该分片)
    
    3.Map的输出结果会缓存到内存里,当内存中buffer in memory当达到阈值(默认的80%),就会把记录溢写到次磁盘文件中,然后用剩余的20%来继续接收数据.    
    ps:优化map时可以调大buffer的阈值,缓存更多数据 
    特殊情况：当达到100%,此时缓冲就不会接受任何数据,直到缓冲区清空才会     开始接受数据
    
    4.内存中进行Partition(分区),默认是HashPartition
	ps:此hash采用 hash(key.hashcode() & Integer.MAX_VALUE) % numReduceTasks有多少个分区就会有多少个reducetask；目的是将map的结果分给不同的reducer,有几个分区就有几个reducetask,
	ps:partition的数据可以在job启动的使用通过参数设置,可有让map的通过均匀的分配给不同的机器上的reducer,保证负载均衡
	ps:有几个文件就有几个分区
	
	5.内存中partition结束之后,对于不同分区的数据,会按照key进行排序,这里key必须实现WritableComparable接口,该口实现了Comparable,因此才可以进行比较排序
	
	6.对于排序之后的<key.value>会按照key进行分组,如果key相同,就会被分到一个组中,最终每个分组会调用一次reduced方法
	
    7.排序分组结束之后,相同的key就在一起组成了一个类表,如果设置Combiner,就合并数据,减少写入磁盘
   ps:combiner本质即使一个reducer
   
   8.当磁盘中的spill文件数目比规定的文件数据多的时候,会多次调用  combiner,在不影响结果的情况下,combiner可以被调用多次
   
   9.Map结束的时候会把spill出来的多个文件合并成一个,合并（merge）过程最多10个文件同时合并成一个文件,多余的文件就多次调用合并
   10.Map端的shuffle完毕,数据都有序的存放到磁盘中,等待reducer来获取
   
ReduceTask:
    1.reducer的后台会被AppMaster指定的机器上将map的output拷贝到本地,先拷贝到内存,内存满了就拷贝磁盘
	2.Reducer采用Merge sort将来自各个map的数据进行merge,merge成一个有序的更大的文件
	3.reduce端job开始,输入是shuffle sort过程所产生的文件
	4.reducer的输入文件,不断的merge后,他们会出现最终文件,这个最终文件可能存储在磁盘也可能在内存.
reduce方法就被自动执行当前这个文件的处理最终该处理结果放到HDFS上
-----------------------------------------------------------------------------------------
  先对数据进行切片,然后将数据传递给map,map的输出是圆形缓冲区,圆形缓冲区默认大小是100M,当 达到80%即0.8的时候将数据溢写到本地,剩余20%用于继续获取数据,在溢写到磁盘的时候会执行partition(分区)和 sort(排序),然后对文件进行合并操作,合并完成之后 reducetask会启动线程去mapTask拉取数据,然后进行文件合并, 并进行排序(归并),然后将小文件合并成一个大文件并调用reduce方法完成最终输出效果
```

### shuffle

```
shuffle: 从map端输出圆形缓冲区开始到reduce端输出之前


```

### MR编程

```java
Combiner:
/**1 combiner是在maptask的节点上运行，每一个map都可能产生大量的本地文件输出combiner就是对map端输出的结果先进行一次合并，可以减少map和reduce节点之间的数据传输量；
2 combiner的存在是为了提高网络IO传输性能也是mapreduce的一种优化手段
3 combiner就是reduce方法的实现，只不过是在map端
*/
自定义combiner类:
	1 必须继承于reducer类，实现类中reduce()方法;
	2 job中设置combiner的加载 --> 
				job.setCombinerClass(自定义combiner类.class)
------------------------------------------------------------
 Partitioner(分区)
 /**Partitioner的作用:
	将mapper 输出的key/value划分成不同的partition,每个reducer对应一个partition; 可以使用自定义Partitioner来达到reducer的负载均衡，提高效率的目的。分区是从0分区开始 逐渐向后
 */       
//1 自定义分区类继承于partitioner
 	重写getPartition指定分区规则
 	
//2  在Driver 类中设置自定义分区类型和分区个数
	job.setPartitionerClass(myPartition.class);
	job.setNumReduceTasks(分区个数);
------------------------------------------------------------
自定义分组：
/**自定义分组类要继承 writableComparartor类 并重写compare方法指定分组规则
*/
public class myGroup extends WritableComparator{
		public myGroup() {	
		}
		//指定分组规则，uid相等的认为是同一组
		@Override
		public int compare(WritableComparable a, WritableComparable b) {
			movieCustomType a1 = (movieCustomType) a;
			movieCustomType b1 = (movieCustomType) b;
		return  a1.getUid() - b1.getUid();
		}
}
------------------------------------------------------------
自定义数据类型：
/**   非排序 writable
继承于writable  重写readFile(); write();
readFile读取字段的顺序要和write的顺序一样
*/
	//序列化，将对象的字段信息写入输入流
	@Override
	public void write(DataOutput out) throws IOException {
		out.writeLong(upPacknum);
		out.writeLong(downPacknum);
	}
	//反序列化,从输入流读取字段的信息
	@Override
	public void readFields(DataInput in) throws IOException{
		this.upPacknum = in.readLong();
		this.downPacknum = in.readLong();
	}
-----------------------------------------------------------
/** 排序  writableCompare 排序的属性必须作为map的 k2
继承于 writableCompare 重写 readFile(); write(); compareTo();
*/
@Override
	public int compareTo(mobileDataSort o) {
		int res = (int) (o.upPacknum - this.upPacknum);
		if(res ==0) {//二次排序 packnum相同就是按照 payload排序
			return (int)(o.uppayload - this.downpayload);
		}else {
			return res;
		}
	}
```

## Hive

### Hive定义

```
hive是一个构建在hadoop上的数据仓库工具，可以将结构化的数据文件映射成一张数据表,并且可以使用类sql语言对数据进读写以及管理，这个类sql语言叫做Hql
--------------------------------------------
hive优点：
	1 操作接口类似于sql语言，提高了上手容易度
	2 避免写复杂的MR程序，减少开发人员学习成本
	3 适合做离线分析处理，延展性好，高容错。
	
hive缺点：
	1 Hive的hql表达能力有限
    2 执行效率低
    3 调优较难
------------------------------
启动matestore服务：
				hive --service metastore &;
				
自定义函数分类：
	UDF: 用户自定义函数 一对一的输入输出；
	UDAF: 用户自定义聚合函数，多对一输入输出 
	UDTF: 用户自定义表生产函数，一对多输入输出
	
常用的Sered： csv,tsv,json,regexp

内部表的表目录会被删除，但是外部表的表目录不会被删除 location
```

### Hive架构

```
1 用户连接接口(CLI;JDBC;WebUI)
2 元数据 ：hive会将元数据保存在相应得元数据库中，derby是自带的数据库
3 驱动器Driver: 解析器， 编译器，优化器，执行器，完成hql语句的语法查	询，分析，编译，优化及查询生成的计划；	
4 Hdfs : hive的数据存储在HDFS中 
```

### HIve工作原理

**建库的本质就是在${hive.metastore.warehouse.dir}对应的目录下，创建一个新的目录，建表就是在该目录下的库下建立一个目录**

```
1 用户提交查询任务给Driver 
2 驱动器将hql发送给编译器，编译器（compiler）根据用户任务去MetaStore	中获取需要的Hive元数据；
3 compiler得到元数据对任务进行编译，也就是生成mr编程程序，然后优化器会对其进行优化
4 编译解析完成 将最终的计划提交给Driver
5 Driver将plan转交执行器，执行器将Plan发送到jobTackre上,然后分配作业到TaskTracker上。
6 执行作业的过程就是一个mapreduce工作过程，执行引擎接收来自数据节点上的处理过的数据
7 执行引擎发送结果给驱动程序，驱动发给Hive接口
```

### 本地模式和远程模式的区别

```
本地模式：
	特点是：hive服务和metastore服务运行在同一个进程中，mysql是单独的进程，可以在同一台机器上，也可以在远程机器上。该模式只需将hive-site.xml中的ConnectionURL指向mysql，并配置好驱动名、数据库连接账号即可

远程模式：
	特点是：hive服务和metastore在不同的进程内，可能是不同的机器。该模式需要将hive.metastore.local设置为false，并将hive.metastore.uris设置为metastore服务器URI，如有多个metastore服务器，URI之间用逗号分隔
```



## Hbase

**hbase是一种分布式，面向列的，可扩展，支持海量数据存储，运行在HDFS上的非关系型数据库**

### Hbase中架构

```
1 Client : hbase客户端，包含hbase接口，维护缓存来加速访问hbase的速度（也就是维护了一个缓存区，存储查询过的Region的信息）

2 zookeeper:
	2.1 监控Hmaster的状态，保证有且仅有一个活跃的hmaster
	2.2 监控RegionServer的状态，感知HRegionServer的上下线信息，并实时通知给Hmaster。
	2.3 zk是存储元数据的统一入口地址
	2.4 存储hbase的部分元数据

3 Hmaster:
	3.1 维护整个集群的负载均衡
	3.2 维护集群的元数据信息
	3.3 为RegionServer分配Region
	3.4 发现失效的Region,并将失效的Region分配到正常的RegionServer上	
	
4 HRegionServer:
	4.1 维护Hmaster分配给的Region
	4.2 负责和底层HDFS交互，存储数据到HDFS
	4.3 负责Region变大以后的拆分

5  HRegion: hbase中分布式存储和负载均衡的最小单元，表或表的一部分
   Store: 相当于一个列族，对应一个memstore
   Memstore ：内存缓冲区，用于将数据批量刷新到hdfs中，默认大小为128M
   			在内存中对存储的数据进行排序，排序规则：先按照rowkey进行排序，然后在按照列族进行字典排序，在             按照K进行排序；
   Hlog : 编辑日志文件，记录客户端的操作
   HStoreFile : 和HFile概念一样，不过是一个逻辑概念。HBase中的数据是以HFile存储在Hdfs上。

6 HDFS:
	提供元数据和表数据的底层分布式存储服务
	数据多副本，保证高可靠和高可用性
	  
```

### Hbase的寻址步骤（二级跳转）

```
1 通过查找zookeeper上的/hbase/mete-region-sever节点来查询哪个Regionserver上存储着hbase:mete,

2 访问含有hbase:meta表所在的Reionserve,通过mete表获得需要查询的行键在哪个Rgeion,以及该Region在哪个RegionServer.

3 连接具体的Regionsever,找到Region开始获取数据

4客户端第一次访问mete会两层跳转寻址，然后把mete信息缓存起来，以后再次查询就 不需要进行两层跳转了
```

### Hbase的读写流程

```
写流程：
	
1 Client通过zookeeper调度，向RegionServer发出写数据请求.

2 数据先在RegionServer中的Hlog文件缓存，然后被写入Region中的Memstore,当memstore达到预设的阈值就会将数据刷新到一个Storefile

3 随着storefile文件的不断增多，其数量增长到一定阈值后，会触发compact合并造作，将多个storefile合并成一个大的Storefile,同时进行版本合并和数据更新。
4  StoreFiles通过不断的Compact合并操作，逐步形成越来越大的StoreFile

5 单个storefile大小超过一定阈值后，会触发split切分操作，将当前Region切分成两个新的Region,旧的Region会下线，新的两个Region会被Hmaster分配到相应的RegionServer上，以此实现负载均衡的目的
--------------------------------------------------
读流程

1 Clent通过访问 zookeeper查询到mete表的存放地址

2 通过mete表获取目标数据的Region信息，找到存储该Region的RegionServer,最后通过访问RegionServer获取到需要查找的数据

3 RgeionServer的内存分为Memstore和BlockCache两个部分MemStore主要用于写数据，BlockCache主要用于读数据。如果你开启了BlockCache,读请求先到BlockCache中查数据，查不到就到Memstore中查，再查不到就会到StoreFile上读，并把读的结果放入BlockCache。
```



### Hbase shell命令

```mysql
0	启动hbase
	start-hbase.sh start
	进入Hbase客户端
	habse shell
	
1 查询所有命名空间
	list_namespace
2 查找指定命名空间的表
	list_namespace_tables '命名空间名'  
3 创建指定命名空间
	create_namespace '命名空间名'
4 查询命名空间的详细信息
	describe_namespace '命名空间名'
5 修改命名空间信息
	5.1 添加命名空间的描述信息
 	alter_namespace '命名空间名', {METHOD => 'set', 'name' => 'michael'}
 	5.2 删除描述信息
	alter_namespace '命名空间名', {METHOD => 'unset', NAME => 'name'}
6 删除指定命名空间
	drop_namespace '命名空间名'
----------------------------------------------------------------------------------------
1 创建表（必须至少指定一个列族名）
	create '命名空间名:表名','列族名', '列族名'
	1.1 创建表同时指定属性
    create '命名空间名:表名',   
    {NAME=>'列族名',VERSIONS=>3,
    TTL=>2592000,BLOCKCACHE=>TRUE}
    
2 查看表属性信息
	describe '命名空间名:表名'
3 列出所有表
	 list
4 修改表属性
	alter '命名空间名:表名',   
    {NAME=>'列族名',VERSIONS=>3,
    TTL=>2592000,BLOCKCACHE=>TRUE}
 ps: version,TTL,BLOCKCACHE都可以修改

5添加列族（可以同时添加多个列族）
	  alter '命名空间名:表名','列族名'，'列族名'
6 删除列族
	  alter '命名空间名:表名','delet'=> '列族名'
6 删除表 （先禁用，在删除）
	   disable '命名空间名:表名'
	   drop '命名空间名:表名'
-----------------------------------------------------------------------------------------
DML操作
1 插入数据
	  put '命名空间名:表名','行键','列族:列名','列值',[时间戳]
2 scan查询数据
	  scan '命名空间名:表名' 
	  2.1 指定列查询
	  	 scan '命名空间名:表名',{COLUMNS=>'列族:列名'}
	  2.2 指定范围查询
	  
3 删除数据：
 	 3.1 删除指定行
 	 delete '空间名:表名'	 
 	 
	 3.2 删除某行内的指定key-value对
	 delete '空间名:表名'，'rowkey','列族:K'
	 
	 3.3 删除指定版本号单元格
	 delete '空间名:表名'，'rowkey','列族:K',version
	  
4 判断表是否存储
	exists 'namespace:tablename'
5 禁用/启用 表
	disable/enable 'namespace:tablename'
6 统计 表行数
	count 'namespace:tablename'
7 清空表数据
	truncat 'namespace:tablename'
```

## Sqoop





























