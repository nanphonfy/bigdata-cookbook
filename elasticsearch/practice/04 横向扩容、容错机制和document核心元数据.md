### 一.图解横向扩容&容错机制
- 图解横向扩容过程，如何超出扩容极限，以及如何提升容错性

>（1）primary&replica自动负载均衡，6个shard，3 primary，3 replica;  
（2）每个node有更少的shard，IO/CPU/Memory资源给每个shard分配更多，每个shard性能更好;  
（3）扩容的极限，6个shard（3 primary，3 replica），最多扩容到6台机器，每个shard可以占用单台服务器的所有资源，性能最好;  
（4）超出扩容极限，动态修改replica数量，9个shard（3primary，6 replica），扩容到9台机器，比3台机器时，拥有3倍的读吞吐量;  
（5）3台机器下，9个shard（3 primary，6 replica），资源更少，但是容错性更好，最多容纳2台机器宕机，6个shard只能容纳1台机器宕机;   
（6）扩容原理：怎么扩容，怎么提升系统整体吞吐量；另一方面要考虑系统容错性，怎么保证提高容错性，让尽可能多的服务器宕机，保证数据不丢失。

- 扩容&容错1  
![image](https://raw.githubusercontent.com/nanphonfy/note-images/master/bigdata-cookbook/elasticsearch/practice/03/%E6%89%A9%E5%AE%B9%26%E5%AE%B9%E9%94%99.png)

- 扩容&容错2  
![image](https://raw.githubusercontent.com/nanphonfy/note-images/master/bigdata-cookbook/elasticsearch/practice/03/%E6%89%A9%E5%AE%B9%26%E5%AE%B9%E9%94%992.png)

### 二.图解es容错机制：master选举，replica容错，数据恢复
>（1）9 shard，3 node；  
（2）master node宕机，自动master选举，red(不是所有的primary都active)；  
（3）replica容错：新master将replica提升为primary shard，yellow；  
（4）重启宕机node，master copy replica到该node，使用原有的shard并同步宕机后的修改，green。

### 三.初步解析document的核心元数据及图解剖析index创建反例
>1、_index元数据  
2、_type元数据  
3、_id元数据  

```
{
  "_index": "test_index",
  "_type": "test_type",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "test_content": "test"
  }
}
```

#### 1、_index元数据
>（1）代表一个document存放在哪个index中;  
（2）类似的数据放一个索引，非类似的数据放不同索引：product index（包含所有商品），sales index（包含所有商品销售数据），inventory index（包含所有库存相关数据）。若把product，sales，human resource（employee），全放一个大的index(eg.company index)，是不合适的;  
（3）index中包含很多类似的document：即这些document的fields很大一部分是相同的，eg.放了3个document，它们的fields都完全不一样——不类似，不太适合放一个index;  
（4）索引名称必须是小写，不能用下划线开头，不能包含逗号：product，website，blog.

#### 2、_type元数据
>（1）代表document属于index中的哪个类别（type）;  
>（2）一个索引通常会划分为多个type，逻辑上对index中有些许不同的几类数据进行分类：一批相同的数据，可能有很多相同的fields，但可能会有一些轻微的不同，可能有少数fields不一样，eg.商品可能划分为电子商品、生鲜商品、日化商品...;   
>（3）type名称可大写或小写，但不能用下划线开头，不能包含逗号.

#### 3、_id元数据
>（1）代表document的唯一标识，与index和type一起，可唯一标识和定位一个document;  
（2）可手动指定document的id（put /index/type/id），也可不指定，由es自动创建id.