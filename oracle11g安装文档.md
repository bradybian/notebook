---
typora-copy-images-to: image
---

## 1、安装相关依赖

```
yum install -y binutils   compat-libcap1 compat-libstdc++   compat-libstdc++*.i686 gcc   gcc-c++   glibc*.1686 glibc  glibc-devel   glibc-devel ksh libgcc*.i686 libgcc  libstdc++   libstdc++*.i686 libstdc++-devel*  libstdc++-devel*.i686 libaio*   libaio*.i686 libaio-devel*  libaio-devel*.i686 make sysstat
#验证是否安装完成
rpm -q binutils compat-libstdc++ elfutils-libelf elfutils-libelf-devel elfutils-libelf-devel-static gcc gcc-c++ glibc glibc-common glibc-devel glibc-headers kernel-headers ksh libaio libaio-devel libgcc libgomp libstdc++ libstdc++-devel make sysstat unixODBC unixODBC-devel
```

## 2、验证udp 和tcp内核参数

```
cat /proc/sys/net/ipv4/ip_local_port_range
```

## 3、禁用透明页表

```
 cat /sys/kernel/mm/redhat_transparent_hugepage/enabled
 echo 'never' > /sys/kernel/mm/redhat_transparent_hugepage/enabled
```

## 4、设置主机名

```
rhel6   vi /etc/sysconfig/network 
rhel7   hostnamectl set-hostname oracle1
echo "oracle1" >> /etc/hosts
```

## 5、创建用户权限组

```
/usr/sbin/groupadd oinstall
/usr/sbin/groupadd -g 502 dba
/usr/sbin/groupadd -g 503 oper
/usr/sbin/groupadd -g 504 asmadmin
/usr/sbin/groupadd -g 506 asmdba
/usr/sbin/groupadd -g 505 asmoper
 /usr/sbin/useradd -u 502 -g oinstall -G dba,asmdba,oper oracle
 passwd oracle
```

## 6、确定limit

![](/image/1552540104482.png)

```
以安装所有者身份登录。

检查文件描述符设置的软限制和硬限制。确保结果在推荐范围内，例如：

$ ulimit -Sn 
1024 
$ ulimit -Hn 
65536
检查软限制和硬限制，以确定用户可用的进程数。确保结果在推荐范围内，例如：

$ ulimit -Su 
2047 
$ ulimit -Hu 
16384

检查堆栈设置的软限制。确保结果在推荐范围内，例如：

$ ulimit -Ss 
10240 
$ ulimit -Hs 
32768

修改limit设置
vi /etc/security/limits.conf
grid soft nproc 2047
grid hard nproc 16384
grid soft nofile 1024
grid hard nofile 65536
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
```

## 7、确定内核配置

```
vi /etc/sysctl.conf
fs.aio-max-nr = 1048576 
fs.file-max = 6815744 
kernel.shmall = 2097152 
kernel.shmmax = 4294967295 
kernel.shmmni = 4096 
kernel.sem = 250 32000 100 128 
net.ipv4.ip_local_port_range = 9000 65500 
net.core .rmem_default = 262144 
net.core.rmem_max = 4194304 
net.core.wmem_default = 262144 
net.core.wmem_max = 1048576


sysctl -p #使其生效
```

## 8、创建oracle基础目录

```
# mkdir -p /u01/app/oracle/
# mkdir -p /u01/app/oraInventory
# chown -R oracle:oinstall /u01/app
# chown -R oracle:oinstall /u01/app/oracle
# chmod -R 775 /u01/app/oracle
```

## 9、创建oracle用户的环境变量

```
su - oracle
vi ~/.bash_profile
umask 022
ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1
export ORACLE_SID=orcl
ORACLE_PID=orcl
export PATH=$PATH:$ORACLE_HOME/bin
export ORACLE_HOME=$ORACLE_HOME

source ~/.bash_profile #激活环境变量
```

## 10、创建xrdp环境

```
yum install xrdp tigervnc-server -y 
service start xrdp
在windows上使用远程桌面进行连接
```

## 11、解压oracle压缩包

```
 unzip linux.x64_11gR2_database_1of2.zip
 unzip linux.x64_11gR2_database_2of2.zip
```

## 12、进行安装

```
cd /data/database
./runInstaller
如果中文乱码：
export LANG=en-US
```

![](/image/1552554456588.png)

