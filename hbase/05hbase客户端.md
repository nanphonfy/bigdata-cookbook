最重要、最常用的客户端：原生Java、shell、thrift客户端。

## 1. 精通原生Java客户端
最主要、最高效的客户端。CRU(没实现)D+建删改表(DDL数据定义语言)+合并、分裂region、分配region。

- 客户端配置

```
zookeeper队列名称：hbase.zookeeper.quorum
zookeeper端口：hbase.zookeeper.property.clientPort
HMaster地址：hbase.master
```

- 表操作入口类(HTable)
>所有方法都有throws IOException)

```Java
//构造函数
public HTable(Configuration conf, String|byte[]|TableName tableName)
//外部维护线程池
public HTable(Configuration conf, byte[]|TableName tableName tableName, ExecutorService pool)

public HTable(TableName tableName, HConnection connection)
//外部维护线程池
public HTable(byte[]|TableName tableName, HConnection connection, ExecutorService pool)
//增
void put(Put put)
put(List<Put> puts)
boolean checkAndPut(final byte[] row, final byte[] family, final byte[] qualifier, final byte[] value, final Put put)
//删
void delete(final Delete delete)
void delete(List<Delete> deletes)
boolean checkAndDelete(final byte[] row, final byte[] family, final byte[] qualifier, final byte[] value, final Delete delete)
//改
Result increment(final Increment increment)
long incrementColumnValue(byte[] row, byte[] family, byte[] qualifier, long amount, boolean writeToWAL)
long incrementColumnValue(final byte[] row, final byte[] family, final byte[] qualifier, final long amount, final Durability durability)
//查
Result get(final Get get)
Result[] get(List<Get> gets)
ResultScanner getScanner(Scan scan)
ResultScanner getScanner(byte[] family)
ResultScanner getScanner(byte[] family, byte[] qualifier)

boolean exists(Get get)
Boolean[] exists(List<Get> gets)

//检测后执行(Compare-And-Set，CAS)，checkAndPut和checkAndDelete处理一行记录时，不希望其他客户端对该行处理。
//检测值是否改变，没有则写入，有则失败。   


//设置属性信息，控制连接写入，客户端性能优化
void setAutoFlushTo(boolean autoFlush)
void setAutoFlush(boolean autoFlush, boolean clearBufferOnFail)
void setOperationTimeout(int operationTimeout)
void setWriteBufferSize(long writeBufferSize)
```
    
- 管理入口类(HBaseAdmin)  
>数据库管理入口类，建删改列表、上下线、管理region、负载均衡、分裂与合并等。


```java
//构造函数
public HBaseAdmin(Configuration c) throws MasterNotRunningException, ZooKeeperConnectionException, IOException 
public HBaseAdmin(HConnection connection) throws MasterNotRunningException, ZooKeeperConnectionException
//增
void createTable(HTableDescriptor desc)
void createTable(final HTableDescriptor desc, byte[][] splitKeys)
void createTable(HTableDescriptor desc, byte[] startKey, byte[] endKey, int numRegions)
void createTableAsync(final HTableDescriptor desc, final byte[][] splitKeys)
//删
void deleteColumn(byte[]|String tableName, String columnName)
void deleteColumn(final TableName tableName, final byte[] columnName)

deleteSnapshot(byte[]|final String snapshotName)
deleteSnapshots(String regex)
deleteSnapshots(Pattern pattern)

void deleteTable(String|byte[]|final TableName tableName)
HTableDescriptor[] deleteTables(String regex)
HTableDescriptor[] deleteTables(Pattern pattern)
改
void modifyColumn(String|byte[]|final TableName tableName, HColumnDescriptor descriptor)
void addColumn(String|byte[]|final TableName tableName, HColumnDescriptor column)

//hbase高级特性，region相关的操作方法。
//region分配、负载均衡、合并、分裂、移动等。
void assign(byte[] regionName)
void unassign(byte[] regionName, final boolean force)
boolean balancer()
void closeRegion(String|byte[] regionname, String serverName)
boolean closeRegionWithEncodedRegionName(String encodedRegionName, String serverName)
void compact(String|byte[] tableNameOrRegionName)
void compact(String|byte[] tableOrRegionName, String|byte[] columnFamily)
void majorCompact(String|byte[] tableNameOrRegionName)
void majorCompact(String|byte[] tableNameOrRegionName, String|byte[] columnFamily)
void split(String|byte[] tableNameOrRegionName)
void split(String|byte[] tableNameOrRegionName, String|byte[] splitPoint)
void move(byte[] encodedRegionName, byte[] destServerName)
```
>快照：一份元信息的合集，不是表的复制而是一个文件名列表。  
完全快照恢复：恢复到之前的表结构和数据，之后不恢复。
```Java
void snapshot(SnapshotDescription snapshot)
void snapshot(String snapshotName, String|byte[] tableName, Type type)
void snapshot(byte[] snapshotName, byte[]|TableName tableName)
snapshot(String snapshotName, String|TableName tableName)
```
- 连接池类(HTablePool)
>通过连接池方式使用HTable，多线程同时写入是线程安全的。HTablePool不是常规意义的线程池，类似简单计数器。HTable写入不是线程安全的，而且要单独维护创建、应用、消亡的整个过程。

