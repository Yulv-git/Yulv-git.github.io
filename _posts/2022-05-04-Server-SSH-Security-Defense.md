---
layout: post
title: SSH security defense after server is hacked and mined | 服务器被黑客攻击和被挖矿之后的SSH安全防御
tags: Server
---

PS：本人非专业网络安全攻防人员，仅为个人实践经验。本文可能难免有不当之处，欢迎您的讨论和指导。

## 1. 确定服务器是否被黑客攻击和被挖矿

确认服务器是否出现被黑客攻击和被挖矿等异常情况的方法有：

### 1.1. 查看显卡使用是否异常

使用`nvidia-smi`或其他命令发现显卡使用出现异常，如：

- 显卡被某用户的不明进程占用，且利用率很高甚至为100%。

### 1.2. 查看进程信息是否异常

使用`top`或其他命令发现进程出现异常，如：

- 某用户的进程运行命令中出现了`eth`、`coin`、`wallet`、`pool`或其他与挖矿相关的参数。

PS：黑客经常很狡猾的，会将挖矿程序改成python或其他常用的程序的名字，其运行参数也可能改成其他名字，故有时较难发现某进程是否为挖矿进程。

### 1.3. 查看登录用户信息是否异常

使用`last`或其他命令发现某用户的登录出现异常，如：

- `登录时间`异常（非内部人员服务器使用习惯的登录时间，比如在三更半夜时登录）；
- `登录ip`异常（非已知的内部人员所用网络设备的ip，比如外网甚至是国外的ip，比如校园网内其他非内部人员的ip）。

### 1.4. 查看可能异常用户是否有异常文件

若发现某用户的文件中存在一些挖矿相关配置的文件或者程序，则该用户很可能被用来进行过挖矿等异常操作。和该用户沟通确认后，对挖矿或其他异常操作的文件或程序进行删除。挖矿相关配置文件示例（黑客在root用户目录的./.cache/.x/目录的config.ini的挖矿配置信息）：

![ETH_show](/images/blog/ETH_show.png)

PS：黑客可能比较狡猾，将进行异常操作的文件或程序命名为不容易察觉的名字或者存放到不容易发现的地方。

## 2. 设置SSH安全防御

配置hosts.allow和hosts.deny文件来过滤通过SSH连接服务器的ip，设置流程如下：

### 2.1. 配置hosts.allow

在`/etc/hosts.allow`文件中添加如下内容：

``` txt
sshd:192.168.1.108
sshd:192.168.2.
```

其中，`sshd:192.168.1.108`是允许单个ip进行SSH访问，`sshd:192.168.2.`是允许ip段内的所有ip进行SSH访问。

### 2.2. 配置hosts.deny

在`/etc/hosts.deny`文件中添加如下内容：

``` txt
sshd:ALL
```

其中，`sshd:ALL`是禁止除了`/etc/hosts.allow`中允许的ip之外的所有其他ip进行SSH访问（hosts.allow权限等级高于hosts.deny）。

### 2.3. 重启SSH服务

``` bash
sudo service sshd  restart
```

## 3. 验证SSH安全防御效果

查看`/var/log/auth.log`或其他登录日志文件（如/var/log/auth.log.1），确定是否有`/etc/hosts.allow`中允许的ip之外的ip被成功禁止了SSH访问。可用如下命令查看：

``` bash
sudo cat /var/log/auth.log | grep refused
```

![SSH_refused_show](/images/blog/SSH_refused_show.png)

此处，公示下某个一直尝试爆破我所用服务器的校园内的ip（至今被探查到连接失败就有54555次）:

``` txt
54555  Apr 28 08:07:17 amax sshd[852562]: refused connect from 172.31.111.189 (172.31.111.189)
```

PS：若您在设置SSH安全防御之后，黑客没有再通过SSH攻击您的服务器，那可能没有被禁止访问的ip记录。此时，您可将您现有某个ip不放到`/etc/hosts.allow`中，并拿其来连接您的服务器就会有其访问记录了。
