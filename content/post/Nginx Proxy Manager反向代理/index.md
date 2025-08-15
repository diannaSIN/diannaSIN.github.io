---
title: Nginx Proxy Manager反向代理
description: 一个图形化界面的Nginx反向代理工具
#slug: hello-world
date: 2025-08-16 03:01:00+0000
#image: cover.jpg
categories:
    - web服务器
    - 反向代理
    - Nginx
tags:
    - 网站相关
#weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

# Nginx Proxy Manager反向代理

#### **Nginx Proxy Manager是一个运行在docker中的反向代理工具**

**注意：docker的安装与配置不在本笔记中展示，同时也不记录docker的使用方式**

### **第一步：安装Nginx Proxy Manager**

**docker-compose安装：**

```yaml
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

**docker-cli安装：**

```bash
docker run -d \\
  --name=npm \\
  -p 80:80 \\
  -p 81:81 \\
  -p 443:443 \\
  -v /home/npm/data:/data \\
  -v /home/npm/letsencrypt:/etc/letsencrypt \\
  --restart=always \\
  jc21/nginx-proxy-manager:latest
```

### **第二步：进入Nginx Proxy Manager的web管理面板：**

web管理面板地址为：vps-ip:81

默认用户名：admin@example.com

默认密码：changeme

### 第三步：配置反向代理

点击`Add Proxy`即可添加反向代理

注意：由于Nginx Proxy Manager是一个docker容器，所以代理本机服务，填写的ip为：172.17.0.1 注意：Nginx Proxy Manager在`Add Proxy`页面中只能申请单域名证书

注意：Nginx Proxy Manager申请的免费证书的颁发机构为：Let's Encrypt
