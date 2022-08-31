---
title: Docker
date: 2020-08-08 14:30:19
description: Docker笔记
categories: Docker
tags: 
	- Docker
	- Ubuntu18.04
---

# Ubuntu18.04安装Docker

> [参考](https://www.runoob.com/docker/ubuntu-docker-install.html)

1.更换国内软件源，推荐中国科技大学的源，稳定速度快（可选）

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sudo apt update
```

2.安装需要的包

```
sudo apt install apt-transport-https ca-certificates software-properties-common curl
```

3.添加 GPG 密钥，并添加 Docker-ce 软件源，这里还是以中国科技大学的 Docker-ce 源为例

```
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
$(lsb_release -cs) stable"
```

4.添加成功后更新软件包缓存

```
sudo apt update
```

5.安装 Docker-ce

```
sudo apt install docker-ce
```

6.设置开机自启动并启动 Docker-ce（安装成功后默认已设置并启动，可忽略）

```
sudo systemctl enable docker
sudo systemctl start docker
```

7.测试运行

```
sudo docker run hello-world
```

8.添加当前用户到 docker 用户组，可以不用 sudo 运行 docker（可选）

```
sudo groupadd docker
sudo usermod -aG docker $USER
```

9.测试添加用户组（可选）

```
docker run hello-world
```

10.更改docker源

```
# 没有就新建
sudo vim /etc/docker/daemon.json
# 修改配置
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
# 使配置文件生效
sudo systemctl daemon-reload
# 重启Docker
sudo service docker restart
```

| 国内Docker镜像仓库名称 | 链接                                                         |
| ---------------------- | ------------------------------------------------------------ |
| Docker 官方中国区      | [https://registry.docker-cn.com](https://registry.docker-cn.com/) |
| 网易                   | [http://hub-mirror.c.163.com](http://hub-mirror.c.163.com/)  |
| 中国科学技术大学       | [https://docker.mirrors.ustc.edu.cn](https://docker.mirrors.ustc.edu.cn/) |
| 阿里云                 | https://<你的ID>.mirror.aliyuncs.com                         |

![](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/26010.jpg)


