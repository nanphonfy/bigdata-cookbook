### 一、深度剖析document数据路由原理

#### （1）document路由到shard上是什么意思？
>一个index的数据会被分为多片，每片都在一个shard中。所以一个document只能存在于一个shard中。  
当客户端创建document时，es会决定document放在index的哪个shard上。  
该过程称为document routing(数据路由)。

#### （2）路由算法：shard = hash(routing) % number_of_primary_shards
>eg.一个index有3个primary shard（P0，P1，P2）。每次增删改查一个document时，都有个routing number，默认为这个document的_id（可能手动指定，也可能自动生成）
routing = _id，假设_id=1。会将这个routing值，传入一个hash函数中，产出一个routing值的hash值，eg.hash(routing) = 21，然后将hash函数产出的值对这个index的primary shard的数量求余数，21 % 3 = 0 这样决定该document放在P0。
>>决定一个document在哪个shard上，最重要的是routing值，默认是_id，也可手动指定，相同的routing值，产出的hash值一定是相同的。  
无论hash值是几，无论是什么数字，对number_of_primary_shards求余数，结果一定是在0~number_of_primary_shards-1之间这个范围内（0,1,2）。

#### （3）_id or custom routing value
>默认的routing就是_id
也可在发送请求时，手动指定一个routing value，比如说put /index/type/id?routing=user_id  
手动指定routing value是很有用的，可以保证某一类document一定被路由到一个shard，那么在后续进行应用级别的负载均衡，以及提升批量读取性能时，是很有帮助的。

#### （4）primary shard数量不可变的谜底
>结合图片，跟index路由数据丢失有关系。  
primary shard一旦建立index，不允许修改，但replica shard可以随时修改。原因如下图：


### 二、document增删改内部原理
>（1）客户端选择一个node发送请求，该node就是coordinating node（协调节点）;    
（2）coordinating node，对document进行路由，将请求转发给对应的node（有primary shard）  
（3）实际的node上的primary shard处理请求，然后将数据同步到replica node；  
（4）coordinating node，如果发现primary node和所有replica node都搞定之后，就返回响应结果给客户端。  

### 三、写一致性原理以及quorum机制
有视频的内容，没有笔记，这部分需要自己补写，切记！！



