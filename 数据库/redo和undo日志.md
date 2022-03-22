### Binlog:

数据库逻辑日志，记录了数据库的更新sql操作。数据库server层的日志。

通过数据库备份 + binlog 恢复数据库。

* binlog的写入是不能被打断的，线程级别的cache：

​		如果一个事务的binlog被拆开的时候，在备库执行就会被当做多个事务分段自行，这样破坏了原子性，是有问题的。

### Redo日志：

`redo`日志本质上只是记录了一下事务对数据库做了哪些修改，记录了某个数据页上做了那些修改。innodb引擎特有的日志。

持久性的提现。

创建表时没有为某个表显式的定义主键，并且表中也没有定义`Unique`键，那么`InnoDB`会自动的为表添加一个称之为`row_id`的隐藏列作为主键。

* 事务执行中间过程的 redo log 也是直接写在 redo log buffer 中的，这些 redo log 也会被后台线程一起持久化到磁盘。也就是说，一个没有提交的事务的 redo log，也是可能已经持久化到磁盘的。

### Undo日志：

​	当前sql操作所对应的回滚操作。

#### 两阶段提交：

<img src="..\picture\mysqllogcommit.webp" style="zoom:50%;" />

```
           prepare redo log
           
             bin log
            
             commit 
           
```

