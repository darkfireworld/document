# KV数据库

随着NoSQL的流行，越来越多的NoSQL数据库被采用。而作为NoSQL中的KV数据库，也是经常出现在架构师的手中，被灵活的使用。

## 存储引擎

为了高效的检索数据，我们需要设计合理的数据引擎，来达到我们想要的效果。

### Hash引擎

借助Hash表的算法，可以非常方便的检索数据，其效率远远高于B-Tree类型的引擎。但是，往往也存在一些不便之处，比如说模糊查询性能差，因为需要扫描整个KEY表。

代表：redis，memcache

### B-Tree引擎

采用B-Tree数据结构作为引擎，特点就是对大数据的磁盘存储比较好，且支持模糊搜索。但是时间复杂度为O(LogN)。

代表：MySQL-Innodb

## KV表设计

在设计KV数据库的表的时候，其实还是和RDB一样，虽然**KV数据库仅仅只有KEY-VALUE两个属性**。

### KEY设计

KEY的设计，主要考虑快速的搜索这个问题，也就是RDB中的主键了。 设计公式为：

**KEY = 表名:PRIMARY_KEY**

### VALUE设计

在RDB中，数据的存储只能采用行的方式，这极大的限制了高效的数据设计，比如说，设计一个用户的所有好友关系，RDB中需要设计为：

    USR_ID , FRIEND_ID

    1      ,10
    1      ,11
    1      ,12
    2      ,10

这种格式，造成的许多冗余数据。
但是在KV数据库中，就不存在这种问题，因为VALUE是丰富的，比如说REDIS中的SET类型，设计的结果为：

    USR_ID:{
        FRIEND_ID,
        FRIEND_ID
    }

利用KV数据库，丰富的Value类型，极大的扩展了数据库设计的模型。

### Key+Value = RDB

在RDB数据库中，一行数据可能包含多个属性，而KV数据库是否可以完整的表示这些属性？答案是可以的，比如说，把所有属性通过JSON格式化作为Value，然后使用RDB中的主键作为Key，既可以完整的表达出这一行数据。公式：

    KEY = RDB_SECHEMA:RDB_PRIMARY_KEY
    VALUE = JSON(RDB_FIELDS)

**如果RDB的一张表的属性过多，则会造成KV的VALUE过大，所以需要注意合理的切分数据库字段。**

## 总结

通过上面对比，**KV数据库相当于只有主键的RDB数据库，并且扩展了VALUE类型的数据库**，能非常方便的设计出合适的数据模型。比如说REDIS的VALUE类型，就非常的丰富，具有：

1. String ：存储基本数据，比如说JSON格式的对象
2. List   ：可以做队列
2. Hash   ：多属性MAP对象
3. Set    ：可以做好友表等
4. SortSet ： 需要排序的Set，比如说阅读排行版

## 参考

[redis数据库的键值设计](http://my.oschina.net/fsmwhx/blog/152130)

[Redis快速入门](http://www.yiibai.com/redis/redis_quick_guide.html)

[Hash和Map](http://my.oschina.net/darkgem/blog/636728)