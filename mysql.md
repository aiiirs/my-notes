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



​	

