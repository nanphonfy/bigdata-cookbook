## 1. 背景
在hbase搭建sql引擎层，整合hive、Phoenix、impala等。
nosql，不用sql查询，一般具备水平扩展特性。现有sql解决方案，不是水平可伸缩。
>在hbase上提供sql接口原因：①更易学易用；②sql编写减少代码量；③可在服务端优化。eg.group by查询，利用hbase协同处理器，聚合可在服务器（极大较少传输量）执行，也可在客户端并行（更快）执行。

## 2. 基于hbase的sql引擎
- hive整合hbase  
利用两者对外API互相通信。
- Phoenix  
构建在hbase的sql中间层，可执行sql。完全用Java编写，不是map-reduce job，是通过标准化语言。10万~100万的简单查询，优于hive。进行相似查询，比原生hbase api、协同处理器、自定义过滤器的impala、openTSDB更快一些。
- kundera  
基于JPA的对象映射框架，支持Cassandra、hbase、mongodb、redis、oracleNoSQL、neo4j等。
- lealone  
淘宝，可用于hbase的分布式SQL引擎，支持高性能分布式事务，纯Java的BigTable和RDBMS融合。
- hbase-sql  
将sql转成scan执行
- interactive query  
非开源，Intel使用封装后的sql解析层替换hive，使用coprocessor聚合计算，性能更强。
- impala  
cloudera发布，实时查询（黑斑羚），3~90倍的hive sql速度。与hive相同的元数据、sql语法等，兼容性较差。
## 3. hive整合hbase
>hive，基于hadoop的数据仓库工具，将结构化文件->数据库表，完整的sql查询(HQL)功能，将sql转换为mapreduce任务运行。比pig有更丰富的类型，更类似sql、table/partition元数据持久化。学习成本低，适合数据仓库统计分析。

- hive所有表  
>内部-本地表、内部-非本地表、外部-本地表、外部-非本地表，hive整合hbase所使用的都是内外部-非本地表。

- hive特性  
HiveQL查询接口、HDFS底层存储、mapreduce执行层
- 整体结构   
>client：CLI、HWI、thriftserver、JDBC、ODBC等；  
metastore：存放元数据的模块；  
driver：编译、优化、执行hive sql的模块。（解析为mapreduce任务）。

- hive整合hbase架构   
>使用hive storage handlers整合，只需变动driver层，driver调用handlers，将任务解析映射到hbase集群(直接或转化为mapreduce访问)。

优点 | 缺点
---|---
配置使用简单 |查询速度慢，大部分都要启动mapreduce
代码量降低 |每个mapreduce要启N个handler连集群，占用连接
低耦合整合，Apache官方支持 |列映射诸多限制  



