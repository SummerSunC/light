# centos安装mysql


---

#安装依赖
yum groupinstall -y "Development Tools"
yum install -y cmake openssl-devel zlib-devel ncurses*

#下载源码
cd /tmp
wget -O mysql-5.6.21.tar.gz http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.21.tar.gz/from/http://cdn.mysql.com/

#添加用户组
groupadd mysql
useradd -g mysql mysql -s /sbin/nologin
mkdir /data/mysql
mkdir /usr/local/mysql

#编译
tar -zxvf  mysql-5.6.21.tar.gz
cd mysql-5.6.21
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql -DSYSCONFDIR=/etc -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DMYSQL_USER=mysql
make && make install

#初始化
#cp /usr/local/mysql/support-files/my-large.cnf /etc/my.cnf
cp -p /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/mysqld
chmod +x /etc/rc.d/init.d/mysqld
chkconfig --level 2345 mysqld on
chkconfig --add mysqld
vi /etc/my.cnf
[mysqld]
datadir=/data/mysql
socket=/tmp/mysql.sock
user=mysql


echo "basedir = /usr/local/mysql" >> /etc/rc.d/init.d/mysqld
echo "datadir = /data/mysql" >> /etc/rc.d/init.d/mysqld

chown -R mysql:mysql /data/mysql
chown -R mysql:mysql /usr/local/mysql

echo 'export PATH=/usr/local/mysql/bin:$PATH' > /etc/profile.d/mysql.sh && . /etc/profile.d/mysql.sh

./scripts/mysql_install_db --user=mysql --datadir=/data/mysql &
service mysqld start

#通过客户端访问并修改密码
mysql
mysql> select user,host,password from user;
+------+--------------+----------+
| user | host         | password |
+------+--------------+----------+
| root | localhost    |          |
| root | iz94w8205fvz |          |
| root | 127.0.0.1    |          |
| root | ::1          |          |
|      | iz94w8205fvz |          |
+------+--------------+----------+
5 rows in set (0.00 sec)

mysql> drop user ''@'iz94w8205fvz';
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host,password from user;
+------+--------------+----------+
| user | host         | password |
+------+--------------+----------+
| root | localhost    |          |
| root | iz94w8205fvz |          |
| root | 127.0.0.1    |          |
| root | ::1          |          |
+------+--------------+----------+
4 rows in set (0.00 sec)

mysql> update user set password = password('your_passwd') where user = 'root';
Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host,password from user;
+------+-----------+-------------------------------------------+
| user | host      | password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | *A69A12862E2F4D62CC4A665AFFC351CBCE992D30 |
| root | 127.0.0.1 | *A69A12862E2F4D62CC4A665AFFC351CBCE992D30 |
| root | %         | *A69A12862E2F4D62CC4A665AFFC351CBCE992D30 |
+------+-----------+-------------------------------------------+
3 rows in set (0.00 sec)




