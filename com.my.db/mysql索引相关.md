#概览
```
1.对innodb引擎来说
    .frm后缀的文件存储的是表结构
    .ibd后缀的文件存放索引文件和数据(需要开启innodb_file_per_table 参数)
2.索引是按照特定的数据结构把数据表中的数据放在索引文件中，以便于快速查找。数据因增删改变化时，mysql会自动维护索引，故不当的索引会影响mysql性能
3.索引存在于磁盘中，会占据物理空间
```
#索引分类
```
1.B+tree索引  【最常见的索引类型，大部分引擎都支持B树索引】
    为什么使用B+tree 
            两种数据结构都更加矮胖。他们两之间一个很大的不同是B+树的节点上不储存value，只储存key，而叶子节点上储存了所有kv集合，并且节点之间都是有序的。这样的好处是每一次磁盘IO能够读取的节点更多，也就是树的度可以设置的更大一些，因为每次磁盘IO读取的磁盘页数是一定的，例如每次磁盘IO能够读取1页=4kb，那么省去value的情况下同样一页数据能够读取更多的key，这样就大大减少了磁盘的IO次数。此外，B+树是排序好的数据结构，数据库中><或者order by等都可以直接依赖这一特性
    
主键索引  
        主键是一种唯一性索引，但它必须指定为“PRIMARY KEY”
        每个表只能有一个主键。 （主键相当于聚合索引，是查找最快的索引）
        create table时添加primary key 或者 alter table add primary key
    唯一索引
        表示唯一的，不允许重复的索引，如果该字段信息保证不会重复例如身份证号用作索引时，可设置为unique
        创建唯一索引CREATE UNIQUE INDEX 索引名 ON 表名(列的列表)
        修改表ALTER TABLE 表名ADD UNIQUE 索引名 (列的列表)
        创建表时指定索引：CREATE TABLE 表名( [...], UNIQUE 索引名 (列的列表) )
    普通索引
        最基本的索引类型，而且它没有唯一性之类的限制
        创建索引: CREATE INDEX 索引名 ON 表名(列名1，列名2,...)
        修改表: ALTER TABLE 表名ADD INDEX 索引名 (列名1，列名2,...)
        创建表时指定索引：CREATE TABLE 表名 ( [...], INDEX 索引名 (列名1，列名 2,...) )
2.HASH 索引  【只有Memory引擎支持，使用场景简单】
3.R-Tree 索引  【空间索引是MyISAM的一种特殊索引类型，主要用于地理空间数据类型】
4.Full-text 索引  【全文索引也是MyISAM的一种特殊索引类型，主要用于全文索引，InnoDB从MYSQL5.6版本提供对全文索引的支持】
4.全文索引
```
#索引增删
```
增加索引
1.ALTER TABLE - ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引
    ALTER TABLE table_name ADD UNIQUE (column_list)
2.CREATE INDEX - CREATE INDEX可对表增加普通索引或UNIQUE索引
    CREATE UNIQUE INDEX index_name ON table_name (column_list)
删除索引
1.如：DROP INDEX index_name ON talbe_name
     ALTER TABLE table_name DROP INDEX index_name
2.如果从表中删除了某列，则索引会受到影响。对于多列组合的索引，如果删除其中的某列，则该列也会从索引中删除。如果删除组成索引的所有列，则整个索引将被删除
查看索引
1.如：show index from tblname;
     show keys from tblname;
```

#索引选择性
```
1.较频繁作为查询条件的应该创建索引
2. 唯一性太差的字段不适合单独创建索引，即使频繁作为查询条件
3.更新非常频繁的字段不适合创建索引
4. 不会出现在 WHERE 子句中的字段不该创建索引
5.可以考虑使用索引的主要有 两种类型的列：在where子句中出现的列，在join子句中出现的列，而不是在SELECT关键字后选择列表的列
6.对字符串列进行索引，应该指定一个前缀长度，可节省大量索引空间，提升查询速度。如一个CHAR(200)列，如果在前10个或20个字符内
不适宜建索引的情况
1.表记录比较少，例如一两千条甚至只有几百条记录的表，没必要建索引，让查询做全表扫描就好了
2.MySQL只对以下操作符才使用索引：<,<=,=,>,>=,between,in, 以及某些时候的like(不以通配符%或_开头的情形)
3.不要过度索引，只保持所需的索引。每个额外的索引都要占用额外的磁盘空间，并降低写操作的性能
```
#索引使用
```
1.or
    当一个表里所有字段都是组合索引时，查询会走索引
    当表里除组合索引外有无索引列，不会走索引
    单列索引的or，会走索引
2.>  <  >= <= between 表达式
    组合索引表达式的后方字段索引不会使用，索引失效
    单个列索引，会走索引
3.索引覆盖
    select id from table 查询直接获取到id值时不会回表，直接索引覆盖，返回了数据
4.like查询
    like “%tab%” 不会走索引
    like “tab%” 走索引
5.组合索引遵循最左匹配原则
6.使用索引扫描来排序 order by
7.union all，in，or都会走索引，推荐使用in
8.隐式类型转换时，会造成索引失效，如'' 没了，导致类型转换，索引失效
9.更新频繁，区分度不高的字段不适合建索引 一般divide value区分度80%以上建索引
10.当需要进行表连接时，最好不要超过3张表，需要join的字段类型最好一致，否则会导致索引失效
11.能使用limit时，最好使用limit
12.单表的索引最好在5个以内（现在无太多限制），单索引字段最好不超过5个，即组合索引字段不超5个
13.
```

#优化小点
```
1.单库中尽量使用自增主键
  分布式数据库使用雪花算法等生成主键
2.使用索引列查询是，尽量不要使用表达式，把运算放业务中处理而不是数据库中
3.尽量使用主键查询，而不是其他索引，主键查询不会触发回表
4.使用前缀索引 如：left(4)
```

#sql优化使用总结
```
一、拆解、分批
1、多层嵌套查询改为多次查询。
2、in 子查询影响查询性能，用join方式代替。
3、一次查询数量过于庞大，拆成多次查询、拼装。
4、用了反向查询（比如 not in）或者 in 语句参数集太多，可能会导致全表扫描 == 可能的情况下，拆分语句，在内存中过滤发。
5、将⼤字段、访问频率低的字段拆分到单独的表中存储，分离冷热数据。
二、不走索引情况
1、 where条件中的1=1：查询条件永远为真，可能导致 WHERE 条件失效进行全表查询。
2、使用函数或者隐式转化会导致不走索引。
3、like 匹配通配符号在前面的时候，不走索引。
4、使用了否定条件。
5、or 其中一个有索引，另一个没有的情况。
6、多列索引需要满足最左匹配原则。
7、两张表字符集不一样或者编码不一样，连表查询时。
8、in 的内容过多，会不走索引。
9、低区分度的列上索引，可能不会补使用。
10.尽量不要使用null not null判断
11.尽量不要使用字符拼接函数
三、高效写法
1、用了limit没有order by 可能会导致结果不确定性，一定要加order by。
2、跑批任务使用limit扫描表会导致慢查询，在limit前加上 where startId > 条件，减少db io。
3、为 group by 显示添加 order by 条件，如果没有，会导致无谓的排序产生，实在不需要排序的，都加上了'order by null’
4、有别名的，都显示的加上 AS。
```