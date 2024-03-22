---
title: "SSH隧道和HTTP代理"
date: 2024-03-22

categories: [常用命令]
tags: [ssh隧道,端口转发,HTTP代理]
keywords: [ssh隧道,HTTP代理]

description: "SSH隧道和HTTP代理"
---

### 使用 ncat 创建HTTP代理
```sh
# 无法代理DEL等特殊Method
ncat -l --proxy-type http localhost 8888
```
### 本地端口转发 （访问本地端口 = 访问远程可以访问的端口）
```sh
ssh -L [local_addr:]local_port:remote_addr:remote_port [user@]sshd_addr
```
### 远程端口转发 （访问远程端口 = 访问本地可以访问的端口）
```sh
ssh -R [remote_addr:]remote_port:local_addr:local_port [user@]gateway_addr
```

### ssh命令端口转发相关参数
- `-N` 不执行远程命令. 用于转发端口. (仅限协议第二版)
- `-g` 允许远端主机连接本地转发的端口
- `-f` 在执行命令前退至后台

### 运用
```sh
# 在A服务器(172.20.144.200)开启HTTP代理，外网需VPN登录
ncat -l --proxy-type http localhost 8888

# PC挂VPN，本地8888端口转发至服务器A代理服务
# 执行后可以通过Top等命令保活
ssh -gL 8888:172.20.144.200:8888 user_name1@172.20.144.200

# 再将PC的8888端口转发至192.168.0.50:55555
ssh -NfgR 55555:localhost:8888 user_name2@192.168.0.50

# 至此，所有可以访问192.168.0.50:55555的机器都可以在浏览器插件或者系统代理配置HTTP代理
# 然后访问A服务器所在网络的网站等HTTP服务了
```
