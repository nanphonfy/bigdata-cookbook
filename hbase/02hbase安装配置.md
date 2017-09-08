修改一台机器配置后，要同步(scp或rsync)到集群所有节点，重启hbase使配置生效。hbase依赖的中间件、系统服务、配置。
## 1. 条件
- jdk
>64位JDK才能管理超过4GB的内存。

- ssh服务
>集群模式下，hbase启动、关闭依赖ssh（查看状态：service sshd status），ssh管理所有节点的守护进程。

- 域名系统DNS
>hbase通过host name或domain name获取IP，得保证正反向DNS解析(先查本地/etc/hosts,自己配置而不适用域名解析服务，解析IP更快)正常。多网卡，可通过hbase.regionserver.dns.interface制定主网卡,默认值default。

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

- 分布式模式
