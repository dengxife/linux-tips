# 安装系统和软件源配置
## 下载CentOS系统

国内系统镜像 [https://opsx.alibaba.com](https://opsx.alibaba.com)

下载路径：[https://mirrors.aliyun.com/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso](https://mirrors.aliyun.com/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso)

## 配置软件源为阿里云

将CentOS的默认软件源文件做好备份，因为CentOS的默认软件源是指向CentOS国外官网，网络情况不佳时会下载比较慢。

`mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup`

将阿里云软件源文件下载到 `/etc/yum.repos.d` 目录下，如果你的CentOS系统版本是5或是6，将下面的 `Centos-7` 版本改为你对应的系统版本 `Centos-6.repo`

`curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo`

如果感兴趣你可以使用命令 `cat /etc/yum.repos.d/CentOS-Base.repo` 查看一下新下载的源文件内容，已经全部替换为阿里云的地址了，如下。

```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
 
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#released updates 
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/contrib/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

最后，我们要为 `yum makecache` 建立一个缓存，将阿里云软件源服务器上的软件包信息在本地建立缓存，这样以后搜索使用的时候提高速度。运行后显示以下信息表示成功。

```
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
base                                      | 3.6 kB  00:00:00     
extras                                    | 3.4 kB  00:00:00     
updates                                   | 3.4 kB  00:00:00     
(1/8): extras/7/x86_64/prestodelta        |  73 kB  00:00:00     
(2/8): extras/7/x86_64/other_db           | 131 kB  00:00:00     
(3/8): base/7/x86_64/other_db             | 2.6 MB  00:00:00     
(4/8): extras/7/x86_64/filelists_db       | 249 kB  00:00:00     
(5/8): updates/7/x86_64/prestodelta       | 945 kB  00:00:00     
(6/8): updates/7/x86_64/other_db          | 764 kB  00:00:00     
(7/8): base/7/x86_64/filelists_db         | 7.1 MB  00:00:02     
(8/8): updates/7/x86_64/filelists_db      | 5.2 MB  00:00:06     
元数据缓存已建立
```



