## 1. 模式设计
>数据库系统都存在模式设计问题。hbase模式结构：表、rowkey、列族、时间戳。
>>与RDBMS区别：单元格(cell)行有序，列族(column family)下的列(qualifier)可自由添加。

属性 | hbase|RDBMS
---|---|---
存储模式 | 基于列式存储|基于表格模式+行式存储
数据类型 | 仅字符串|很丰富
数据操作 | CRUD,不支持join|表连接+函数
建立索引 | 只能在rowkey|多列
更新保护 | 保留旧版本|替换
可伸缩性 | 兼容性，轻易&uarr;节点|需中间层，牺牲性能
**表1-1 表设计模式对比**

>**注意：** ①hbase不支持join，需一条行记录+一个特定关键字解决；②rowkey设计，eg.用户最近听的音乐，userId放前面，可聚合同一用户记录，时间串倒置使记录从新到旧排列。

## 2. rowkey设计
>rowkey，按字典顺序存储。rowkey的设计(有序+底层存储格式)决定了访问hbase表的性能：①region(region会把内存数据刷写到hfile)基于rowkey区间的行服务并负责区间每一行；②hfile在硬盘上存储有序行。不用rowkey查找，只能通过扫描全表。  

- 字典顺序：保持时间最新，最大值-rowkey来倒置；
- 尽量散列：避免读写集中在个别region。eg.用户观影记录，①反转userid；②对userid散列；③userid取模，MD5加密，前六位做为userid前缀。  
>**索引表：** rowkey可设计：观影时间的long值（前缀），建议都为string（方便线上shell排错、让数据均匀分布、不必考虑存储成本）。
- rowkey尽量短：rowkey太长，会增加存储开销，内存利用率降低，进而降低索引命中率。一般时间使用Long+尽量使用编码压缩。

## 3. 列族
>列族是一些列的集合。物理上，一个列族的成员都存储在一起。目前hbase不能很好处理>=2个列族【每个表最好不超3个列族】，故尽量减少列族数量，最好只使用一个列族。
>>flush和compaction是针对一个region。  
>>>- flush：当一个列族操作大量数据引发flush时，其他列族不管操作数据多少也要flush。
>>>- compaction:触发条件：根据一个列族下全部文件数量(不是文件大小)。
>>当很多列族flush和compaction时，很多IO负载无用，解决途径为只设置一个列族。(eg.A列族100万行,B列族10亿行，A族会被分散到很多region，导致扫描A效率降低。多列族执行flush和compaction，会造成很多IO负载。)

- hfile数据块  
>默认64KB，索引存储每个hfile数据块的起始键。数据块&darr;，索引&uarr;，内存占用&uarr;，序列扫描性能&uarr;，随机读&darr;。反之，内存占用&darr;，随机读&uarr;。

- 数据块缓存(BLOCKCACHE)
>默认打开。把数据放入读缓存，不一定能&uarr; 性能。eg.表或表列族只顺序化扫描访问或很少访问，可关闭列族缓存。关闭，可腾出更多缓存给其他表或同一表的列族用。*IN_MEMORY默认false,实际应用可设置true。（具体优化情景分析查阅网络资料）*

- 布隆过滤器(Bloom Filter)
>默认NONE。ROW、ROWCOL分别表示行级(检查特定行键是否不存在)、列标识符(检查行和列标识符 体是否不存在)布隆过滤器。**ROWCOL空间开销>ROW**。
只在整个数据块起始行键建立索引，粒度不够细。eg.某行占100字节，hfile(64*1024)/100=655.53，只把起始行放索引，要查找的行不一定就在特定数据块的行区间。原因：①表不存在该行；②存在另一个hfile；③在memstore。(这些情况，读硬盘的数据库带来IO开销，滥用缓存，影响性能)
>>Bloom Filter对每个数据块或行内单元格做反向测验，查询某行或某列时，先检查Bloom Filter(确定回答在不在该数据块或该单元格)。  
**代价:** 占用额外的索引空间，并随数据&uarr;而&uarr;。

- 数据压缩(COMPRESSION)  
>压缩hfile(存放到HDFS，只在硬盘压缩，内存MemStore、BlockCache或网络传输没压缩)，节省磁盘IO，但&uarr;读写CPU。推荐打开表压缩，关闭它的情况有：①数据不能被压缩；②CPU利用率有限制。  
hbase可使用的压缩编码：LZO、SNAPPY、GZIP，前两者最流行(解压速度相当)，SNAPPY更容易和hadoop、hbase发行版捆绑在一起。  
改变列族压缩编码，可直接更改表(先将表下线，更改完后再上线)，下次region合并会采用新编码压缩（该过程不需建新表+复制数据）。

- 单元时间版本(VERSIONS)
>默认每个单元格维护3个时间版本。可设置指定列族属性，eg.create 'table',{NAME=>'colfml',VERSIONS=>1,TTL=>'18000'}

- 生存时间(TTL)
>Time To Live，设置单元格生存周期。默认值INT.MAX_VALUE，永远启用，单位秒。eg.TTL=>'18000'，18000s/60*60s=5小时，超过5小时在下次大合并时会被删除。

## 4. 案例
- 互粉

列名 | 含义
---|---
id | 主键
nickname | 昵称
... | ...
用户表

列名 | 含义
---|---
user_id | 用户主键
fans_id | 粉丝主键
type |互粉类型
用户粉丝对应表

---

rowkey | 列| 族
---|---|---
user_id | info|fans 
. |info:nickname|fans:userid=type
. |...|.

- 消费记录

列名 | 含义
---|---
id | 主键
user_id | 用户id
product | 购买商品
time |时间
---
rowkey | 列| 族
---|---|---
user_id+Long.MAX_VALUE-当前时间戳+productid | name
. |name:product|naem:time

- 店铺商品  

列名 | 含义
---|---
id | 主键
name | 店铺名称
... | ...
店铺表

列名 | 含义
---|---
id | 主键
name | 商品名称
... | ...
商品表

列名 | 含义
---|---
id | 主键
shop_id | 店铺主键
item_id| 商品主键
type |关联类型
店铺商品关系表

---
rowkey | 列| 族
---|---|---
shopid | info|item
. |info:name ...|item:itemid=type

rowkey | 列| 族
---|---|---
itemid | info|shop
. |info:name ...|shop:shopid=type

- 分类

id| name|parentid|childid
---|---|---|---
1|生物|.|2
2|动物|1|3,4,5
3|狗|2,1|.
4|猪|2,1|.
5|牛|2,1|.
分类

---
rowkey | 列| .|族
---|----|---|---
id | name|parent:id|child:id
1 |生物|.|child:2=动物
2 |动物|parent:1=生物|child:3=狗,child:4=猪,child:5=牛
3|狗|parent:2=动物,parent：1=生物|.