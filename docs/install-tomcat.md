# CentOS安装Git及配置SSH秘钥
## 下载Git安装文件

从[Tomcat的官网](https://tomcat.apache.org/download-80.cgi)上下载安装包，在下方选择对应的tar.gz包。使用FTP软件将下载的tar.gz文件上传到CentOS目录中，以下为apache-tomcat-8.5.45.tar.gz版本为例。

## 安装git

使用命令 `tar -zxvf apache-tomcat-8.5.45.tar.gz `  解压安装包即可。

## 配置tomcat的编码

完成解压过程后，使用 `cd` 命令进入解压好的 apache-tomcat-8.5.45/conf 目录中。

使用 `vi` 命令编辑 `server.xml` 文件，修改如下位置添加 `URIEncoding="utf-8"`。

```
<Connector port="8080" protocol="HTTP/1.1"
	connectionTimeout="20000"
	redirectPort="8443" 
  URIEncoding="utf-8" />
```

## 启动与停止服务

在 tomcat 目录下的 bin 目录下，使用 `./startup.sh` 启动tomcat服务。使用 `./shutdown.sh` 停止tomcat服务。

在客户端浏览器访问 `http://ip:8080` 可以打开tomcat主页。

