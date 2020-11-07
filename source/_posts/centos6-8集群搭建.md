---
title: centos6.8集群搭建
date: 2020-05-29 15:30:49
description: Centos6.8集群搭建
categories: Hadoop
tags: [集群,MySQL,Redis]
---

关机：halt

重启：reboot

修改本机ip：

```shell
vim /etc/sysconfig/network-scripts/ifcfg-eth0 
```



```shell
DEVICE=eth0
HWADDR=00:0C:29:81:D1:F5
TYPE=Ethernet
UUID=2934354a-b8b3-4fcd-9bd4-8bb9142b87ce
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.2.128
GATEWAY=192.168.2.1
BROADCAST=192.168.2.255
DNS1=8.8.8.8
DNS2=114.114.114.114

```

<!-- more -->

修改hosts：

```shell
vim /etc/hosts
```

```shell
192.168.2.128 hadoop100
```

重启网络服务：

```shell
service NetworkManager stop
/etc/init.d/network restart
```

关闭关机自启：

```shell
chkconfig NetworkManager off
```

设置nameserver：

```shell
vim /etc/resolv.conf
nameserver 192.168.2.1
```

配置防火墙：

```shell
# 关闭
service iptables stop
# 关闭开机自启
chkconfig iptables off
# 查看防火墙状态
service iptables status
```

Ubuntu黑屏

`netsh winsock reset`

修改ip之后需要删除一个文件才能成功

```shell
rm /etc/udev/rules.d/70-persistent-net.rules
```

SSH免密登录

1. 生产密钥`ssh-keygen -t rsa`
2. 私钥发送本机`ssh-copy-id localhost`
3. 公钥发送其他计算机`ssh-copy-id 192.168.2.128`



记一次UUID和HWADDR的坑

在虚拟机设置中获取MAC地址`00:50:56:25:32:15`

在`/etc/sysconfig/network-scripts/ifcfg-eth0`中添加`HWADDR=xxx`

使用`uuidgen eth0`生成`UUID`，并在`/etc/sysconfig/network-scripts/ifcfg-eth0`中添加

删除`rm -rf /etc/udev/rules.d/70-persistent-net.rules`

重启

> 也可以克隆虚拟机后，直接修改`/etc/udev/rules.d/70-persistent-net.rules`，将`eth1`改为`eth0`，然后复制`HWADDR`写入`/etc/sysconfig/network-scripts/ifcfg-eth0 `



### 设置yum阿里镜像

1. 备份

   `cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup `

2. 配置软件源

   ```
   # Centos-6
   wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
   
   # Centos-7
   wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
   ```

3. 生成缓存

   `yum makecache`

集群分发工具

- scp

- rsync

- xsync

  ```shell
  #!/bin/bash
  #1 获取输入参数个数，若没有参数，直接退出
  pcount=$#
  if((pcount==0)); then
  echo no args;
  exit;
  fi
  
  #2 获取文件名称
  p1=$1
  fname=`basename $p1`
  echo fname=$fname
  
  #3 获取上级目录的绝对路径
  pdir=`cd -P $(dirname $p1); pwd`
  echo pdir=$pdir
  
  #4 获取当前用户名称
  user=`whoami`
  
  #5 循环
  for((host=103; host<105; host++)); do
  	echo ----------hadoop$host------------
  	rsync -rvl $pdir/$fname $user@hadoop$host:$pdir
  done
  ```

### 集群时间同步ntp



------

## Centos6.8安装MySQL5.7

1. 下载https://dev.mysql.com/downloads/mysql,选择对应版本，上传到虚拟机，解压

2. 卸载系统自带MySQL`rpm -qa|grep mysql`，若存在则`yum remove ***`

3. 创建用户组和用户

   创建用户组`groupadd mysql`

   创建用户`useradd -r -g mysql mysql`

