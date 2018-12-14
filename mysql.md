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









































































