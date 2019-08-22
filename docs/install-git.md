# CentOS安装Git及配置SSH秘钥
## 下载Git安装文件

从[Git的GitHub官网](https://github.com/git/git/releases)上下载最新稳定版本的安装包，在下方选择对应的tar.gz包。使用FTP软件将下载的tar.gz文件上传到CentOS目录中，以下为git-2.23.0-rc2.tar.gz版本为例。

## 安装git的依赖库

使用 `yum -y install zlib-devel openssl-devel cpio expat-devel gettext-devel curl-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker` 命令，将git所需的依赖库安装。

## 安装git

使用命令 `tar -zxvf git-2.23.0-rc2.tar.gz `  解压安装包。

完成解压过程后，使用 `cd` 命令进入解压好的 git-2.23.0-rc2 目录中。

编译命令 `make prefix=/usr/local all` 进行编译。

最后，进行安装  `make prefix=/usr/local install` 。

使用 `git --version` 验证一下版本信息。

## 配置SSH秘钥

### 生成SSH秘钥

```bash
# ssh-keygen命令 用于为“ssh”生成、管理和转换认证密钥，它支持RSA和DSA两种认证密钥。
# -C：添加注释
# -t：指定要创建的密钥类型
# 最后接Github邮箱账号
~ $ ssh-keygen -t rsa -C "你仓库网站的邮箱账号"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/dengxiongfei/.ssh/id_rsa):
/Users/dengxiongfei/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/dengxiongfei/.ssh/id_rsa.
Your public key has been saved in /Users/dengxiongfei/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ycy/...一串字符串... 你仓库网站的邮箱账号
The key's randomart image is:
+---[RSA 2048]----+
|   +==oo+...     |
|  . .o+*.+.      |
| . . .+ = o      |
|o o .. * = . .   |
|oo . .o S o o    |
| .E o  . + .  .  |
|     . .  +. o   |
|      o + .=+    |
|       o ++oo.   |
+----[SHA256]-----+

# ssh-add命令 是把专用密钥添加到ssh-agent的高速缓存中
~ $ ssh-add ~/.ssh/id_rsa
\Identity added: /Users/username/.ssh/id_rsa (/Users/username/.ssh/id_rsa)

# 查看生成的SSH-Keys
~ $ cat ~/.ssh/id_rsa.pub
ssh-rsa ...一串字符... 你仓库网站的邮箱账号
```

### 添加SSH秘钥

使用账号登录的GitHub官网后，在左上角的用户头像下选择 settings 项，在左侧导航选择 SSH and GPG keys ，点击左上角的绿色按钮 New SSH key 后，填写SSH名称和将你上面生成的SSH字符串复制粘贴到这里，保存即可。