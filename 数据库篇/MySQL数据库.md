## 1.1 如何获取Spring容器对象?
> 1、InnoDB中的锁机制,那么不可避免的要涉及到MySQL的日志系统,binlog、redo log、undo log,数据库事务的原子性、事务性、隔离性、一致性.

> 2、MySQL的日志分类.
> 日志是mysql数据库的重要组成部分,记录着数据库运行期间各种状态信息.mysql日志主要包括错误日志、查询日志、慢查询日志、事务日志、二进制日志、通用查询日志几大类.
> binlog用于记录数据库执行的写入性操作(不包括查询)信息,以二进制的形式保存在磁盘中。binlog是mysql的逻辑日志,并且由Server层进行记录,使用任何存储引擎的mysql数据库都会记录binlog日志.
> 逻辑日志：可以简单理解为记录的就是sql语句.
> 物理日志：mysql数据最终是保存在数据页中的,物理日志记录的就是数据页变更.
> binlog是通过追加的方式进行写入的,可以通过max_binlog_size参数设置每个binlog文件的大小,当文件大小达到给定值之后,会生成新的文件来保存日志.

> 3、binlog使用场景.
> 在实际应用中,binlog 的主要使用场景有两个,分别是主从复制和数据恢复.
> ①、主从复制:在Master端开启binlog,然后将binlog发送到各个Slave端,Slave端重放binlog从而达到主从数据一致.
> ②、数据恢复:通过使用mysqlbinlog工具来恢复数据.

> 4、binlog刷盘时机.
> ①、对于InnoDB存储引擎而言,只有在事务提交时才会记录biglog,此时记录还在内存中,那么biglog是什么时候刷到磁盘中的呢?
> mysql通过sync_binlog参数控制biglog的刷盘时机,取值范围是 0-N:
> 0：不去强制要求，由系统自行判断何时写入磁盘.
> 1：每次 commit 的时候都要将 binlog 写入磁盘.
> N：每N个事务，才会将 binlog 写入磁盘.
从上面可以看出,sync_binlog最安全的是设置是1,这也是MySQL 5.7.7之后版本的默认值.但是设置一个大一些的值可以提升数据库性能,因此实际情况下也可以将值适当调大,牺牲一定的一致性来获取更好的性能.

> 5、binlog日志格式.
> binlog日志有三种格式,分别为STATMENT、ROW和MIXED.
> 在 MySQL 5.7.7 之前,默认的格式是STATEMENT,MySQL 5.7.7 之后,默认值是ROW.日志格式通过binlog-format指定.
> ROW：基于行的复制(row-based replication, RBR ),不记录每条sql语句的上下文信息，仅需记录哪条数据被修改了.
> STATMENT：基于SQL语句的复制(statement-based replication,SBR),每一条会修改数据的sql语句会记录到binlog中.
> MIXED：基于STATMENT和ROW两种模式的混合复制(mixed-based replication, MBR),一般的复制使用STATEMENT模式保存binlog对于STATEMENT模式无法复制的操作使用ROW模式保存binlog.

> 6、undo log.
> 数据库事务四大特性中有一个是原子性,具体来说就是原子性是指对数据库的一系列操作,要么全部成功,要么全部失败,不可能出现部分成功的情况.
  实际上,原子性底层就是通过 undo log实现的。undo log主要记录了数据的逻辑变化,比如一条INSERT语句,对应一条DELETE 的 undo log,对于每个UPDATE语句,对应一条相反的UPDATE的undo log,这样在发生错误时,就能回滚到事务之前的数据状态.
  同时,undo log也是MVCC(多版本并发控制)实现的关键.
> 7、redo log.
> 为什么需要redo log,我们都知道,事务的四大特性里面有一个是持久性,具体来说就是只要事务提交成功,那么对数据库做的修改就被永久保存下来了,不可能因为任何原因再回到原来的状态.
> 因为Innodb是以页为单位进行磁盘交互的,而一个事务很可能只修改一个数据页里面的几个字节,这个时候将完整的数据页刷到磁盘的话,太浪费资源了!,是大小固定的,Innodb是引擎层实现的,采用循环写的.
> 因此mysql设计了redo log,具体来说就是只记录事务对数据页做了哪些修改，这样就能完美地解决性能问题了(相对而言文件更小并且是顺序IO).
> 8、数据库ACID的实现.
> 原子性：通过undolog来实现.
> 持久性：通过binlog、redolog来实现.
> 隔离性：通过(读写锁+MVCC)来实现.
> 一致性：MySQL通过原子性,持久性,隔离性最终实现（或者说定义）数据一致性.
> 附录MySQL的各种日志说明:
> 重做日志（redo log）
> 回滚日志（undo log）
> 归档日志（binlog）
> 错误日志（errorlog）
> 慢查询日志（slow query log）
> 一般查询日志（general log）
> 中继日志（relay log）