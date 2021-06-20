## 1.1 SQL优化
[2021最新版](https://mp.weixin.qq.com/s/4P_sPFbf20etv4TrHgCifA)
> 线上优化成本: 硬件<系统配置<数据库表结构<SQL及索引.线上很多数据库CPU持续90+,大概率是频繁执行的SQL语句没有合适的索引甚至就没加索引!

> 首先，对于MySQL层优化我一般遵从五个原则：
  减少数据访问：设置合理的字段类型，启用压缩，通过索引访问等减少磁盘IO
  返回更少的数据：只返回需要的字段和数据分页处理 减少磁盘io及网络io
  减少交互次数：批量DML操作，函数存储等减少数据连接次数
  减少服务器CPU开销：尽量减少数据库排序操作以及全表查询，减少cpu 内存占用
  利用更多资源：使用表分区，可以增加并行操作，更大限度利用cpu资源

> 优化SQL核心思想：最大化利用索引、避免全表扫描、减少无效数据的查询

> SQL语句的语法顺序

```
1. SELECT 
2. DISTINCT <select_list>
3. FROM <left_table>
4. <join_type> JOIN <right_table>
5. ON <join_condition>
6. WHERE <where_condition>
7. GROUP BY <group_by_list>
8. HAVING <having_condition>
9. ORDER BY <order_by_condition>
10.LIMIT <limit_number>
```

> SQL语句的执行顺序
```
FROM
<表名> # 选取表，将多个表数据通过笛卡尔积变成一个表。
ON
<筛选条件> # 对笛卡尔积的虚表进行筛选
JOIN <join, left join, right join...> 
<join表> # 指定join，用于添加数据到on之后的虚表中，例如left join会将左表的剩余数据添加到虚表中
WHERE
<where条件> # 对上述虚表进行筛选
GROUP BY
<分组条件> # 分组
<SUM()等聚合函数> # 用于having子句进行判断，在书写上这类聚合函数是写在having判断里面的
HAVING
<分组筛选> # 对分组后的结果进行聚合筛选
SELECT
<返回数据列表> # 返回的单列必须在group by子句中，聚合函数除外
DISTINCT
# 数据除重
ORDER BY
<排序条件> # 排序
LIMIT
<行数限制>
```

> 一、避免不走索引的场景
> - 尽量避免在字段开头模糊查询，会导致数据库引擎放弃索引进行全表扫描。
```
SELECT * FROM t WHERE username LIKE '%陈%'
```
```
SELECT * FROM t WHERE username LIKE '%陈%'
```
   
> - 尽量避免使用in 和not in，会导致引擎走全表扫描。
  
> - 优化方式：如果是连续数值，可以用between代替。
  
```
SELECT * FROM t WHERE id BETWEEN 2 AND 3
``` 
> - 如果是子查询，可以用exists代替。
  
```
-- 不走索引
select * from A where A.id in (select id from B);
-- 走索引
select * from A where exists (select * from B where B.id = A.id);
```
  
> - 尽量避免使用 or，会导致数据库引擎放弃索引进行全表扫描。优化方式：可以用union代替or。如下：

```
SELECT * FROM t WHERE id = 1
   UNION
SELECT * FROM t WHERE id = 3
```

> - 尽量避免进行null值的判断，会导致数据库引擎放弃索引进行全表扫描。如下：

```
SELECT * FROM t WHERE score IS NULL
优化为
SELECT * FROM t WHERE score = 0
```

> - 尽量避免在where条件中等号的左侧进行表达式、函数操作，会导致数据库引擎放弃索引进行全表扫描。可以将表达式、函数操作移动到等号右侧。如下：

```
-- 全表扫描
SELECT * FROM T WHERE score/10 = 9
-- 走索引
SELECT * FROM T WHERE score = 10*9
```

> - 隐式类型转换造成不使用索引

```
select col1 from table where col_varchar=123;
```

> - order by 条件要与where中条件一致，否则order by不会利用索引进行排序

```
-- 不走age索引
SELECT * FROM t order by age;
-- 走age索引
SELECT * FROM t where age > 0 order by age;
```

> - where条件仅包含复合索引非前置列,如下：复合（联合）索引包含key_part1，key_part2，key_part3三列，但SQL语句没有包含索引前置列"key_part1"，按照MySQL联合索引的最左匹配原则，不会走联合索引。

```
select col1 from table where key_part2=1 and key_part3=2
```

> 二、SELECT语句其他优化

> - 避免出现select *

> - 避免出现不确定结果的函数

> - 多表关联查询时，小表在前，大表在后。

> - 在MySQL中，执行 from 后的表关联查询是从左往右执行的（Oracle相反），第一张表会涉及到全表扫描，所以将小表放在前面，先扫小表，扫描快效率较高，在扫描后面的大表，或许只扫描大表的前100行就符合返回条件并return了。
    例如：表1有50条数据，表2有30亿条数据；如果全表扫描表2，你品，那就先去吃个饭再说吧是吧。

> - 使用表的别名

> - 调整Where字句中的连接顺序

> 三、增删改 DML 语句优化

> - 大批量插入数据,减少SQL语句解析的操作，MySQL没有类似Oracle的share pool，采用方法二，只需要解析一次就能进行数据的插入操作,在特定场景可以减少对DB连接次数,SQL语句较短,可以减少网络传输的IO.

```
Insert into T values(1,2),(1,3),(1,4);
```

> 四、查询条件优化

> - 对于复杂的查询，可以使用中间临时表 暂存数据。

> - 优化group by语句。
> - 优化join语句。
> - 优化union查询。
> - 拆分复杂SQL为多个小SQL，避免大事务。
> - 使用合理的分页方式以提高分页效率。

> 五、建表优化

> - 在表中建立索引，优先考虑where、order by使用到的字段。
> - 尽量使用数字型字段（如性别，男：1 女：2），若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。
> - 查询数据量大的表 会造成查询缓慢。主要的原因是扫描行数过多。这个时候可以通过程序，分段分页进行查询，循环遍历，将结果合并处理进行展示。要查询100000到100050的数据。
> - 用varchar/nvarchar 代替 char/nchar。
    尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。   
    不要以为 NULL 不需要空间，比如：char(100) 型，在字段建立时，空间就固定了， 不管是否插入值（NULL也包含在内），都是占用 100个字符的空间的，如果是varchar这样的变长字段， null 不占用空间。

> 六、MySQL的主键设置
> 使用随机id或uuid作为Mysql主键!!!,同等条件下表都存在数据的情况下,uuid要耗时更多.
> - 原理分析:mysql的auto_increment是主键自增的,自增的主键的值是顺序的,所以Innodb把每一条记录都存储在一条记录的后面。当达到页面的最大填充因子时候(innodb默认的最大填充因子是页大小的15/16,会留出1/16的空间留作以后的     修改):
> - 下一条记录就会写入新的页中,一旦数据按照这种顺序的方式加载，主键页就会近乎于顺序的记录填满，提升了页面的最大填充率,不会有页的浪费.新插入的行一定会在原有的最大数据行下一行,mysql定位和寻址很快，不会为计算新行的位置而做出额外的消耗.
> - 减少了页分裂和碎片的产生.

> 使用uuid的索引内部结构弊端:
> - 写入的目标页很可能已经刷新到磁盘上并且从缓存上移除，或者还没有被加载到缓存中，innodb在插入之前不得不先找到并从磁盘读取目标页到内存中，这将导致大量的随机IO.
> - 因为写入是乱序的,innodb不得不频繁的做页分裂操作,以便为新的行分配空间,页分裂导致移动大量的数据，一次插入最少需要修改三个页以上.
> - 由于频繁的页分裂,页会变得稀疏并被不规则的填充,最终会导致数据会有碎片,在把随机值（uuid和雪花id）载入到聚簇索引(innodb默认的索引类型)以后,有时候会需要做一次OPTIMEIZE TABLE来重建表并优化页的填充，这将又需要一定的时间消耗。





