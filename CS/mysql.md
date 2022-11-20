# 1、update语句和select语句执行

## update

1、update语句执行时，若对应的数据页在内存中，则直接修改内存中的数据页

​		若对应的数据页不在内存中，则在内存的change buffer 中写下修改的内容

2、将第一步操作写入redo log（磁盘），update执行完成

3、后台操作：将内存中change buffer中的数据持久化到磁盘的change buffer中；定期将change buffer中的修改记录merge到数据页中

## select

1、若对应的数据页在内存中，直接读取内存返回结果

2、若内存中没有对应数据页，则将对应数据页读入内存，与change buffer中的记录做merge操作后返回结果



change buffer：

​			不适合update后马上读的表；因频繁触发merge，性能更差

​			不可用于唯一索引；唯一索引插入时需要先判断是否重复，不可直接写入change buffer

​			既在内存中，也在磁盘中有一份备份；

​			降低了对磁盘的随机读；

redo log

​			存在于磁盘中；

​			将写入磁盘操作由随机写转化为顺序写；



## 1、如何控制长事务

### 开发

确认autocommit = 0 ，开启 general_log 确认

确认只读事务数量是否过多

设置 MAX_EXCUTION_TIME 控制SQL最长执行时间

### DBA

监控 information_schema.Innodb_trx 表，设置长事务阈值

分析general_log

innodb_undo_tablespaces 设置为2

​	—设定表空间个数，每个默认10M



## 2、事务视图隔离性



事务视图中，除自己的修改永远可见外：

1、未commit的版本不可见

2、视图创建后commit，不可见；视图创建前提交，可见



注：

update语句是先读后写，此处为当前读，永远读取最新版本

select语句加锁后，同样也是当前读

​	select k from T where id = 1 lock in share mode;-读锁

​	select k from T where id = 1 for update;-写锁

当前读必须加写锁，若此时有其他事务占用写锁，必须等到释放这个锁，才可以完成当前读
