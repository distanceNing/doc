查看当前sql执行状态：
````mysql
 show processlist;
````

拼接：

````mysql
update  t set field = concat(field,"xxx");
````

查看数据库隔离级别：

````
select @@transaction_isolation;

设置数据库隔离级别：
set session transaction isolation level REPEATABLE READ;
````

创建用户：

````mysql
CREATE USER 'test'@'%' IDENTIFIED BY 'testp';

skip-grant-tables

    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'rootp';
    FLUSH privileges;
````

