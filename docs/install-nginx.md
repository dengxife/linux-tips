# CentOS安装nginx
## 下载JDK安装文件

从[Nginx官网](http://nginx.org/en/download.html)上下载安装包，选择稳定版本 Stable version。使用FTP软件将下载的rpm文件上传到CentOS目录中，以下为nginx-1.16.1版本为例。

## 安装Nginx之前需要的库

依次输入以下命令安装所需要的库。

```
yum install -y gcc-c++
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
```

## 安装Nginx

依次执行以下的命令操作。

1. 使用 `tar -zxvf nginx-1.16.1.tar.gz` 对下载的压缩包机型解压。

2. 使用 `cd` 命令进入到解压的包目录中，可以看到有个configure文件，在终端执行 `./configure` 。
3. 使用命令 `make`。
4. 使用命令 `make install` 进行安装。

安装完成后，使用 `whereis nginx` 查看一下nginx安装的位置，一般在 `/usr/local/nginx` 下。使用 `cd ` 进入到 `/usr/local/nginx/sbin` 目录下，可以看到有一个 `nginx` 文件。执行 `./nginx` 在后台启动nginx服务。

最后查看一下是否启动成功，使用命令 `ps aux | grep nginx` 查看nginx的进程，看到以下信息表示启动成功。

```
root  12018  0.0  0.0  20552   612 ?  Ss   15:38   0:00 nginx: master process ./nginx  
nobody  12019  0.0  0.0  21004  1328 ?  S    15:38   0:00 nginx: worker process          
root  12862  0.0  0.0 112724   988 pts/1  S+   15:53   0:00 grep --color=auto nginx
```

我们可在浏览器通过 `http:127.0.0.1` 来访问，nginx默认占用80端口。

### nginx命令

在`/usr/local/nginx/sbin` 目录下执行。

`./nginx` 在后台启动nginx服务。

`./nginx -s stop`  直接查出nginx进程id再使用kill命令强制杀掉进程。

`./nginx -s quit` 按照nginx进程的任务完毕后停止服务。 

`./nginx -s reload` 在后台重启nginx服务

`./nginx` 在后台启动nginx服务

## 可能的问题扩展

### CentOS的防火墙问题

有可能在安装并成功启动nginx服务后，远端无法通过浏览器访问nginx的欢迎页。

如果是开发环境，可以使用 `systemctl status firewalld` 命令查看[CentOS防火墙](http://wangchujiang.com/linux-command/c/systemctl.html)的状态，如果是 `active (running)` 的运行状态，可以 尝试使用 `systemctl stop firewalld` 将防火墙关闭。

如果是生产环境，可以尝试配置防火墙端口规则，将需要的端口放开。

### 使用Parallels Desktop虚拟机安装的CentOS

如果是使用Parallels Desktop安装的CentOS系统，在以上都确认正确的情况下，检查一下Parallels Desktop的网络适配器选项，在Parallels Desktop的右上角网络图标，将“共享网络(建议)”修改为"桥接网络(默认适配器)"。

重新使用 `ifconfig` 命令查看新的内网IP，使用新的IP访问即可。

## nginx的反向代理服务域名解析和配置

使用 `cd` 进入到nginx的配置文件所在的目录 `/usr/local/nginx/conf` 下的 `nginx.conf` 为主配置文件。一般为了规划好服务配置，我们不会再这个文件中添加服务配置信息。

在当前目录下使用 `mkdir server` 创建一个用于存放服务配置的文件目录。

然后使用 `vi nginx.conf` 编辑主配置文件，在 `http {...}` 中最后插入 `include server/*.conf;` 表示引入，退出编辑模式并保存。然后在 `/usr/local/nginx/sbin/ ` 下使用 `./nginx -s reload` 重启nginx服务生效。

在上面创建的 server 目录中可以根据情况创建以域名为名称的.conf配置文件，如 www.domain.com.conf 。

将端口代理到8080端口，在 www.domain.com.conf 的内容如下。

```json
server {
    default_type 'text/html';
    charset utf-8;
    listen 80;
    autoindex on;
    server_name www.domain.com;
    access_log /usr/local/nginx/logs/access.log combined;
    index index.html index.htm index.jsp index.php;
    #error_page 404 /404.html;
    if ( $query_string ~* ".*[\;'\<\>].*" ){
        return 404;
    }
    
    location ~ /(mmall_fe|mmall_admin_fe)/dist/view/* {
        deny all;
    }

    location / {
        proxy_pass http://127.0.0.1:8080;
        add_header Access-Control-Allow-Origin *;
    }
}
```

重启nginx服务。

下面是转发文件目录的配置。

```json
server {
    listen 80;
    autoindex off;
    server_name image.domain.com;
    access_log /usr/local/nginx/logs/access.log combined;
    index index.html index.htm index.jsp index.php;
    #error_page 404 /404.html;
    if ( $query_string ~* ".*[\;'\<\>].*" ){
        return 404;
    }

    location ~ /(mmall_fe|mmall_admin_fe)/dist/view/* {
        deny all;
    }

    location / {
        root /filespath;
        add_header Access-Control-Allow-Origin *;
    }
}
```

重启nginx服务。

## 在本机模拟域名转发

如果要模拟在本机通过域名访问到对应的服务器ip，需要修改本机 hosts 文件，路径在 `/etc/hosts` 。

例如在mac电脑上，使用命令 `vi /etc/hosts` 。在最后行添加如下。

```
172.16.4.165 www.domain.com
172.16.4.165 image.domain.com
172.16.4.165 s.domain.com
```