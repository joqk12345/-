
## SQL数据库
1. 连接相关
    1. Inner join 内连接产生的结果是AB的交集
    2. Left [Outer] join 产生表A的全集,而B表中匹配的则有值,则以null值取代
    3. right [Outer] join 产品的b的全集,而a表中没有的值则也null值取代
    4. Full  [Outer] join 产生的是A与b的并集,对于没有匹配的记录,则会以null作为值
    5. select top 子句用于规定要返回的记录的数目
    6. select union操作符用于合并两个或者多个SELECT 语句的结果集( UNION内部的Select 语句必须拥有相同数量的列,列也必须拥有相似的数据类型,每条Select语句中的列的顺序必须相同)
    7. Select
2. 索引(应该建立的字段)
    1. 作为经常需要查询条件的字段
    2. 外键
    3. 经常需要排序的字段
    4. 分组排序的字段


3. mysql中的b-tree索引
    1. b-tree的b是balance
    2. 作用显著减少定位记录所经历的中间过程.
    3. b+tree是B-tree索引的一个变种,大名鼎鼎的MySQL就普遍使用B-tree实现其索引

4. mysql中的hash索引
    1. hash索引其检索效率非常高,索引的检索可以一次定位,所以hash索引的检索效率远高于B-tree索引
    2. Hash索引仅仅满足"=","IN"和"<=>"查询,不能使用范围查询,由于hash索引比较的是进行hash之后的Hash值,所以他只能用于等值的过滤,不能用于基于范围的过滤,因为经过相应的Hash算法处理之后的Hash值的大小关系,并不能保证和Hash运算前完全一样.
    3. hash 索引无法被用来避免数据的排序操作
    4. Hash索引不能利用部分索引键查询(由于has计算的时是利用组合索引一起计算Hash值)
    5. Hash索引在任何时候都不能避免扫描表
        1. 计算原理是将hash键通过hash运算完毕之后将运算结果的Hash值和所对应的行指针信息存放于一个Hash表中,由于不同索引键存在相同的Hash值,所以即使取满足某个Hash键值的数据的记录条数,也无法从hash索引中直接完成查询,还是要通过访问表中的实际数据进行相应的比较,并的到相应的结果
    6. Hash 索引遇到大量Hash值相等的情况后性能并不一定会比B-tree索引性能高
        1. 对于选择性比较低的索引键,如果创建Hash索引,那么将会存在大量记录指针信息存储同一个Hash值相关联,这样要定某一条记录就会非常麻烦,会浪费多次表数据的访问,而造成整体性能的地下

5. mysql的存储引擎介绍与说明
    1. mysan
    2. innodb


## 数据结构相关
1. hash表
    1. hash表也成散列表,也有直接译做哈希表,他同数组链表以及二叉排序树相比较有明显的区别,
    2. 他能够快速定位到想要查找到的记录,而不是与表中存在的记录的关键字进行比较来查找.
    3. 这源于,他采用了函数映射的思想将记录的储存位置与记录的关键字关联起来了,从而能够快速的进行查找
    4. 时间复杂度变为O(1)
    5. 通过key直接过去到该记录在表中的存储位置,能够省掉中间关键词比较的这个中间环节
2. 原理
2. hashMap和hashTable的区别
    1. HashMap和HashTable都实现了Map接口,但决定用哪个之前先要弄清楚他们之间的分别.
    2. HashMap几乎可以等价于HashTable,除了HashMap是非cynchronized,并可以接受null,
    3. HashTable是线程安全的,,是synchronized,多个线程可以共享HashTable;如果没有正常同步的话多线线程是不能共享HashMap的.
    4. HashMap的迭代器(Iterator)是fail-fast迭代器
    5. 单线程环境下HashTable要比HashMap要慢
    6. HashMap不能保证随着时间的推移Map中的元素次序是不变的.
3. 重要术语
    1. sychronized意味着在一次仅有一个线程能够更改HashTable.就是说任何线程要修改更新Hashtable时要活得同步锁,其他线程要等同步锁释放之后才能在次更新HashTable
    2. 结构上的更改是只删除或者插入一个元素,这样会影响map的结构
    3. Fail-safe和iterator迭代器相关.如果某个集合对象创建了Iterator或者ListIterator,然后其他的线程试图"结构上"更改集合对象,将会抛出"ConcurrentModificationException".但是其他线程可以通过set()方法更改集合对象是允许的,因为这并没有从结构上更改集合对象.
4. 能够让HashMap同步
    1. HashMap可以通过下面的术语同步:
        1. Map m = Collection.synchronizeMap(hashMap);
5. Haashtable和HashMap有一个主要的不同:
    1. 线程安全以及速度.
    2. 仅在需要线程安全的时候使用hashTable,
    3. 在高版本上的话请使用ConcurrentHashMap

## 算法相关

## 多媒体相关
1. bit-map
2. 语音

##  MongoDB

## HBase 
1. flash数据结构

##　redis



