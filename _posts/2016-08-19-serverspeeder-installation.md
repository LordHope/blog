---
layout:            post
title:             "安装服务器加速软件 ServerSpeeder"
date:              2016-08-19 15:07:00 +0800
author:            Lord Hope
category:          Tutorials
tags:              ServerSpeeder 
language:          zh-Hans
---

锐速安装到服务器上，对于从服务器到访问服务器的用户的流量，会起到最好的加速效果；而对于从用户到服务器的流量，加速效果会不稳定甚至没有加速效果（但不会比没安装锐速之前慢）。锐速本质是一个实现了 Zeta-TCP 的软件。

#### 内核支持

+ [支持的内核版本](http://dl.serverspeeder.com/ls.do?m=availables)

选用 Ubuntu 14.04 x64 并升级内核为

+ 3.13.0-46-generic
+ 3.16.0-28-generic

选用 CentOS 6.5 x64 并升级内核为

+ 2.6.32-431.el6

### 替换内核

#### 1. Ubuntu 14.04 x64 更换为 3.16.0-28-generic

安装内核 3.16.0-28-generic

```bash
sudo apt-get install linux-image-extra-3.16.0-28-generic
```

显示当前系统已安装内核

```bash
dpkg -l | grep linux-image
```

卸载除 **3.16.0-28-generic** 以外的内核

```bash
sudo apt-get purge linux-image-3.13.0-xx-generic linux-image-extra-3.13.0-xx-generic
```

更新 grub 系统引导文件

```bash
sudo update-grub
```

重启系统查看验证

```bashbash
sudo reboot
uname -r
```

#### 2. CentOS 6.5 x64 更换为 2.6.32-431.el6

下载安装内核

```bash
rpm -ivh http://vault.centos.org/6.5/centosplus/x86_64/Packages/kernel-firmware-2.6.32-431.el6.centos.plus.noarch.rpm
rpm -ivh http://vault.centos.org/6.5/centosplus/x86_64/Packages/kernel-2.6.32-431.el6.centos.plus.x86_64.rpm --force
```

显示当前系统已安装内核

```bash
rpm -qa | grep kernel
```

重启系统查看验证

```bash
sudo reboot
uname -r
```

### 安装锐速破解版

+ [91yun](https://www.91yun.org/archives/683) 3.10.61.0

安装

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh
```

卸载

```bash
chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f
```

+ [X鸡的博客](https://www.233.wiki/2016/02/21/124.html)

获取安装脚本

```bash
wget -q -O- http://file.idc.wiki/get.php?serverSpeeder | bash -
```

运行安装脚本

```bash
bash serverSpeeder_setup.sh
```

### 启动与关闭

```bash
service serverSpeeder start
service serverSpeeder stop
```