```
//maxSize默认值为：2147483647(源代码实现)，当容量大于最大值，不再添加新线程到池中，导致写入始终自动flush。
public HTablePool(Configuration config, int maxSize)
public HTablePool(Configuration config, int maxSize, HTableInterfaceFactory tableFactory)
public HTablePool(Configuration config, int maxSize, PoolType poolType)
public HTablePool(Configuration config, int maxSize, HTableInterfaceFactory tableFactory, PoolType poolType)

public static enum PoolType {
        Reusable,
        ThreadLocal,
        RoundRobin;
...
}

ReusablePool
class ReusablePool<R> extends ConcurrentLinkedQueue<R> implements PoolMap.Pool<R>

RoundRobinPool
class RoundRobinPool<R> extends CopyOnWriteArrayList<R> implements PoolMap.Pool<R>

//每个线程维护自己独有的变量拷贝，彻底消除竞争条件，最大限度CPU调度并发执行，性能更高(以空间换取线程安全)
ThreadLocalPool
class ThreadLocalPool<R> extends ThreadLocal<R> implements PoolMap.Pool<R>

//HTableFactory用户创建HTabel的工具类。
HtablePool pool=new HTablePool(conf,30,factory,PoolType.ThreadLocal);
```
- 创建表

```Java
// 创建表管理类
HBaseAdmin admin = new HBaseAdmin(config); 
// 创建表描述类
HTableDescriptor desc = new HTableDescriptor(TableName.valueOf("nanphonfy"));
//HTableDescriptor desc = new HTableDescriptor("nanphonfy");...
// 创建列族的描述类
for (String columnDescriptor : columnDescriptors) {
    HColumnDescriptor family = new HColumnDescriptor(columnDescriptor);
    desc.addFamily(family);
}
// 将列族添加到表中
desc.addFamily(family);
// 创建表
admin.createTable(desc);
```
- 删除表

```Java
HBaseAdmin admin = new HBaseAdmin(config);
admin.disableTable("nanphonfy");
admin.deleteTable("nanphonfy");
admin.close();
```
- 插入数据

