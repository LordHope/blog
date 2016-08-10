---
layout:            post
title:             "使用 Squid 搭建 Https 代理"
date:              2016-08-10 11:21:00 +0800
author:            Lord Hope
category:          Tutorials
tags:              Proxy Http
language:          zh-Hans
---

## 基础安装

安装 apache2-utils 用于 HTTP 认证文件的生成

```bash
apt-get install apache2-utils -y
```

安装 Squid

```bash
apt-get install squid3 -y
```

安装 stunnel

```bash
apt-get install stunnel4 -y
```

## 配置 Squid

修改 Squid 默认配置，配置文件位于 `/etc/squid3/squid.conf`

### 1. 修改监听地址与端口号

找到`TAG: http_port`注释，把其下方的

> Squid normally listens to port 3128
> http_port 3128

中`http_port`的值`3128`修改为`127.0.0.1:3128`，使得 Squid 只能被本地（127.0.0.1）访问。此处可以修改为监听其他端口号。

### 2. 修改访问权限与HTTP认证（可选）

若不需要添加 HTTP 认证，只需将`http_access deny all`修改为`http_access allow all`即可，无需下列的操作。

使用如下命令生成认证文件，

```bash
htpasswd -c /etc/squid3/squid.passwd <登录用户名>
```

再次打开Squid配置文件`/etc/squid3/squid.conf`，找到`TAG: auth_param`注释，在其下方添加，

> auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid3/squid.passwd
> auth_param basic children 5
> auth_param basic realm Squid proxy-caching web server
> auth_param basic credentialsttl 2 hours
> auth_param basic casesensitive off

找到`TAG: acl`，在其下方添加，

> acl ncsa_users proxy_auth REQUIRED

找到`TAG: http_access`，在其下方添加，使得只允许经过认证的用户访问，

> http_access deny !ncsa_users
> http_access allow ncsa_users

### 3. 重启Squid

```bash
service squid3 restart
```

## 配置 stunnel

接下来，我们需要在 Squid 上添加一层加密。

### 1. 生成公钥和私钥

生成私钥（privatekey.pem）：

```bash
openssl genrsa -out privatekey.pem 2048
```

生成公钥（publickey.pem）：

```bash
openssl req -new -x509 -key privatekey.pem -out publickey.pem -days 1095
```

（需要注意的是，`Common Name`需要与**服务器的IP或者主机名**一致）
合并：

```bash
cat privatekey.pem publickey.pem >> /etc/stunnel/stunnel.pem
```

### 2. 修改 stunnel 配置

新建一个配置文件`/etc/stunnel/stunnel.conf`，输入如下内容

```json
client = no
[squid]
accept = 4128
connect = 127.0.0.1:3128
cert = /etc/stunnel/stunnel.pem
```

配置中指定了 stunnel 所暴露的 HTTPS 代理端口为`4128`，可以修改为其他的值。

修改`/etc/default/stunnel4`配置文件中`ENABLED`值为`1`。

 > ENABLED=1

### 3. 重启 stunnel

```bash
service stunnel4 restart
```

至此，服务器端已配置完成了。

### 本地浏览器配置

添加证书到受信任的根证书颁发机构列表中

以 Windows下Chrome 浏览器为例，将服务器上的公钥`publickey.pem`下载至本地，重命名至`publickey.crt`，在Chrome中依次点击 “设置” - “显示高级设置” - “HTTP/SSL” - “管理证书”，在“受信任的根证书颁发机构”选项卡中“导入”这个 crt 证书就完成了。

### 代理客户端配置

将本地的代理客户端指向`https://<你的服务器IP或主机名>:4128`，这里的**IP或主机名**和生成公钥时的Common Name 一致，端口为 stunnel 的端口。如果有配置 HTTP 认证的话，需要在客户端中配置对应的用户名和密码。如果没有 HTTP 客户端的话，推荐使用 Chrome 的插件 [Proxy SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega)（使用教程可以参考 Github 上的 [Wiki](https://github.com/FelisCatus/SwitchyOmega/wiki)）。