title: MySql常用命令总结
date: 2015-8-25 22:30:49
tags: MySql
---
1:使用SHOW语句找出在服务器上当前存在什么数据库：
```
mysql> SHOW DATABASES;
```
2:2、创建一个数据库MYSQLDATA
```
mysql> CREATE DATABASE MYSQLDATA;
```
3:选择你所创建的数据库
```
mysql> USE MYSQLDATA;
```
 (按回车键出现Database changed 时说明操作成功！)
4:查看现在的数据库中存在什么表
```
mysql> SHOW TABLES;
```
5:创建一个数据库表
```
mysql> CREATE TABLE MYTABLE (name VARCHAR(20), sex CHAR(1));
```
6:显示表的结构：
```
mysql> DESCRIBE MYTABLE;
```
7:往表中加入记录
```
mysql> insert into MYTABLE values (”hyq”,”M”);
```
8:用文本方式将数据装入数据库表中（例如D:/mysql.txt）
```
mysql> LOAD DATA LOCAL INFILE “D:/mysql.txt” INTO TABLE MYTABLE;
```
9:导入.sql文件命令（例如D:/mysql.sql）
```
mysql>use database;
mysql>source d:/mysql.sql;
```
10:删除表
```
mysql>drop TABLE MYTABLE;
```
11:清空表
```
mysql>delete from MYTABLE;
```
12:更新表中数据
```
mysql>update MYTABLE set sex=”f” where name=’hyq’;
```

以下是无意中在网络看到的使用MySql的管理心得,
在windows中MySql以服务形式存在，在使用前应确保此服务已经启动，未启动可用net start mysql命令启动。而Linux中启动时可用“/etc/rc.d/init.d/mysqld start”命令，注意启动者应具有管理员权限。
刚安装好的MySql包含一个含空密码的root帐户和一个匿名帐户，这是很大的安全隐患，对于一些重要的应用我们应将安全性尽可能提高，在这里应把匿名帐户删除、 root帐户设置密码，可用如下命令进行：
use mysql;
delete from User where User=”";
update User set Password=PASSWORD(’newpassword’) where User=’root’;
如果要对用户所用的登录终端进行限制，可以更新User表中相应用户的Host字段，在进行了以上更改后应重新启动数据库服务，此时登录时可用如下类似命令：
mysql -uroot -p;
mysql -uroot -pnewpassword;
mysql mydb -uroot -p;
mysql mydb -uroot -pnewpassword;
上面命令参数是常用参数的一部分，详细情况可参考文档。此处的mydb是要登录的数据库的名称。
在 进行开发和实际应用中，用户不应该只用root用户进行连接数据库，虽然使用root用户进行测试时很方便，但会给系统带来重大安全隐患，也不利于管理技 术的提高。我们给一个应用中使用的用户赋予最恰当的数据库权限。如一个只进行数据插入的用户不应赋予其删除数据的权限。MySql的用户管理是通过 User表来实现的，添加新用户常用的方法有两个，一是在User表插入相应的数据行，同时设置相应的权限；二是通过GRANT命令创建具有某种权限的用 户。其中GRANT的常用用法如下：
grant all on mydb.* to NewUserName@HostName identified by “password” ;
grant usage on *.* to NewUserName@HostName identified by “password”;
grant select,insert,update on mydb.* to NewUserName@HostName identified by “password”;
grant update,delete on mydb.TestTable to NewUserName@HostName identified by “password”;
若 要给此用户赋予他在相应对象上的权限的管理能力，可在GRANT后面添加WITH GRANT OPTION选项。而对于用插入User表添加的用户，Password字段应用PASSWORD 函数进行更新加密，以防不轨之人窃看密码。对于那些已经不用的用户应给予清除，权限过界的用户应及时回收权限，回收权限可以通过更新User表相应字段， 也可以使用REVOKE操作。
下面给出本人从其它资料(www.cn-java.com)获得的对常用权限的解释：
全局管理权限：
FILE: 在MySQL服务器上读写文件。
PROCESS: 显示或杀死属于其它用户的服务线程。
RELOAD: 重载访问控制表，刷新日志等。
SHUTDOWN: 关闭MySQL服务。
数据库/数据表/数据列权限：
ALTER: 修改已存在的数据表(例如增加/删除列)和索引。
CREATE: 建立新的数据库或数据表。
DELETE: 删除表的记录。
DROP: 删除数据表或数据库。
INDEX: 建立或删除索引。
INSERT: 增加表的记录。
SELECT: 显示/搜索表的记录。
UPDATE: 修改表中已存在的记录。
特别的权限：
ALL: 允许做任何事(和root一样)。
USAGE: 只允许登录–其它什么也不允许做。