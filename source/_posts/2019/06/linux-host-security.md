---
title: linux主机安全设置
date: 2019-06-26 23:08
tags:
- security
- server
- linux
categories: linux
---

一直在vps安全方面关注较少，某天服务器异常才注意到安全问题，看日志发现网上恶意攻击的行为很多，学习了一些简单的安全防护方法，整理一下。

### ssh相关
1.修改默认端口
修改ssh默认端口是操作最简单，性价比最高的一条，可以阻挡大量的恶意攻击。

```conf
# vim /etc/ssh/sshd_config 
#Port [端口]
Port 1234
```
*修改Port前请确认防火墙相应端口已打开，如果修改失败可以先关闭selinux再试*

2.使用证书登录，禁用密码登录
生成ssh key，将pub key添加到$HOME/.ssh/authorized_keys中。

```conf
# 使用证书登录
AuthorizedKeysFile .ssh/authorized_keys
# 禁用密码登录
PasswordAuthentication no
```

3.禁用root帐户登录
添加新用户并使用visudo命令为新用户添加sudo权限，禁用root帐户登录。

```conf
PermitRootLogin no
```

### 端口设置白名单

现在很多vps后台自带端口管理工具，我们在后台中将需要使用的端口开放即可。后台未提供管理工具的，可以使用iptables工具进行设置，为iptables为例：

```bash
# 允许访问 127.0.0.1
iptables -A INPUT -i lo -p all -j ACCEPT
# 允许ping
iptables -A INPUT -p icmp -j ACCEPT
# 允许ssh，将端口修改为自己的ssh服务端口
iptables -A INPUT -p tcp –dport 1234 -j ACCEPT
# 开启web端口
iptables -A INPUT -p tcp –dport 80 -j ACCEPT
# 禁用配置之外的访问
iptables -P INPUT DROP
```

### 查看登录记录

```bash
last
```

查看登录日志，查看是否有异常登录行为。

### 查看ssh日志

```bash
tail /var/log/secure
```

查看ssh服务日志，是否有异常登录及密码错误尝试。