# CentOS安装Maven和配置问题的解决办法
这次我在MAc的虚拟机Parallels中[安装了Centos](software-sources.md)，并且[安装了jdk](install-jdk.md)和maven，在安装maven的过程中遇到了一个jdk指向的问题排查了一天，过程中也学习了不少linux的知识，这里一并做下记录。

版本定义如下：

CentOS：7.6.1810

JDK：8u221（oracle官网提供的1.8最后一个版本）

maven：3.6.1

## 下载和安装MAVEN

从Apache[官网](http://maven.apache.org/download.cgi)下载最新版本的maven，找到Linux版本的[apache-maven-3.6.1-bin.tar.gz](http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz)下载后通过FTP上传到CentOS的目录中。使用命令 `cd` 到你放置的目录中使用 `tar -zxvf apache-maven-3.6.1-bin.tar.gz` 进行解压后会在当前目录下得到一个解压的 `apache-maven-3.6.1` 的目录。

接下来配置maven的环境变量，使用命令 `vi /etc/profile` 进行编辑，如下。

```properties
export JAVA_HOME=/usr/java/jdk1.8.0_221-amd64
export MAVEN_HOME=/root/apache-maven-3.6.1
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.ja
r
export PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH
```

到这里结束，使用 `mvn -v` 如果现实maven版本信息表示安装成功。

## 修改仓库源为阿里云

在[阿里云仓库的网站](https://maven.aliyun.com/mvn/view)选择Repositoroes View查看提供的源信息。

使用 `cd` 命令进入到maven解压目录的conf目录下，编辑settings.xml 文件。

在 `<mirrors>` 标签下插入 `<mirror>` 。

```xml
<mirror>
	<id>aliyunmaven</id>
	<mirrorOf>public</mirrorOf>
	<name>阿里云公共仓库</name>
	<url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

按键 `Ecs` 退出编辑模式后 `:wq` 保存退出即可。

## 关于 `mvn -v` 显示 `NB: JAVA_HOME should point to a JDK not a JRE` 的问题

```
[root@centos-linux ~]# mvn -v
The JAVA_HOME environment variable is not defined correctly
This environment variable is needed to run this program
NB: JAVA_HOME should point to a JDK not a JRE
```

从信息来看是JDK的环境变量指向配置得有问题，它指向了JRE（JAVA运行环境）而不是JDK。

### 排查一:命令是否指向jdk的bin目录下，而不是jre的bin目录下

首先要排查JDK是否安装正确，使用 `java -version` 和 `javac -version` 命令查看是否都能正确输出版本信息。

使用 `echo $JAVA_HOME` 命令查看在 `/etc/profile` 中配置的java变量是否正确， `JAVA_HOME` 应该指向JDK的安装目录 `/usr/java/jdk1.8.0_221-amd64`。

使用 `which java` 命令查看java命令的指向。

```
[root@centos-linux ~]# which java
/usr/bin/java
[root@centos-linux ~]# ll /usr/bin/java
lrwxrwxrwx. 1 root root 22 8月  19 16:27 /usr/bin/java -> /etc/alternatives/java
[root@centos-linux ~]# ll /etc/alternatives/java
lrwxrwxrwx. 1 root root 37 8月  19 16:27 /etc/alternatives/java -> /usr/java/jdk1.8.0_221-amd64/jre/bin/java
[root@centos-linux ~]# ll /usr/java/jdk1.8.0_221-amd64/jre/bin/java
-rwxr-xr-x. 1 root root 8534 7月   4 19:38 /usr/java/jdk1.8.0_221-amd64/jre/bin/java
```

**发现我的java命令是来自于jre的bin目录中的java命令**，使用同样的方法查看一下`javac` 命令会发现是来自于 `/usr/java/jdk1.8.0_221-amd64/bin` 目录中。

所以第一步要将 `/usr/bin/java` 的连接指向到正确的jdk目录中的bin下的java命令。

使用 `alternatives --config java`  命令列出 `java` 命令可以连接的命令路径。会是以下的样子。

```
*+ 1           /usr/java/jdk1.8.0_221-amd64/jre/bin/java
```

由于我的可用列表中只有一项，所以我需要将 `/usr/java/jdk1.8.0_211-amd64/bin/java` 添加到列表中。

使用 `alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_211-amd64/bin/java 100`后，再次使用 `alternatives --config java` 会看到多了一项。

```
共有 2 个提供“java”的程序。
  选项    命令
-----------------------------------------------
*  1           /usr/java/jdk1.8.0_221-amd64/jre/bin/java
 + 2           /usr/java/jdk1.8.0_221-amd64/bin/java
按 Enter 保留当前选项[+]，或者键入选项编号：
```

输入2，按`Enter` 保存，即可。再次查看确认java命令是指向jdk的bin目录下。

```
[root@centos-linux ~]# ll /usr/bin/java                                                                                
lrwxrwxrwx. 1 root root 22 8月  19 16:27 /usr/bin/java -> /etc/alternatives/java
[root@centos-linux ~]# ll /etc/alternatives/java
lrwxrwxrwx. 1 root root 37 8月  19 16:27 /etc/alternatives/java -> /usr/java/jdk1.8.0_221-amd64/bin/java
```

### 排查二:排查JAVA_HOME的路径配置是否正确

通过排查一中的都确认正确后，使用 `mvn -v` 如果还是会出现以上报错的话。

通过 `echo $JAVA_HOME` 命令输出$JAVA_HOME变量的值 `/usr/java/jdk1.8.0_211-amd64` 复制，拼接 `/bin/java` 。

使用拼接后的结对路径命令 `/usr/java/jdk1.8.0_211-amd64/bin/java -version`  ，发现输出 `-bash: cd: /usr/java/jdk1.8.0_221-amd64/bin/java: 不是目录` 的错误信息。表示 `JAVA_HOME` 这个变量是有问题的（肉眼看不出的问题）。原因是我通过以下的方式获取的绝对路径命令又是可以正确输出版本信息的。

```
[root@centos-linux ~]# which java
/usr/bin/java
[root@centos-linux ~]# ll /usr/bin/java
lrwxrwxrwx. 1 root root 22 8月  19 16:27 /usr/bin/java -> /etc/alternatives/java
[root@centos-linux ~]# ll /etc/alternatives/java
lrwxrwxrwx. 1 root root 37 8月  19 16:27 /etc/alternatives/java -> /usr/java/jdk1.8.0_221-amd64/bin/java
```

通过以上获得的到的 `java` 命令路径又是正确的，于是我确信环境变量中的 `$JAVA_HOME` 应该是字符编码有问题。

修改 `/etc/profile` 文件中的 `$JAVA_HOME` 变量的值，直接从上面的输出中将 `/usr/java/jdk1.8.0_221-amd64` 这一段复制粘贴过来。使用 `source /etc/profile`  使修改生效。

至此，`mvn -v` 可以正常输出版本信息。

```
[root@centos-linux ~]# mvn -version
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /root/apache-maven-3.6.1
Java version: 1.8.0_221, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_221-amd64/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.el7.x86_64", arch: "amd64", family: "unix"
```

