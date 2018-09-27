---
layout: post
title: 使用fail2ban防止ssh暴力登录
guid: 208ba9affb8b4d95a3954aa3e96e5944
categories: [Linux]
tags: [linux]
redirect_from:
    - /2018/09/27/
---

**目录：**
* Kramdown table of contents
{:toc .toc}
* * * 

# 使用Fail2Ban防止ssh暴力登录 {#use-fail2ban-to-protect-your-vps}
今儿中午，心血来潮统计了一下ssh失败次数，发现最高的一个ip有4000多次登录失败记录。这让我想起了那个传闻：暴露在外网的机器，如果不做防护，八个小时内就易主了。所以找了找防止暴力尝试的方法，发现现在常规方法就是使用fail2ban监控，然后自动禁用可疑ip，部署方法如下：

## 安装fail2ban {#install-fail2ban}
```shell
yum -y install fail2ban
```
**请注意防火墙软件是`firewalld`而不是`iptables`**

```shell
# 查看状态
firewall-cmd --state

# 启动firewalld
systemctl start firewalld

# 开机启动
systemctl enable firewalld
```

默认的`firewalld`会禁用所有端口连接，因此你需要手动的放行端口，比如你的博客服务的web端口：
```shell
firewall-cmd --zone=public --add-port=80/tcp --permanent

# 重载配置
firewall-cmd --reload

# 查看已经放行的端口
firewall-cmd --zone=public --list-ports
```

## 配置fail2ban {#config-fail2ban}

fail2ban的配置文件位于`/etc/fail2ban/`， 我们在该目录新建一个自定义配置文件`jail.local`
然后加入如下信息：
```shell
[DEFAULT]
ignoreip = 127.0.0.1/8
bantime = 86400
findtime = 600
maxretry = 5
banaction = firewallcmd-ipset
sender = email@email.com
senername = Fail2Ban
action = %(action_mwl)s

[sshd]
enabled = true
filter = sshd
port = 22
action = %(action_mwl)s
logpath = /var/log/secure
```

## 启动fail2ban {#start-fail2ban}

```shell
systemctl start fail2ban
```

## 查看fail2ban日志以及ip禁用列表 {#check-banned-ip-list}
```shell
# 查看日志
tail -F /var/log/fail2ban

# 查看禁用ip
fail2ban-client status sshd
```
