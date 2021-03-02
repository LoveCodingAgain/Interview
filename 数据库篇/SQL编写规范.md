## 1.1 SQL
> - 写完SQL先explain查看执行计划(SQL性能优化)日常开发写SQL的时候，尽量养成这个好习惯呀：写完SQL后，用explain分析一下，尤其注意走不走索引。
```
explain select * from user where userid =10086 or age =18;
``` 

> - 操作delete或者update语句，加个limit(SQL后悔药）。降低写错SQL的代价,SQL效率很可能更高,避免了长事务,数据量大的话，容易把CPU打满;
```
delete from euser where age > 30 limit 200;
```

> - 设计表的时候，所有表和字段都添加相应的注释（SQL规范优雅）设计数据库表的时候，所有表和字段都添加相应的注释，后面更容易维护.
```
 CREATE TABLE `account` (
   `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键Id',
   `name` varchar(255) DEFAULT NULL COMMENT '账户名',
   `balance` int(11) DEFAULT NULL COMMENT '余额',
   `create_time` datetime NOT NULL COMMENT '创建时间',
   `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
   PRIMARY KEY (`id`),
   KEY `idx_name` (`name`) USING BTREE
 ) ENGINE=InnoDB AUTO_INCREMENT=1570068 DEFAULT CHARSET=utf8 ROW_FORMAT=REDUNDANT COMMENT='账户表';
```

> - SQL书写格式，关键字大小保持一致，使用缩进。（SQL规范优雅）

```
SELECT stu.name, sum(stu.score)
FROM Student stu
WHERE stu.classNo = '1班'
GROUP BY stu.name
```

> - INSERT语句标明对应的字段名称。（SQL规范优雅）

```
insert into Student(student_id,name,score) values ('XXX','XXX','100');
```

> - 变更SQL操作先在测试环境执行，写明详细的操作步骤以及回滚方案，并在上生产前review。（SQL后悔药）

> - 设计数据库表的时候，加上三个字段：主键，create_time,update_time。（SQL规范优雅）

> - 写完SQL语句，检查where,order by,group by后面的列，多表关联的列是否已加索引，优先考虑组合索引。（SQL性能优化）

> - 修改或删除重要数据前，要先备份，先备份，先备份。（SQL后悔药）

> - where后面的字段，留意其数据类型的隐式转换（SQL性能优化）【因为不加单引号时，是字符串跟数字的比较，它们类型不匹配，MySQL会做隐式的类型转换，把它们转换为浮点数再做比较，最后导致索引失效】
```
select * from user where userid ='123';
```

> -  尽量把所有列定义为NOT NULL。（SQL规范优雅）

> - 修改或者删除SQL，先写WHERE查一下，确认后再补充 delete 或 update（SQL后悔药）

> - 减少不必要的字段返回，如使用select <具体字段> 代替 select * （SQL性能优化）【节省资源、减少网络开销。可能用到覆盖索引，减少回表，提高查询效率。】

> - 所有表必须使用Innodb存储引擎。（SQL规范优雅）

> - 数据库和表的字符集统一使用UTF8。（SQL规范优雅）

> - 尽量使用varchar代替 char。（SQL性能优化）

> - 如果修改字段含义或对字段表示的状态追加时，需要及时更新字段注释（SQL规范优雅）

> - SQL修改数据，养成begin + commit 事务的习惯。(SQL后悔药)

> - 索引命名要规范，主键索引名为 pk_ 字段名；唯一索引名为 uk _字段名 ； 普通索引名则为 idx _字段名。（SQL规范优雅）

> - WHERE从句中不对列进行函数转换和表达式计算。

> - 如果修改/更新数据过多，考虑批量进行。【大批量操作会会造成主从延迟,大批量操作会产生大事务,阻塞】

> - UPDATE更新多个字段的时候注意使用set XXX,XXX隔开，注意不要用and相连。

> - UPDATE更新语句的时候注意引号的位置,不要包含where条件了,危险。
```
update TableName set source_name ="where source_name" ="XXXX"
update TbaleName set source_name= "XXX" ="YYY"
select "XXX" ="YYY" 的值为0,更新数据直接回导致为0的,所以说线上更新时候要加limit
```

