---
title: 阿里云服务器搭建笔记
catalog: true
date: 2020-07-02 00:43:53
subtitle:
header-img:
tags:
- devops
categories:
- 工程
---

创建用户

```
# useradd –d  /home/soaringhawkcheng -m soaringhawkcheng
# passwd soaringhawkcheng
# su - soaringhawkcheng
```

将用户添加到sudoer列表

```
以root用户去修改/etc/sudoers文件，修改该文件我们使用visudo命令（命令行直接写visudo，回车即可）

## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
soaringhawkcheng     ALL=(ALL)       ALL
```

ssh配置

```
ssh-keygen
cd ~/.ssh
touch authorized_keys
# 粘贴mac公钥
ssh-copy-id -i ~/.ssh/id_rsa.pub "root@192.168.XXX.XXX"
sudo vim /etc/ssh/sshd_config
```

```
RSAAuthentication yes
PubkeyAuthentication yes
```

```
//用户权限

chmod 700 /home/username

//.ssh文件夹权限

chmod 700 ~/.ssh/

// ~/.ssh/authorized_keys 文件权限

chmod 600 ~/.ssh/authorized_keys
```

