# mysql用户操作
---
## 参考资料
[The MySQL Access Privilege System](http://dev.mysql.com/doc/mysql-security-excerpt/5.6/en/privilege-system.html)

[Account Management Statements](http://dev.mysql.com/doc/refman/5.6/en/account-management-sql.html)

[数据库权限表](http://www.cnblogs.com/Richardzhu/p/3318595.html)

## 小记

### 注意事项
设置远程用户对单个指定数据库的访问权限，如果用[user]@'%'，则事实上远程用户只有可数据库的基本访问权限[select,update,insert,delete]而没有[drop,alter等权限].

如果想使远程用户拥有这些表操作权限则必须指定远程地址 如[user]@"192.168.135.1"或将权限应用到所有database中
```
#有表操作权限（权限适用范围为*.*）
mysql> show grants for www@"%";
+-------------------------------------------------------------------------------------------------------------+
| Grants for www@%                                                                                            |
+-------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'www'@'%' IDENTIFIED BY PASSWORD '*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19' |
| GRANT ALL PRIVILEGES ON `fxt`.* TO 'www'@'%'                                                                |
+-------------------------------------------------------------------------------------------------------------+

# 有表操作权限(权限使用范围为fxt.*,但是指定了远程主机) 
mysql> show grants for www@"192.168.135.1";
+----------------------------------------------------------------------------------------------------------------+
| Grants for www@192.168.135.1                                                                                   |
+----------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'www'@'192.168.135.1' IDENTIFIED BY PASSWORD '*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19' |
| GRANT ALL PRIVILEGES ON `fxt`.* TO 'www'@'192.168.135.1'                                                       |
+----------------------------------------------------------------------------------------------------------------+

# 有表操作权限（本地主机访问，无限制）
mysql> show grants for www@"localhost"
    -> ;
+------------------------------------------------------------------------------------------------------------+
| Grants for www@localhost                                                                                   |
+------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'www'@'localhost' IDENTIFIED BY PASSWORD '*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19' |
| GRANT ALL PRIVILEGES ON `fxt`.* TO 'www'@'localhost'                                                       |
+------------------------------------------------------------------------------------------------------------+

#无表操作权限，只有列操作权限（适用所有远程用户，但是未指定主机）
mysql> show grants for www@"%";
+----------------------------------------------------------------------------------------------------+
| Grants for www@%                                                                                   |
+----------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'www'@'%' IDENTIFIED BY PASSWORD '*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19' |
| GRANT ALL PRIVILEGES ON `fxt`.* TO 'www'@'%'                                                       |
+----------------------------------------------------------------------------------------------------+
```

## 查找用户
使用root用户登录mysql
``` sql
select host，user from mysql.user 
```
## 创建用户
``` sql
insert into mysql.user(Host,User,Password) values("localhost","dev",password("1234"));
```
-  **此处的"localhost"，是指该用户只能在本地登录，不能在另外一台机器上远程登录。如果想远程登录的话，将"localhost"改为"%"，表示在任何一台电脑上都可以登录。也可以指定某台机器可以远程登录。**

## 更新用户
``` sql
update mysql.user set password = password("password") where user = 'www'
```

## 分配权限
``` sql
# grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码";　
```
### 授权用户拥有某个数据库的所有权限
``` sql
grant all privileges on testDB.* to test@localhost identified by '1234';
#刷新数据表
flush privileges; 
```
### 指定部分权限给一用户
``` sql
grant select,update on testDB.* to test@localhost identified by '1234';
flush privileges; 
```
### 授权用户拥有所有数据库的某些权限
```
grant select,delete,update,create,drop on *.* to test@"%" identified by "1234";
```
### 回收用户权限
```
revoke all on fxt.* from www@"%";
```
## 删除用户
``` sql
Delete FROM user Where User='test' and Host='localhost';
flush privileges;
drop database testDB; #删除用户的数据库

#删除用户及权限
drop user 用户名@'%';
drop user 用户名@ localhost; 
```
## 授权权限
如果加上WITH GRANT OPTION 则用户就可以给其他用户赋权
```
grant all privileges on *.* to www@"%" identified by "password" WITH GRANT OPTION;
```