```Java
HTableInterface table = null;
try {
    table = Constants.CONNECTION.getTable(tableName);
    Put put = new Put(Bytes.toBytes(rowKey));// 创建put，添加rowKey
    put.add(Bytes.toBytes(family), Bytes.toBytes(qualifier), Bytes.toBytes(value));// 放入数据
    table.put(put);// 将数据存入table
    table.flushCommits();
} catch (IOException e) {
    e.printStackTrace();
    return false;
} finally {
    try {
        if(table != null) {
            table.close();// 关闭table
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}

//添加字段
Put add(byte[] family, byte[] qualifier, byte[] value)
Put add(byte[] family, byte[] qualifier, long ts, byte[] value)
//获取键值对
List<Cell> get(byte[] family, byte[] qualifier)
//判断是否存在
boolean has(byte[] family, byte[] qualifier)
boolean has(byte[] family, byte[] qualifier, long ts)
boolean has(byte[] family, byte[] qualifier, byte[] value)
boolean has(byte[] family, byte[] qualifier, long ts, byte[] value)
boolean has(byte[] family, byte[] qualifier, long ts, byte[] value, boolean ignoreTS, boolean ignoreValue)
```
- 查询

```java
Result row = null;
try {
    HTableInterface table = Constants.CONNECTION
            .getTable(TableName.valueOf(Constants.NOTE_TABLE_NAME));// 从连接池中获取表
    Get get = new Get(rowKey.getBytes());//获取get，注入rowKey
    //get.addColumn(byte[] family, byte[] qualifier);
    row = table.get(get);
    table.close();// 关闭table
} catch (Exception e) {
    e.printStackTrace();
}

//添加列族或列
Get addColumn(byte[] family, byte[] qualifier)
Get addFamily(byte[] family)
//设置查询属性
void setCacheBlocks(boolean cacheBlocks)
Get setFilter(Filter filter)
Get setMaxVersions()//2147483647
Get setMaxVersions(int maxVersions)
Get setTimeRange(long minStamp, long maxStamp)
Get setTimeStamp(long timestamp)
//查询属性
boolean getCacheBlocks()
public int getMaxVersions()
public TimeRange getTimeRange()
public Set<byte[]> familySet()
Map<byte[], NavigableSet<byte[]>> getFamilyMap()
Map<String, Object> getFingerprint()
Filter getFilter()
byte[] getRow()
int numFamilies()
boolean hasFamilies()
```
- 扫描读
>不确定行键下，遍历全表或部分数据，可限制时间戳、版本数量、列族、列名等。对于扫描操作，设置过滤器比单行读更重要。

```java 
HTableInterface table = Constants.CONNECTION.getTable(TableName.valueOf(tableName));// 从连接池中获取表
Scan scan = new Scan();// 创建scan，用于查询
//version
//scan.setTimeRange(long minStamp, long maxStamp);
//batch and caching
//scan.setBatch(0);
//scan.setCaching(100000);

ResultScanner scanner = table.getScanner(scan);// 通过scan查询结果
for (Result row : scanner) {//循环出每一条row
    System.out.println(row.getRow());
    for (KeyValue kv : row.raw()) {//循环row的不同列
        System.out.println(new String(kv.getRow()));//获取rowkey
        System.out.println(new String(kv.getFamily()));//获取列族
        System.out.println(new String(kv.getQualifier()));//获取列描述
        System.out.println(new String(kv.getValue()));//获取值
        System.out.println(kv.getTimestamp());//获取版本时间
    }
}
table.close();// 关闭table

//添加列族或列
Scan addFamily(byte[] family)
Scan addColumn(byte[] family, byte[] qualifier)
//设置查询属性
void setBatch(int batch)
void setCacheBlocks(boolean cacheBlocks)
void setCaching(int caching)
Scan setFamilyMap(Map<byte[], NavigableSet<byte[]>> familyMap)
Scan setFilter(Filter filter)
Scan setMaxVersions()//2147483647
Scan setMaxVersions(int maxVersions)
Scan setStartRow(byte[] startRow)
Scan setStopRow(byte[] stopRow)
Scan setTimeRange(long minStamp, long maxStamp)
Scan setTimeStamp(long timestamp)
//查询属性
int getBatch()
boolean getCacheBlocks()
int getCaching()
byte[][] getFamilies()
Map<byte[], NavigableSet<byte[]>> getFamilyMap()
Filter getFilter()
Map<String, Object> getFingerprint()
int getMaxVersions()
byte[] getStartRow()
byte[] getStopRow()
TimeRange getTimeRange()
boolean hasFamilies()
boolean hasFilter()
boolean isRaw()
boolean isGetScan()
int numFamilies()
```
- 删除

