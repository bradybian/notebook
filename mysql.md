# mysql8.0 二进制安装

本次安装使用的时centos7.4的操作系统

cat /etc/redhat-release   \#查看操作系统版本

## 1、下载mysql8.0二进制压缩包

[https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)     \#mysql下载地址![](/image/import.png)选择相对应的操作系统及版本。注册登陆并下载

## 2、安装mysql

### 1、使用ftp工具将软件包上传到服务器

在这里使用了rz工具![](/image/rz.png)

![](/image/rz1.png)

### 2、解压并安装

```
tar -xzvf  mysql-8.0.13-el7-x86_64.tar.gz  #解压
mv  mysql-8.0.13-el7-x86_64  /usr/local/mysql   #将解压后的文件移动到/usr/local下面
```

使用tree列出解压后的文档

tree -d /usr/local/mysql

![](/image/tree.png)

安装依赖文件

mysql依赖libaio 这个库文件

我这里是centos，所以使用yum进行安装

```
yum install libaio
groupadd mysql   #创建组
useradd -r -g mysql -s /bin/false mysql #创建用户
```

创建mysql配置文件

```
vi /etc/my.cnf

[client]
port = 3306
socket = /var/run/mysqld/mysql.sock

[mysqld]

server-id = 1
port = 3306
skip-name-resolve
basedir = /usr/local/mysql                                #安装目录
datadir = /var/lib/mysql/data                             #数据目录
tmpdir  = /tmp
pid-file = /var/lib/mysql/data/mysql.pid


socket = /var/run/mysqld/mysql.sock
max_connections = 400                                     #最大连接数


[mysqld_safe]
log_error = /var/log/mysql/error.log
pid-file = /var/lib/mysql/data/mysql.pid
```

创建对应目录

```
mkdir -pv /var/lib/mysql/data  #创建数据目录
mkdir -pv /var/log/mysql/  #创建日志目录
mkdir -pv /var/run/mysqld   #创建socket目录
chown mysql:mysql /var/log/mysql/
chown mysql:mysql /var/run/mysqld
```

进行初始化

mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql   \#使用刚才的配置文件进行初始化

在输出的日志文件中会有默认密码产生

```
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/   #
service mysql.server start #进行启动
```

使用初始化的密码进行登陆

```
mysql -uroot -p"默认密码"
alter user root identified by "123456"  #修改默认密码
```

最后，从mysql5.7起，mysql启用了密码验证插件。在8.0版本做了相应修改，5.7的设置，在8.0无法使用.

可以在my.cnf配置文件添加

```
validate_password.policy = LOW #降低密码强度
validate_password.length = 6  #设置密码长度
```

或者使用mysql\_secure\_installation  进行调整

# mysql8.0高可用双主热备

## 1、准备

本次架构为keepalived + mysql 主主同步复制关系，保证两台mysql的数据一致性，实现双主热备，利用keepalived提供虚拟ip,并且使用keepalive进行故障检测，实现mysql 故障时，能够进行自动切换。

操作系统版本：centos7.4

虚拟ip:  vip 192.168.160.177

| 主机 | ip | 应用 |
| --- | --- | --- |
| docker1 | 192.168.160.165 | mysql8.0   keepalived |
| docker2 | 192.168.160.160 | mysql8.0   keepalived |

## 2、进行配置

1、按照第一章的方式，在两台机器上进行安装相同版本的mysql8.0

2、对两台机器上的my.cnf进行配置

work1的配置文件

```
#主标服务标识号,必需唯一

server-id = 1

#因为MYSQL是基于二进制的日志来做同步的,每个日志文件大小为 1G

log-bin=/var/log/mysql/mysql-bin.log

#要同步的库名

binlog-do-db = test

#不记录日志的库,即不需要同步的库

binlog-ignore-db=mysql

#用从属服务器上的日志功能

log-slave-updates

#经过1日志写操作就把日志文件写入硬盘一次(对日志信息进行一次同步)。n=1是最安全的做法，但效率最低。默认设置是n=0。

sync_binlog=1

# auto_increment，控制自增列AUTO_INCREMENT的行为

#用于MASTER-MASTER之间的复制，防止出现重复值,

#auto_increment_increment=n有多少台服务器，n就设置为多少,

#auto_increment_offset＝1设置步长,这里设置为1,这样Master的auto_increment字段产生的数值是:1, 3, #5, 7, …等奇数ID

auto_increment_offset=1

auto_increment_increment=2

#进行镜像处理的数据库

replicate-do-db = test

#不进行镜像处理的数据库

replicate-ignore-db= mysql
```

work2配置文件

