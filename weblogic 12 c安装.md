# weblogic 12 c安装

## 1、安装jdk

[https://www.oracle.com/technetwork/java/javase/downloads/index.html](https://www.oracle.com/technetwork/java/javase/downloads/index.html)  \#下载连接

```
rpm -ivh jdk-8u111-linux-x64.rpm  #安装jdk
配置环境变量
vi /etc/profile
export JAVA_HOME=/usr/java/jdk1.8.0_111
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
source /etc/profile  #激活配置
```

## 2、创建用户

```
创建用户组：groupadd weblogic

创建用户添加用户组weblogci指定家录：

useradd -g weblogic -m -d /home/lip weblogic

设置密码  passwd xxxxx

指定用户目录 chown -R weblogic:weblogic /home
```

## 3、安装

运行jar包进行安装

```
su - weblogic   #切换用户
#切换到jar包所在目录，运行以下命令进行安装
java -jar fmw_12.1.3.0.0_wls.jar
```

![](/image/1552616971622.png)

选择安装目录

![](/image/1552617066620.png)

![1552617100795](images\1552617100795.png)

检查先决条件

![1552617125549](image\1552617125549.png)

![1552617144849](image\1552617144849.png)

![1552617172199](image\1552617172199.png)

![1552617206142](image\1552617206142.png)

![1552617234557](image\1552617234557.png)

安装完成后配置域

![1552617287184](image\1552617287184.png)

![1552617327008](image\1552617327008.png)

设置密码：（一般为weblogic/weblogic12c）

![1552617439689](image\1552617439689.png)

![1552617419692](image\1552617419692.png)

![1552617476794](image\1552617476794.png)

![1552617493506](image\1552617493506.png)

![1552617508231](image\1552617508231.png)

到这一步。安装已经完成

![1552617948088](image\1552617948088.png)

## 4、验证

```
安装路径/data/weblogic/user_projects/domains/base_domain/bin

启动：startWebLogic.sh
后台启动：nohup./startWebLogic.sh
停止：stopWebLogic.sh
```

访问：[http://10.10.198.103:7001/console](http://10.10.198.103:7001/console)

账号和密码就是先前设置的

![1552618327715](image\1552618327715.png)

![1552618469400](image\1552618469400.png)

