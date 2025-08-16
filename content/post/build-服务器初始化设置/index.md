---
title: 服务器初始化设置
description: 刚买的服务器刚开机的一些必要设置
#slug: hello-world
date: 2025-08-16 02:45:00+0000
image: cover.jpeg
categories:
    - VPS配置
tags:
    - VPS配置
    - 服务器相关    
#weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

# 服务器初始化设置

**注意：基本上机遇Debian12**

### DD系统

在购买到一台全新的vps之后，一般情况下 要DD系统，这里使用一键DD

一键DD原项目仓库地址：[Tools](https://github.com/leitbogioro/Tools)

安装wget

```bash
apt install wget sudo -y
```

下载脚本

```bash
wget --no-check-certificate -qO InstallNET.sh '<https://raw.githubusercontent.com/leitbogioro/Tools/master/Linux_reinstall/InstallNET.sh>' && chmod a+x InstallNET.sh
# 国内vps使用这条命令
wget --no-check-certificate -qO InstallNET.sh '<https://gitee.com/mb9e8j2/Tools/raw/master/Linux_reinstall/InstallNET.sh>' && chmod a+x InstallNET.sh
```

执行DD

要选择与自己的操作系统对应的命令，以下五条命令都是常用的系统服务器使用该脚本的DD命令

```bash
# 1> Debian12
sudo bash InstallNET.sh -debian
# 2> Kali Rolling
sudo bash InstallNET.sh -kali
# 3> Alpine Linux Edge
sudo bash InstallNET.sh -alpine
# 4> CentOS 9 stream
sudo bash InstallNET.sh -centos
# 5> Ubuntu 22.04
sudo bash InstallNET.sh -ubuntu
```

脚本执行后在终端输入以下指令执行重启会自动安装系

```bash
reboot
```

DD系统后会生成默认的用户名与密码，DD系统后应该首先修改默认密码

用户名与密码

```bash
默认用户名：root
默认密码：LeitboGi0ro
```

如果DD系统后主机名是ip地址想要更换那么编辑以下内容更换为你想要的即可

```bash
nvim /etc/hostname
```

### 默认连接设置

**出于安全考虑，应该首先更改默认的ssh连接配置**

使用以下命令编辑ssh配置文件修改默认ssh连接端口

```bash
sudo nvim /etc/ssh/sshd_config
```

编辑Port 为自己想要使用的端口默认为：

```bash
Port 22
```

重启ssh服务

```
sudo systemctl restart ssh
```