```
HTableInterface table = null;
try {
    table = Constants.CONNECTION.getTable(TableName.valueOf(tableName));
    Delete del = new Delete(Bytes.toBytes(rowKey));// 创建delete类，存入要删除的rowKey
    //            del.deleteColumn(byte[] family, byte[] qualifier, long timestamp);//
    //            del.deleteColumns(byte[] family, byte[] qualifier);
    //            del.deleteFamily(byte[] family);
    table.delete(del);// 删除数据
    table.flushCommits();
} catch (IOException e) {
    e.printStackTrace();
    return false;
} finally {
    try {
        if (table != null) {
            table.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}

//删除某列某版本
Delete deleteColumn(byte[] family, byte[] qualifier)
Delete deleteColumn(byte[] family, byte[] qualifier, long timestamp)
//删除某列所有版本
Delete deleteColumns(byte[] family, byte[] qualifier)
Delete deleteColumns(byte[] family, byte[] qualifier, long timestamp)
//删除某个列族
Delete deleteFamily(byte[] family)//timestamp=9223372036854775807L
Delete deleteFamily(byte[] family, long timestamp)
//设定删除操作的时间戳
```
## 2. hbase shell
>hbase的shell工具用Ruby编写，使用JRuby解释器。分为：交互式模式(实时随机访问)和命令批处理模式(shell编程批量、流程化处理)。

bin/hbase shell  
help（帮助手册）
shell所有命令分6组：常规(general)、DDL、DML、工具(tools)、复制（replication）和安全（security）。

- general  
status(集群状态)、version(hbase版本)

- ddl  
数据定义语言，建删改列表、上下线表等  
help "${COMMAND_NAME}"  
>- =>赋值，eg{NAME=>'f1'};
>- 字符串必须用单引号，eg'f1'
>- 指定列族特定属性，用花括号，eg{NAME=>'f1',VERSIONS=5}

- dml  
数据操纵语言，增删改查、清空等。

```
1. 扫全表
scan '.META.'
行包括一个列族info和三个列名regioninfo、server和serverstartcode。
2. 指定列名全表扫描
scan '.META.',{COLUMNS=>'info:regioninfo'}
scan '.META.',COLUMNS=>'info:regioninfo'
3. 指定多列、限定返回行数、设置开始行的全表扫描
scan '.META.',{COLUMNS=>['info:regioninfo','info:serverstartcode'],LIMIT=>2,STARTROW=>'XXXX'}
4. 设定时间戳返回全表扫描
scan '.META.',{COLUMNS=>'info:regioninfo',TIMERANGE=>[1505023226047,1505023426047]}
5. 过滤条件全表扫描
scan '.META.',{FILTER=>“(PrefixFilter('前缀') AND (QualifierFilter(=,'binary:regioninfo'))) AND (TimestampsFilter(1505023226047,1505023426047))”}
"="表示比较器，"binary:"表示二进制比较，两个时间戳是数组的两元素不是区间。
```
- tools
>多用于hbase集群管理+调优。包含合并、分裂、负载均衡、日志回滚、region分配和移动、zookeeper信息查看等。eg.compact可合并一张表、一个region的某个列族、一张表的某个列族。

命令 | 含义
---|---
assign | 分配region
balance_switch | 负载均衡器开关
balancer|触发集群负载均衡器
close_region|关闭某region
compact|合并表或region
flush|flush表或region
hlog_roll|regionserver的hlog回滚
major_compact|大合并表或region
move|移动region
split|分裂表或region
unassign|解除某region
zk_dump|zookeeper信息(主节点、regionserver+zookeeper节点状态统计)
- replication  
待完善
- security
>DCL（数据控制语言），三个安全命令：grant、revoke和user_permission。使用前提：附带security的hbase版本+kerberos安全认证。














