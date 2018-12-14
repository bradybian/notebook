# mysql8.0 二进制安装

本次安装使用的时centos7.4的操作系统

cat /etc/redhat-release   \#查看操作系统版本

## 1、下载mysql8.0二进制压缩包

[https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)     \#mysql下载地址![](/assets/import.png)选择相对应的操作系统及版本。注册登陆并下载

## 2、安装mysql

### 1、使用ftp工具将软件包上传到服务器

在这里使用了rz工具![](/assets/rz.png)

![](/assets/rz1.png)

### 2、解压并安装

```
tar -xzvf  mysql-8.0.13-el7-x86_64.tar.gz  #解压
mv  mysql-8.0.13-el7-x86_64  /usr/local/mysql   #将解压后的文件移动到/usr/local下面
```

使用tree列出解压后的文档

tree -d /usr/local/mysql

![](/assets/tree.png)

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
default-character-set=utf8

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











