---
title: logvar弹幕搭建教程（喂饭版）
description: 搭建logvar弹幕的针对小白的教程
#slug: snellbuild1
date: 2025-08-16T18:05:00+08:00
image: https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage20250923102444058.png
categories:
    - VPS配置
tags:
    - 
#weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

# Logvar弹幕喂饭教程

本文是一个喂饭教程，没有任何知识，纯喂饭。独立于本站的任何其他文章。 

操作环境：debian12，非大陆VPS

目的：使用Docker搭建Logvar弹幕。

### Docker安装

依次执行以下命令，安装docker以及docker composeV2(就复制粘贴就行了 别干别的)

```
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

```
sudo apt-get update
```

```
sudo apt-get install ca-certificates curl
```

```
sudo install -m 0755 -d /etc/apt/keyrings
```

```
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
```

```
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```
sudo docker run hello-world
```

当你依次执行完以上所有命令后得到了下面这个输出，那么恭喜你，Docker已经安装好了

![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage20250923111211283.png)

### 创建相对应的文件夹

*docker compose的容器部署由于其特殊性，我一般的做法是单独创建一个docker-compose文件夹来存放所有通过compose方式创建的容器。*

在这之前，确保你的终端有你会用的编辑器，比如nano,vim,vi,emacs,neovim等，或者你会通过ssh工具编辑你vps里面的文件。我这里使用的是我最喜欢的neovim，如果你想完全照搬我的命令那可以通过以下命令先下载一下（前提是你会用neovim，它和vim的使用方式基本上一样）

```
sudo apt install neovim
```

还是依次执行以下命令，只需要复制不要做任何其他的操作

```
mkdir -p /docker-compose/logvar
```

```
cd /docker-compose/logvar
```

```
touch compose.yml
```

```
nvim compose.yml
```

把下面的内容复制进去

```
services:
  danmu-api:
    image: logvar/danmu-api:latest
    container_name: danmu-api
    ports:
      - "9321:9321"
    environment:
      - TOKEN=your_token_here  # 请将等于号有面的内容 替换为你想要的token值，方便自己用的
      #-BILIBILI_COOKIE=bilibilicookie  #这个环境变量默认是注释掉的也就是不生效，如果你需要填写的话删掉前面的#号并将等于号后面的内容改成你自己的bilibilicookie
    restart: unless-stopped    
```

保存好以后执行以下命令，检查一下这个文件里面的内容是否是你刚刚复制进去的内容

```
cat compose.yml
```

检查好没有问题之后执行以下命令

```
docker compose up -d
```

```
docker ps
```

如果你的终端给你输出了一个类似下面这张图的内容包含了danmu-api字样 并且STATUS 的表头下面写的是up 那么恭喜你已经搭建好logvar弹幕了

logvar的默认端口是9321，现在你可以在sen里面填入以下网址来使用你的弹幕API了。

```
http://VPSIP:9321/{上面compose文件里面TOKEN=后面的内容写在这里}
```

如果需要自定义域名，可以查看本站的Nginx反向代理和acme.sh申请证书这两篇文章结合起来。将弹幕服务反向代理一下。
