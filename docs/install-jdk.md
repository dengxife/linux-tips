# CentOS安装JDK
## 下载JDK安装文件

从Oracle官网上下载JDK，注意选择x64.rpm。使用FTP软件将下载的rpm文件上传到CentOS目录中，以下为oracle官网的8u221版本为例。

## 安装过程

使用命令 `cd` 到你放置rpm的目录中使用 `rpm -ivh jdk-8u221-linux-x64.rpm` 进行安装。

## 配置环境变量

使用命令 `vi /etc/profile` 打开配置文件，使用 `i` 进入编辑模式，使用键盘上的 `END` 按键快速的移动到最后，插入以下的内容。

```properties
export JAVA_HOME=/usr/java/jdk1.8.0_221-amd64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.ja
r
export PATH=$JAVA_HOME/bin:$PATH
```

按 `ESC` 退出编辑模式，使用命令 `:wq` 保存并退出编辑。

最后，使用 `source /etc/profile` 是修改的内容生效。

可以使用 `java -version` 验证jdk是否安装成功。