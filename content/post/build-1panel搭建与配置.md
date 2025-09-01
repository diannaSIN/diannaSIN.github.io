---
title: 1panel搭建与配置
description: 一个图形化界面的Nginx反向代理工具
#slug: hello-world
date: 2025-08-17T20:05:00+08:00
image: https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage/20250902012055586.jpeg
categories:
    - VPS配置
tags:
    - VPS配置
    - VPS面板
#weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

# 1panel搭建与配置

> 1panel是一个linux服务器运维管理面板

#### 安装

**在你的VPS中执行以下安装脚本，根据命令提示完成1panel的安装**（国际版1panel，推荐）：

```
curl -sSL https://resource.1panel.pro/quick_start.sh -o quick_start.sh && bash quick_start.sh
```

如果你的服务器是在国内或者想要使用国内版1panel使用以下命令：

```
bash -c "$(curl -sSL https://resource.fit2cloud.com/1panel/package/v2/quick_start.sh)"
```

安装完成以后在网页中进入你的1panel管理面板，地址为：

```
http://目标服务器 IP 地址:目标端口/安全入口
```

#### 配置

##### 1. Docker存储目录迁移(在此之前尽量先不要执行任何其他的操作，不要安装任何容器)

> 因为默认安装后docker的存储目录非常小，如果要安装大量容器或者容器可能产生大量数据的情况下就需要迁移Docker容器的目录

第一步停止docker：

![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage/20250902012113770.png)

第二步：首先找到Docker默认的存储目录，并且将它删除

![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage/20250902012127670.png)

第三步：在根目录创建Docker存储目录

在vps终端执行以下命令：

```
mkdir -p /docker
```

或者直接在1panel的根目录创建：

![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage/20250902012145512.png)

第四步:在Docker的配置中添加如下代码将存储目录指向你刚刚创建的目录，并执行重启

```
{
    "data-root": "/docker"
}
```

![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage/20250902012205178.png)

> 至此Docker的目录就已经迁移完成了

##### 2.安装第三方库

> 由于1panel面板的docker不能直接使用docker-compose、缺少部分有趣/实用的应用容器等，我们需要安装第三方应用库

前提：现在你的终端安装git

```
sudo apt install git
```

默认`1Panel`安装在`/opt/`路径下，如果你不是安装在这个目录，那需要你自己更改一下命令中的路径

在你的VPS终端，或者1panel中的系统-->终端中，依次执行以下命令，安装第三方应用库

```
git clone -b localApps https://github.com/okxlin/appstore /opt/1panel/resource/apps/local/appstore-localApps
```

```
cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/
```

```
rm -rf /opt/1panel/resource/apps/local/appstore-localApps
```

**如果你是国内网络或、国内VPS那么选择以下命令执行**

```
git clone -b localApps https://ghp.ci/https://github.com/okxlin/appstore /opt/1panel/resource/apps/local/appstore-localApps
```

```
cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/
```

```
rm -rf /opt/1panel/resource/apps/local/appstore-localApps
```

然后在终端执行以下更新软件库命令：

```
curl -sSL https://install.lifebus.top/auto_install.sh | bash
```

接着在这里就可以看到刚刚导入本地的应用库

![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage/20250902012225640.png)

在本地应用中找到1Panel-Apps这个应用就可以使用docker-compose的方式安装应用了

![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage/20250902012242460.png)

按照以下方式操作：

![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage/20250902012259895.png)

下拉，在这里编辑docker-compose文件

![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage/20250902012430893.png)
