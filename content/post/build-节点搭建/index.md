---
title: snell节点搭建
description: 在你的vps中搭建snell节点
#slug: hello-world
date: 2025-08-16 18:40:00+0000
image: cover.jpeg
categories:
    - 魔法制作
tags:
    - 魔法上网
    - VPS配置
    - 节点搭建
#weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

本文参考了[整点猫咪](https://surge.tel/16/349/)大佬的文章，并做了一些修改与简化

注意：基于debian12，应该适用于debian系列机器

#### 第一步：下载必要软件

```
sudo apt update && sudo apt install wget unzip neovim -y
```

如果你不是debian系列机器还需要这一步操作：

```
sudo dnf install unzip
```

#### 第二部：下载SnellServer

> 文档使用的是snell的最新版本也就是V5，根据你的创建时间和实际需求自行去[官网](https://kb.nssurge.com/surge-knowledge-base/zh/release-notes/snell)查找版本号。

**linux-amd64**

```
wget https://dl.nssurge.com/snell/snell-server-v5.0.0-linux-amd64.zip
```

**ARM架构**

```
wget https://dl.nssurge.com/snell/snell-server-v5.0.0-linux-aarch64.zip
```

#### 第三步解压 Snell Server 到置顶目录

**linux-amd64**

```
sudo unzip snell-server-v5.0.0-linux-amd64.zip -d /usr/local/bin
```

**ARM架构**

```
sudo unzip snell-server-v5.0.0-linux-aarch64.zip -d /usr/local/bin
```

#### 第三步：操作节点搭建

**赋予服务器权限**

```
chmod +x /usr/local/bin/snell-server
```

**创建配置文件夹**

```
sudo mkdir /etc/snell
```

**使用Snell的wizard生成一个配置文件**

```
sudo snell-server --wizard -c /etc/snell/snell-server.conf
```

**配置Systemd服务文件**

编辑snell.serverce文件

```
sudo nvim /lib/systemd/system/snell.service
```

直接把下面的代码复制粘贴进去

```
[Unit]
Description=Snell Proxy Service
After=network.target

[Service]
Type=simple
User=nobody
Group=nogroup
LimitNOFILE=32768
ExecStart=/usr/local/bin/snell-server -c /etc/snell/snell-server.conf
AmbientCapabilities=CAP_NET_BIND_SERVICE
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=snell-server

[Install]
WantedBy=multi-user.target
```

**继续执行重载服务**

```
sudo systemctl daemon-reload
```

**配置snell开机自启动**

```
sudo systemctl enable snell
```

**查看snell状态**

```
sudo systemctl status snell
```

> 执行完以上步骤，并且snell状态没有报错（全绿），那节点就搭建好了

#### 第四步：查看Snell配置，并且在Surge中添加

**查看snell配置**

```
cat /etc/snell/snell-server.conf
```

**你会得到如下输出：**

```
[snell-server]
listen = 0.0.0.0:11807
psk = AijHCeos15IvqDZTb1cJMX5GcgZzIVE
ipv6 = false
```

**如果直接在surge的配置文件中添加那么直接写**：

```
节点名称 = snell, XXX.XXX.XXX.XXX, 11807, psk=psk, version=5, tfo=true
```

**如果要在Surge的GUI中添加那么这样填写**：

名称：节点名称

代理类型：选择SNELL

服务器地址：填写搭建SNELL服务器的IP地址

端口号：填写listen那里冒号后面的部分

psk:填写psk

协议版本：选择v5

Obfs：Disabled

TCP Fast Open：开启

IPV4&IPV6：双栈