4. 给MySQL用户指定专有用户和用户组

   - 创建data文件夹`cd /usr/local/mysql`，`mkdir data`
   - 指定用户组和用户`cd /usr/local`,`chown -R mysql mysql/`,`chgrp -R mysql mysql/`

5. 初始化MySQL

   - `cd /usr/local/mysql/bin`,`yum -y install numactl`

   - `./mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/ --lc_messages_dir=/usr/local/mysql/share --lc_messages=en_US`,记住生成的临时密码`T(nUEMv&f2-7`

     > 如果忘记密码或者想重新初始化，可以先将mysql/data目录中文件删除，然后再执行初始化命令

6. 配置my.cof

   `vim /etc/my.cof`

   ```shell
   [mysqld]
   basedir=/usr/local/mysql
   datadir=/usr/local/mysql/data
   ```

7. 启动MySQL

   `cd /usr/local/mysql/bin`

   `./mysqld_safe --user=mysql &`

8. 设置开机自启动

   `cd /usr/local/mysql/support-files/`

   `cp mysql.server /etc/init.d/mysql`

   `vim /etc/init.d/mysql`

   ```shell
   basedir=/usr/local/mysql/
   datadir=/usr/local/mysql/data/
   ```

   授权`chmod +x /etc/init.d/mysql`

   设为开机启动`chkconfig --add mysql`

9. 启动服务

   - 启动`service mysql start`
   - 停止`service mysql stop`
   - 重启`service mysql restart`
   - 查看`service mysql status`

10. 登录MySQL

    `cd /usr/local/mysql/bin`

    `./mysql -u root -p`

    修改密码`set password=password("root");`

    登录授权`grant all privileges on *.* to'root' @'%' identified by '123456';`

    授权生效`flush privileges;`



## Centos6.8安装Redis

1. 下载http://download.redis.io/releases/redis-5.0.8.tar.gz解压

2. 进入redis目录，`make && make install`

3. 将redis安装为系统服务并后台启动

   `cd /usr/local/redis/utils`

   `./install_server.sh `

4. 查看redis服务启动情况`systemctl status redis_6379.service`

   > centos6没有systemctl命令，可以使用service --status-all显示所有服务

5. 启动redis客户端

6. 设置允许远程连接

   - 编辑redis配置文件`vim /etc/redis/6379.conf`
   - 将`bind 127.0.0.1`改为`0.0.0.0`
   - 设置访问密码，将`#requirepass foobared`注释去掉，并将`foobared`改为密码。使用`auth password`进行验证
   - 重启redis `systemctl restart redis_6379.service`

- Docker部署Redis

  - `docker pull redis`

  - `mkdir  -p /root/redis/redis01/conf
    mkdir  -p /root/redis/redis01/data
    cd /root/redis/redis01/conf
    #下载一个redis.conf文件
    wget http://download.redis.io/redis-stable/redis.conf
    `

  - `cd ../
    docker run -p 6379:6379 --name hbk_redis -v $PWD/conf/redis.conf:/etc/redis/redis.conf -v $PWD/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes`

  - 修改redis.conf

    ```conf
    # bind 127.0.0.1
    bind 0.0.0.0
    
    # protected-mode yes
    protected no
    
    # requirepass foobared
    requirepass 123456
    
    #daemon 不能是yes
    ```

    

## Centos6.8安装RabbitMQ

1. 安装Erlang
   - 安装依赖包`yum install xmlto gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel unixODBC-devel wxBase wxGTK wxGTK-gl perl -y`
   - Erlang下载`yum install -y erlang-19.0.4-1.el6.x86_64.rpm`
2. 安装RabbitMQ

`curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm
.sh | sudo bash `





- 使用Docker部署RabbitMQ(management才有页面)
  - `docker pull rabbitmq:management`
  - `docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management`
  - 









Ubuntu18.04更换阿里镜像

1. 查找源配置文件

   `find / -name sources.list`

2. 修改source.list

   ```
   deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
   ```

3. `sudo apt-get update`