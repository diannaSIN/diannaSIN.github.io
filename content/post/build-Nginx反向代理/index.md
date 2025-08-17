---
title: Nginx反向代理
description: 简单的利用Nginx配置反向代理
#slug: hello-world
date: 2025-08-16 02:59:00+0000
image: cover.jpg
categories:
    - 网站相关
tags:
    - web服务器
    - 反向代理
    - Nginx
#weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

# Nginx反向代理

**注意：基于debian12**

### 第一步：安装Nginx

**由于debian的apt包常年不更新软件包版本所以需要先添加nginx的官方软件包到 debian 系统中**

下载密钥

```bash
sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring
```

导入官方密钥签名

```bash
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

配置存储库

```bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

设置固定存储库

```bash
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

更新软件包之后安装 nginx

```bash
sudo apt update
sudo apt install nginx
```

### 第二步：定位nginx配置

在debian12中 nginx默认的配置文件夹路径

```bash
cd /etc/nginx/
```

进入这个文件夹创建你要反向代理的服务的文件

```bash
cd nginx.d
touch server.conf
```

在nginx根目录下新建一个用于存放SSL证书的文件夹

证书文件夹应该直接存放在 nginx 的根目录

文件夹内部每一个域名单独设置 比如 nginx/sslFile/example.com

```bash
mkdir sslFile
```

### 第三步：配置nginx

在conf.d文件夹中创建新的配置文件，一般情况下反代的每一个服务都创建一个新的.conf文件

结构为：nginx/conf.d/server1.conf server2.conf

以下配置为个人常用配置模板可以直接使用修改域名和服务端口号即可

以下配置，如果你没有什么自定义的其他需求直接复制到自己创建的反向代理文件用就可以

```bash
# 强制HTTP重定向到HTTPS（Force SSL）
server {
    listen 80;
    listen [::]:80;
    server_name exp.example.com;        # 改为您的域名
    return 301 https://$host$request_uri;  # 自动跳转HTTPS
}

# 主HTTPS服务配置
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;                      #开启http2支持
    server_name exp.example.com;

    # SSL证书配置
    ssl_certificate sslFile/example.com/fullchain.cer;     # 替换为您的证书路径
    ssl_certificate_key sslFile/example.com/example.key;   # 替换为您的私钥路径
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # 安全防护配置（Block common exploits）
    add_header X-Frame-Options "SAMEORIGIN" always;     # 防点击劫持
    add_header X-XSS-Protection "1; mode=block" always; # 防XSS攻击
    add_header X-Content-Type-Options "nosniff" always; # 防MIME嗅探
    add_header Referrer-Policy "strict-origin" always;   # 控制referrer信息泄露
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always; # 强制HTTPS增强
    
    # 主请求代理配置
    location / {
        proxy_pass http://127.0.0.1:3001;   # 自定义后端服务地址
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}

```

nginx常用命令

```
# 检查nginx配置是否有错误
nginx -t

# nginx启动
nginx

# nginx重启
nginx -s reload
```

