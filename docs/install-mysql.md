# CentOS安装mysql及配置远程连接
## 下载JDK安装文件

从[Mysql官网](https://dev.mysql.com/downloads/repo/yum/)上下载安装包，在下方选择对应的RPM版本。使用FTP软件将下载的rpm文件上传到CentOS目录中，以下为mysql-8.0.17版本为例。

## 安装Mysql

使用 `cd` 命令到下载的rpm目录下，使用 `sudo rpm -ivh mysql80-community-release-el7-3.noarch.rpm` 进行安装。

下一步使用 `yum install -y mysql-server` 进行安装包的下载与安装过程，时间稍长，耐心等待直到完成。

## 启动Mysql服务

使用命令 `systemctl start mysqld` 启动mysql服务。可以使用 `systemctl status mysqld` 查看服务是否成功启动。

为了以后方便访问，可以使用 `systemctl enable mysqld.service` 设置为mysql随开机启动。

## 配置默认字符集

使用 `vi /etc/my.cnf` 打开配置文件，按 `i` 按键进入编辑模式，添加 `character_set_server = utf8mb4` 。然后按 `Esc` 按键退出编辑模式，使用 `:wq` 保存并退出。

## 本机首次登陆mysql

Mysql8的版本首次登陆的时候，要使用一个mysql提供的临时密码登录。

使用 `cat /var/log/mysqld.log|grep 'A temporary password'` 查看临时密码，显示如下，临时密码为 `ClDIJ5/5glll` 。

```
2018-04-19T09:31:04.464421Z 1 [Note] A temporary password is generated for root@localhost: ClDIJ5/5glll
```

使用 `mysql -u root -p ClDIJ5/5glll` 进行登录。

第一次登录，我们无法进行任何的操作，都会提示让我们去修改密码。

使用 `alter user user() identified by "PASSWORD_STRING";` 修改当前用户的密码。

这里要注意MySQL8的密码安全策略，如果是自己开发的使用场景，对密码安全要求弱的情况下，我们可以把密码的安全策略降低，否则会有强密码限制。

**所以我们先按照强密码的要求将root用户的密码设置好，然后降低面貌安全策略后，在更新密码为弱密码。**

降低密码安全策略的方法如下。

登录MySQL后，使用 `SHOW VARIABLES LIKE 'validate_password%';` 可以查看当前的安全策略。

```mysql
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.00 sec)
```

使用如下命令 `set global validate_password.policy=0;`  将策略级别降为LOW。使用 `set global validate_password.length=1;` 将密码字符长度降为最低的4。

再次查看，显示如下。

```mysql
mysql> SHOW VARIABLES LIKE 'validate_password%';                                                                       
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password.check_user_name    | ON    |
| validate_password.dictionary_file    |       |
| validate_password.length             | 4     |
| validate_password.mixed_case_count   | 1     |
| validate_password.number_count       | 1     |
| validate_password.policy             | LOW   |
| validate_password.special_char_count | 1     |
+--------------------------------------+-------+
7 rows in set (0.00 sec)
```

## 修改root用户可以远程连接

常规的生产环境的场景下考虑安全性，我们应该新创建用户并赋予权限来远程访问，这里由于我们为了说明MySQL的快速学习和使用，我们假设是本地的开发环境上安装MySQL并远程连接访问。

使用 `select user,host from mysql.user;` 查看MySQL的用户表信息，如下。

root的host为localhost，表示只有本机可以方案。

```mysql
mysql> select user,host from mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
4 rows in set (0.00 sec)
```

需要将root的host改为%，“%”表示所有ip可以访问。

使用命令 `update user set host='%' where user='root';` 进行修改。

使用 `flush privileges;` 刷新下即可。

## 可能遇到的问题

- 设置密码时报错 `ERROR 1819 (HY000): Your password does not satisfy the current policy requirements` 。这个的原因就是设置的密码强度太弱，需要设置强密码，如：大小写字母、数组、符号、划线等的组合。如果你觉得很麻烦，可以按照上面的办法修改密码的安全策略。
- 设置root用户的host为“%”后还是无法远程连接，请检查CentOS的防火墙是否开启，处理的方法一：配置防火墙将3306端口放行。方法二：直接关闭防火墙，命令是 `systemctl stop firewalld.service` 。