---
title: Linux网络基础
date: 2020-06-01 12:42:38
summary: Linux网络基础
categories: Linux
tags: [网络,Linux]
toc: true
---

# Linux网络基础笔记

## Linux配置IP地址

### 修改网络配置文件

1. 网卡信息文件

   ```shell
   vim /etc/sysconfig/network-scripts/ifcfg-eth0
   ---------------------------------------------
   #代网卡设备名
   DEVICE=eth0
   #是否自动获取IP（none、static表示手动，dhcp表示自动）
   BOOTPROTO=static
   #MAC地址
   HWADDR=xx:xx:xx:xx
   #是否由Network Manager图形管理工具托管
   NM_CONTROLLED=yes
   #是否随网络服务启动，eth0生效
   ONBOOT=yes
   #类型为以太网
   TYPE=Ethernet
   #唯一标识码
   UUID=xxxxxxxxxxxxxxxx
   #IP地址
   IPADDR=x.x.x.x
   #子网掩码
   NETMASK=x.x.x.x
   #网关
   GATEWAY=x.x.x.x
   #DNS
   DNS1=x.x.x.x
   #不允许非root用户控制此网卡
   USERCTL=no
   ```
<!-- more -->
2. 主机名文件
   
   ```shell
   vim /etc/sysconfig/network
   ---------------------------
   NETWORKING=yes
   HOSTNAME=localhost.localdomain 
   ```
   
3. DNS配置文件

   ```shell
   vim /etc/resolv.conf
   ---------------------------
   #DNS
   nameserver x.x.x.x
   search localhost
   ```

## 虚拟机网络参数配置

1. 配置Linux的IP地址

2. 启动网卡

   将`ONBOOT=no`改为`ONBOOT=yes`,然后重启网络

3. 修改UUID(当克隆虚拟机时执行)

   - `vi /etc/sysconfig/network-scripts/ifcfg-eth0`删除MAC地址
   - `rm -rf /etc/udev/rules.d/70-persistent-net.rules`删除网卡和MAC地址绑定文件
   - 重启系统

## Linux网络命令

### 网络环境查看命令

1. ifconfig:查看与配置网络状态命令
2. ifdown 网卡设备名:禁用网卡
3. ifup 网卡设备名:启用网卡
4. netstat 查询网络状态
   - -t:列出TCP协议端口
   - -u:列出UDP协议端口
   - -n:不适应域名与服务名,而使用IP地址和端口号
   - -l:仅列出在监听状态网络服务
   - -a:列出所有的网络连接
   - -rn:查看网关
5. nslookup:查看域名

### 网络测试命令

1. ping
   - -c 次数:指定ping包的次数
2. telnet [域名或IP] [端口]:远程管理与端口探测命令
3. traceroute [选项] IP或域名:路由跟踪协议
   - -n:使用IP,不使用域名,速度更快
4. wget 下载命令
5. tcpdump -i eth0 -nnX port 21
   - -i:指定网卡接口
   - -nn:将数据包中的域名与服务转为IP和端口
   - -X:以十六进制和ASCII码显示数据包内容
   - port:指定监听的端口

## Linux远程登录

### SSH命令

1. ssh 用户名@ip  #远程管理指定Linux服务器
2. scp [-r] 用户名@ip:文件路径 本地路径   #下载文件
3. scp [-r] 本地路径 用户名@ip:文件路径   #上传文件