```
#主标服务标识号,必需唯一

server-id = 2

#因为MYSQL是基于二进制的日志来做同步的,每个日志文件大小为 1G

log-bin=/var/log/mysql/mysql-bin.log

#要同步的库名

binlog-do-db = test

#不记录日志的库,即不需要同步的库

binlog-ignore-db=mysql

#用从属服务器上的日志功能

log-slave-updates

#经过1日志写操作就把日志文件写入硬盘一次(对日志信息进行一次同步)。n=1是最安全的做法，但效率最低。默认设置是n=0。

sync_binlog=1

# auto_increment，控制自增列AUTO_INCREMENT的行为

#用于MASTER-MASTER之间的复制，防止出现重复值,

#auto_increment_increment=n有多少台服务器，n就设置为多少,

#auto_increment_offset＝2设置步长,这里设置为2,这样Master的auto_increment字段产生的数值是:2, 4, #6, 8, …等奇数ID

auto_increment_offset=2

auto_increment_increment=2

#进行镜像处理的数据库

replicate-do-db = test

#不进行镜像处理的数据库

replicate-ignore-db= mysql
```

docker1的my.cnf

```
[client]
port = 3306
socket = /var/run/mysqld/mysql.sock

[mysqld]

port = 3306
skip-name-resolve
basedir = /usr/local/mysql
datadir = /var/lib/mysql/data
tmpdir  = /tmp
pid-file = /var/lib/mysql/data/mysql.pid

socket = /var/run/mysqld/mysql.sock

max_connections = 400
validate_password.policy = LOW #降低密码强度
validate_password.length = 6 #设置密码长度


#master/master配置


server-id = 1
log-bin=/var/lib/mysql/mysql-bin
log-slave-updates
sync_binlog = 1
auto_increment_offset=1
auto_increment_increment=2




[mysqld_safe]
log_error = /var/log/mysql/error.log
pid-file = /var/lib/mysql/data/mysql.pid
```

docker2的my.cnf

```
[client]
port = 3306
socket = /var/run/mysqld/mysql.sock
default-character-set=utf8

[mysqld]

port = 3306
skip-name-resolve
basedir = /usr/local/mysql #安装目录
datadir = /var/lib/mysql/data #数据目录
tmpdir = /tmp
pid-file = /var/lib/mysql/data/mysql.pid
validate_password.policy = LOW #降低密码强度
validate_password.length = 6  #设置密码长度


socket = /var/run/mysqld/mysql.sock
max_connections = 400 #最大连接数


#master/master配置

server-id = 2
log-bin = /var/lib/mysql/mysql-bin
log-slave-updates
sync_binlog = 1
auto_increment_offset=2
auto_increment_increment=2


[mysqld_safe]
log_error = /var/log/mysql/error.log
pid-file = /var/lib/mysql/data/mysql.pid
```

重启服务器

server mysql.server restart

由于是使用二进制安装的，在启动过程中因权限原因报错

![](/image/1544776527793.png)

```
chown mysql:mysql /usr/lib/mysql  #进行授权
```

之后启动成功

## 3、进行相互授权

登陆后，创建复制账户（repl）并授权

```
CREATE USER 'repl'@'%' IDENTIFIED BY '123456';  #创建用户
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';  #授权
```

查看二进制日志的文件名和位置

docker1

```
flush logs #刷新日志
show master status \G;
```

![](/image/1544777226459.png)

docker2

```
flush logs #刷新日志
show master status \G;
```

![](/image/1544777269385.png)

通过sql语句在docker2上指定master为docker1

```
CHANGE MASTER TO
        MASTER_HOST='192.168.160.165',
        MASTER_USER='repl',
        MASTER_PASSWORD='123456',
        MASTER_LOG_FILE='mysql-bin.000002',
        MASTER_LOG_POS=155;
```

通过sql语句在docker1上指定master为docker2

```
CHANGE MASTER TO
        MASTER_HOST='192.168.160.160',
        MASTER_USER='repl',
        MASTER_PASSWORD='123456',
        MASTER_LOG_FILE='mysql-bin.000002',
        MASTER_LOG_POS=155;
```

然后在两台机器上启动

```
start slave #启动
show slave status\G;  #检查状态
```

出现了错误

![](/image/1544778095929.png)

使用flush privileges,后，重启了一下，解决问题，

可能的原因是创建用户并授权后，没有刷新授权表，导致远程连接没有权限。

## 4、进行测试

在docker1上测试

创建数据库和表，进行查找

```
create database Student;  #创建库
CREATE TABLE Persons
(
Id_P int,
LastName varchar(255),
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)                   #创建表
```

如果两边都有相同的表创建出来则说明成功

在docker2上测试

创建数据库和表，进行查找

```
create database School
CREATE TABLE Persons
(
Name int,
Age int,
Sex varchar(255)
)  

insert into Persons VALUES ('111','33','男')
```

如果两边都有相同的表创建出来则说明成功

