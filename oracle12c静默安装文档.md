---

---

# 基础环境

操作系统:centos7.5

## 1、安装相关依赖

```
yum install -y binutils compat-libcap1 compat-libstdc++-33 compat-libstdc++-33*.i686 glibc glibc*.i686 glibc-devel glibc-devel*.i686 ksh libaio libaio*.i686 libaio-devel libaio-devel*.i686 libX11 libX11*.i686 libXau libXau*.i686 libXi libXi*.i686 libXtst libXtst*.i686 libgcc libgcc*.i686 libstdc++ libstdc++*.i686 libstdc++-devel libstdc++-devel*.i686  libxcb libxcb*.i686 make nfs-utils net-tools smartmontools sysstat unixODBC unixODBC-devel gcc gcc-c++ libXext libXext*.i686 zlib-devel zlib-devel*.i686 unzip

#验证是否安装完成
rpm -q binutils compat-libcap1 compat-libstdc++-33 compat-libstdc++-33*.i686 glibc glibc*.i686 glibc-devel glibc-devel*.i686 ksh libaio libaio*.i686 libaio-devel libaio-devel*.i686 libX11 libX11*.i686 libXau libXau*.i686 libXi libXi*.i686 libXtst libXtst*.i686 libgcc libgcc*.i686 libstdc++ libstdc++*.i686 libstdc++-devel libstdc++-devel*.i686  libxcb libxcb*.i686 make nfs-utils net-tools smartmontools sysstat unixODBC unixODBC-devel gcc gcc-c++ libXext libXext*.i686 zlib-devel zlib-devel*.i686 unzip
```

## 2、验证udp 和tcp内核参数

```
cat /proc/sys/net/ipv4/ip_local_port_range
```

## 3、禁用透明页表

```
 cat /sys/kernel/mm/transparent_hugepage/enabled   #centos用这个
 echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled
 cat /sys/kernel/mm/redhat_transparent_hugepage/enabled  #red-hat用这个
 echo 'never' > /sys/kernel/mm/redhat_transparent_hugepage/enabled
```

## 4、设置主机名

```
rhel6   vi /etc/sysconfig/network 
rhel7   hostnamectl set-hostname oracle1
echo "oracle1" >> /etc/hosts
#设置selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config  #永久生效
getenforce #查看selinux状态
setenforce 0 #暂时关闭selinux ,立即生效
#设置防火墙
systemctl stop firewalld && systemctl disable firewalld
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
net.core.rmem_default = 262144 
net.core.rmem_max = 4194304 
net.core.wmem_default = 262144 
net.core.wmem_max = 1048576


sysctl -p #使其生效
```

## 8、创建oracle基础目录

```
# mkdir -pv /data/u01/app/oracle/
# mkdir -pv /data/u01/app/oraInventory
# mkdir -pv /data/u01/app/oracle/product/12.2.0.1/dbhome_1
# chown -R oracle:oinstall /data/u01/app/oraInventory
# chown -R oracle:oinstall /data/u01/app/oracle
# chmod -R 775 /data/u01/app/oracle
```

## 9、创建oracle用户的环境变量

```
su - oracle
vi ~/.bash_profile
umask 022
ORACLE_BASE=/data/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/12.2.0.1/dbhome_1
export ORACLE_SID=orcl
ORACLE_PID=orcl
export PATH=$PATH:$ORACLE_HOME/bin
export ORACLE_HOME=$ORACLE_HOME

source ~/.bash_profile #激活环境变量
```

## 10、创建xrdp环境

```
yum install xrdp tigervnc-server -y 
service start xrdp   #rhel6
systemctl start xrdp #rhel7
在windows上使用oracle用户进行远程桌面进行连接
```

## 11、解压oracle压缩包

```
 使用sftp工具将软件包（winscp）上传到服务器上
 unzip linux.x64_11gR2_database_1of2.zip
 unzip linux.x64_11gR2_database_2of2.zip
 chown -R oracle:oinstall database/
```

## 12、进行安装

```
cd /data/database
./runInstaller
如果中文乱码：
export LANG=en-US
```

## 13、静默安装配置文件修改

```
#备份原始文件
cp /data/database/response/db_install.rsp /data/database/response/db_install.rsp.bak 
#修改配置文件
vi /data/database/response/db_install.rsp

ORACLE_HOSTNAME=oracle1
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/data/u01/app/oraInventory
SELECTED_LANGUAGES=en
ORACLE_HOME=/data/u01/app/oracle/product/12.2.0.1/dbhome_1
ORACLE_BASE=/data/u01/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=oinstall
oracle.install.db.BACKUPDBA_GROUP=dba
oracle.install.db.DGDBA_GROUP=dba
oracle.install.db.KMDBA_GROUP=dba
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE 
oracle.install.db.config.starterdb.globalDBName=orcl
oracle.install.db.config.starterdb.SID=orcl
oracle.install.db.config.starterdb.characterSet=AL32UTF8
oracle.install.db.config.starterdb.password.ALL=admin#123
oracle.install.db.config.starterdb.emAdminUser=oracle
oracle.install.db.config.starterdb.emAdminPassword=oracle12c
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
DECLINE_SECURITY_UPDATES=true

```

## 14、安装oracle数据库

```
./runInstaller -force -silent -noconfig -ignorePrereq -ignoreSysPreReqs -responseFile /data/database/response/db_install.rsp

```

![](/image/1552983333631.png)

![](/image/1552983391946.png)

## 15、配置默认监听

```
netca -silent -responsefile /data/database/response/netca.rsp
```

![](/image/1552983777182.png)

验证端口是否打开

```
netstat -tlnp | grep 1521
```

![](/image/1552983820667.png)

## 16、静默创建数据库实例修改dbca.rsp文件

```
 vi /data/database/response/dbca.rsp
 

[CREATEDATABASE]
GDBNAME = "orcl"
SID = "orcl"
TEMPLATENAME = "General_Purpose.dbc"
SYSPASSWORD = "admin#123"
SYSTEMPASSWORD = "admin#123"
EMCONFIGURATION = "DBEXPRESS"
EMEXPRESSPORT = "5520"
DBSNMPPASSWORD = "admin#123"
CHARACTERSET = "AL32UTF8"
NATIONALCHARACTERSET= "AL32UTF8"
```

```
/data/u01/app/oracle/product/12.2.0.1/dbhome_1/bin/dbca -silent -responseFile /data/database/response/dbca.rsp
```

![](/image/1552986344535.png)

![](/image/1552986370570.png)

## 17、测试

```
#连接数据库
su - oracle
sqlplus / as sysdba
#查询库名

select name,dbid from v$database；
```

![](/image/1552986614005.png)