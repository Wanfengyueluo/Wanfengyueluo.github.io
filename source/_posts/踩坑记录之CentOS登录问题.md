---
title: 踩坑记录之CentOS登录问题
date: 2020-12-03 10:38:08
description: 前事不忘，后事之师
categories: 踩坑记录
tags: 
	- 踩坑
	- CentOS
cover: https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/note_cover1.png
---

![image-20201203104847731](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/note_cover1.png)

最近登录虚拟机，发现正确的用户和密码无法登录，出现如下图的问题提示：

![image-20201203105125584](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201203105125584.png)

刚好前几天看《Linux就该这么学》这本书上有讲述如何重置root管理员密码的内容，索性尝试一下。

1. 重启系统

![image-20201203105803864](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201203105803864.png)

2. 在进入系统之前选中第一行，点击`e`进入该选项

![image-20201203105826300](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201203105826300.png)

3. 进入的编辑模式是这样的：

![image-20201203110226134](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201203110226134.png)

4. 往下找，找到`linux16`这一行，在末尾`UTF-8`后面进行编辑，空格之后输入`rd.break`，换行的时候会自动加上换行符`\`，然后`Ctrl + X`运行该程序。

![image-20201203110336712](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201203110336712.png)

5. 大概30秒后会进入下面的界面，即系统的紧急救援模式。依次输入以下命令，然后等待系统重启。

```shell
mount -o remount,rw /sysroot
chroot /sysroot
passwd
rouch /.autorelabel
exit
reboot
```



![image-20201203110554935](https://cdn.jsdelivr.net/gh/Wanfengyueluo/images/image-20201203110554935.png)

6. 完成！