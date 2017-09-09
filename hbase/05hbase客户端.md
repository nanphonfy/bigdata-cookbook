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




