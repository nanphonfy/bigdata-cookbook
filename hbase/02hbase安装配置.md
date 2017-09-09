修改一台机器配置后，要同步(scp或rsync)到集群所有节点，重启hbase使配置生效。hbase依赖的中间件、系统服务、配置。
## 1. 条件
- jdk
>64位JDK才能管理超过4GB的内存。

- ssh服务
>集群模式下，hbase启动、关闭依赖ssh（查看状态：service sshd status），ssh管理所有节点的守护进程。

- 域名系统DNS
>hbase通过host name或domain name获取IP，得保证正反向DNS解析(先查本地/etc/hosts,自己配置而不使用域名解析服务，解析IP更快)正常。多网卡，可通过hbase.regionserver.dns.interface指定主网卡,默认值default。

- 本地环回地址(Loopback IP)
>配置为127.0.0.1 localhost

- 网络时间协议(NTP)
>集群节点间时间要基本一致，默认可容忍30s以内(hbase.master.maxclockskew)。偏差较多，集群会产生一些奇怪行为。

- 资源限制命令（ulimit和nproc）
>Linux的ulimit(同时打开文件数的限制)默认值1024，对于hbase而言太小了，eg.每个region3列族，每列族平均3个hfile，每个regionserver有100个region，约900个文件，频繁被客户端调用，涉及大量磁盘操作。  
noproc（限制打开进程数），若不设置可能异常：“java.lang.OutOfMemoryError: unable to create new native thread”。
对HDFS和mapreduce也至关重要，启动hadoop前就该设置好。

- 版本
>hbase依赖于hadoop，为避免版本不匹配，将分布式hadoop的jar替换hbase的lib目录下的hadoop的jar。
dfs.datanode.max.xcievers同时处理文件上限(默认256，应该设>=4096)。

- hbase安全
>基于用户、用户组和角色对表(或列族、列)进行安全检查。

- zookeeper
>是hbase集群的协调器，负责HMaster单点问题+root表路由。

## 2. 运行模式
- 单机模式
>默认模式，使用本地文件系统，所有服务和zookeeper运行在同一JVM，zookeeper监听端口，客户端连接。
`hbase.rootdir：`数据存放位置
`hbase.zookeeper.property.dataDir：`zookeeper数据存放位置
- 分布式模式  
伪分布式（所有进程在同一机器不同JVM）+完全分布式（所有进程分布在各个节点），都需要HDFS，可使用相同验证脚本。  
- 伪分布式
配置hbase-site.xml会覆盖默认的hbase-default.xml。
hbase.rootdir：HDFS（默认3个副本）使用的目录位置。eg.hdfs://localhost:9000/hbase，把HDFS绑定到了localhsot，其他机器连不上，需替换为主机名。  
>>1. 在同一服务器启动额外备份HMaster:
`bin/local-master-backup.sh start n(n<=9) ...`   
停止：`cat /${PID_DIR}/hbase-${USER}-n(n<=9)-master.pid|xargs kill -9`
*可启动9个备份HMaster，最多10个HMaster。*
>>2. 增加额外的regionserver，*最多100个*：  
`bin/local-regionservers.sh start 1 2 3 ...`  
停止：`bin/local-regionservers.sh stop 1 2 3...`
- 完全分布式
1. zookeeper集群  
**zoo.cfg** 
```
//心跳时间,2s
tickTime=2000
//zookeeper集群连接到Leader初始化连接忍受的心跳时间间隔数，10*tickTime=20s
initLimit=10
//leader和follower间的请求应答时长，5*tickTime=10s
syncLimit=5
//该目录创建myid，编辑数字n
dataDir=/home/hadoop/zookeeper-${version}/zookeeperdir/zookeeper-data
dataLogDir=/home/hadoop/zookeeper-${version}/zookeeperdir/logs
clientPort=2181
//第n号服务器=ip:该服务器与集群leader交换信息端口:执行选举的通信端口
server.1=host1:2888：3888

//最后设置环境变量
```

>2. zookeeper测试  
```
zkServer.sh start
zkServer.sh stop
zkServer.sh status
jps
```
>3. hbase配置

```
hbase-env.sh
//默认true，由hbase自己管理zookeeper集群。使用独立安装的zookeeper，改为false
export HBASE_MANAGES_ZK=false 

hbase-site.xml
  <property>
  <name>hbase.rootdir</name>
#regionserver的共享目录，持久化hbase。HDFS的NameNode运行在主机名host1的9000端口
默认file:///tmp/hbase-${user.name}/hbase
  <value>hdfs://host1:9000/hbase</value>
  </property>
  <property>
#默认false单机模式，hbase和zookeeper运行在同一个JVM，改为true为分布式
  <name>hbase.cluster.distributed</name>  
  <value>true</value>
  </property>
  <property>
#生产环节部署奇数个zookeeper，偶数个不行。越多可靠性越强，最好分配独立磁盘(保证高性能)。
#集群负载重时，别把zookeeper和regionser运行在同一机器。
  <name>hbase.zookeeper.quorum</name>
  <value>host1, host2,host3</value>
  </property>
  <property>
#zookeeper快照信息的位置，默认/tmp，重启清空
  <name>hbase.zookeeper.property.dataDir</name>
  <value>/home/hadoop/zookeeperdata</value>
  </property>
 
  regionservers    
#是从机器的域名，希望运行的全部regionserver，随集群启动和停止
    host1
    host2
    host3
    
启动：  
start-hbase.sh  
停止(hmaster和regionser全部退出):  
stop-hbase.sh
//集群越大，停止hbase时间越长，在彻底停止前，不能停hadoop。
```
> jps在HMaster：HMaster HQuorumPeer；  
在RegionServer：HRegionServer HQuorumPeer


## ui和shell
>hbase的web界面，默认在hmaster的60010上，http://master:60010。ui包含hbase版本、zookeeper集群主机列表、hbase根目录等。-ROOT-和.META.永远存在且加载成功，反之无法读写数据。

```
hbase shell
长按ctrl+删除，可以正常删除字符
scan 'tablename'
详细命令，参考网络资料
```



