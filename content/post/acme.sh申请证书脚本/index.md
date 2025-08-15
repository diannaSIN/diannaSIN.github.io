---
title: acme.sh申请证书脚本
description: 利用acme.sh申请公益的免费证书
#slug: hello-world
date: 2025-08-16 02:57:00+0000
#image: cover.jpg
categories:
    - web服务器
    - 网站证书
tags:
    - 网站相关
#weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

# acme.sh申请证书脚本

注意：

**基于debian12**

**基于bash，如果使用的是zsh需要自行研究是否需要转义**

**使用nginx做反向代理，部署证书**

**完全基于我个人的nginx使用习惯**

*简介：*

[*acme.sh](http://acme.sh) 是一个运行在linux终端的证书申请脚本，可以申请免费和付费/企业级两种证书。*

本笔记以申请ZeroSSL的证书为基准 acme.sh申请免费证书的机构虽然有5个，但是推荐的的机构只有三个，见下表

| CA名称 | ZeroSSL<br /> (acme.sh默认)                                  | Let's Encrypt <br />(使用最广泛，最推荐)                     | Google Public CA<br /> (谷歌提供的免费CA)                    |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 特点   | 90天有效期<br />✅ 通配符支持 <br />✅ 无速率限制 <br />🚫 通过邮箱或EAB强制验证 <br />✅自动续期不占用额度 <br />✅单证书验证域名数：100 | 90天有效期<br />✅ 通配符支持 <br />🚫速率：50张/域名/周<br />✅ 无需账户/邮箱验证<br />✅自动续期不占用额度<br />✅单证书验证域名数：100 | 90天有效期<br />✅ 通配符支持 <br />🚫速率：100张/小时 <br />🚫 通过GCP项目EAB强制认证 <br />✅自动续期不占用额度 <br />✅单证书验证域名数：100 <br />✅ 集成谷歌云服务 <br />🚫项目账户申请证书 <br />🚫限制：1000张/项目 |



## 安装与使用：

### 第一步：安装

如果是国内VPS，务必需要先执行这一行命令，修改一下host

```bash
echo -e "185.199.111.154 github.githubassets.com\\n140.82.113.22 central.github.com\\n185.199.108.133 desktop.githubusercontent.com\\n185.199.109.133 camo.githubusercontent.com\\n185.199.109.133 github.map.fastly.net\\n151.101.193.194 github.global.ssl.fastly.net\\n140.82.116.3  gist.github.com\\n185.199.110.153 github.io\\n140.82.116.4  github.com\\n140.82.116.5  api.github.com\\n185.199.110.133 raw.githubusercontent.com\\n185.199.110.133 user-images.githubusercontent.com\\n185.199.110.133 favicons.githubusercontent.com\\n185.199.109.133 avatars5.githubusercontent.com\\n185.199.110.133 avatars4.githubusercontent.com\\n185.199.108.133 avatars3.githubusercontent.com\\n185.199.108.133 avatars2.githubusercontent.com\\n185.199.109.133 avatars1.githubusercontent.com\\n185.199.111.133 avatars0.githubusercontent.com\\n185.199.111.133 avatars.githubusercontent.com\\n140.82.116.10 codeload.github.com\\n52.217.229.201  github-cloud.s3.amazonaws.com\\n52.216.185.51 github-com\\n.s3.amazonaws.com\\n52.217.225.81 github-production-release-asset-2e65be.s3.amazonaws.com\\n52.217.120.41 github-production-user-asset-6210df.s3.amazonaws.com\\n3.5.28.232  github-production-repository-file-5c1aeb.s3.amazonaws.com\\n185.199.111.153 githubstatus.com\\n185.199.109.133 media.githubusercontent.com\\n185.199.108.133 objects.githubusercontent.com\\n185.199.109.133 raw.github.com\\n138.91.182.224  copilot-proxy.githubusercontent.com" > /etc/hosts
```

安装必要软件

```bash
sudo apt install -y cron curl socat openssl git
```

通过git下载acme.sh包

```bash
git clone <https://github.com/acmesh-official/acme.sh.git>
```

进入安装目录

```bash
cd ./acme.sh
```

执行安装

```bash
./acme.sh --install                         # 不添加 -m 参数可以直接无邮箱安装（推荐）
./acme.sh --install -m myemail@example.com  # 添加 -m 参数带邮箱注册安装（谷歌CA必需）
```

重启终端以启用命令

```bash
source ~/.bashrc
```

启用脚本自动更新

```bash
acme.sh --upgrade --auto-upgrade
```

### 第二部：配置

**证书颁发机构配置**

*[acme.sh](http://acme.sh) 刚下载好之后默认的证书颁发机构为ZeroSSL，如果希望使用此机构那么不用做修改*

将默认证书颁发机构改为**Let's Encrypt**与**Google Public CA的命令如下**

```bash
acme.sh --set-default-ca --server letsencrypt  #将默认的证书颁发机构改成**Let's Encrypt**
acme.sh --set-default-ca --server google       #将默认的证书颁发机构改成**Google Public CA**
```

**证书颁发机构账号认证配置**

1. **Let's Encrypt**无需账号配置，不需要邮箱或者密钥认证
2. **ZeroSSL**账号配置*（推荐我的这种方式，尽量不要用邮箱）;* ps:需要先到ZeroSSL官网中注册账户，并在Developer中创建EAB Credentials for ACME Clients

```bash
acme.sh --register-account --server zerossl \\
--eab-kid xxxxxxxxxxxx \\
--eab-hmac-key xxxxxxxxx
```

1. **Google Public CA**账号配置

   1)需要先拥有一个Google Cloud Platform（GCP）账号，然后进入控制台左上角新建/选择一个项目

   2)在顶部搜索栏搜索并进入Public Certificate Authority API 之后点击启用

   3)再之后点击右上角shell图标进入Cloud Shell使用以下命令获取 EAB 密钥 ID 和 HMAC

   4)谷歌EAB密钥使用一次/七天后自动失效，所以每次申请证书的时候都要来获取一次密钥，但续费不用

```bash
gcloud config set project projectID           #进入项目设置
gcloud publicca external-account-keys create  #申请密钥
  5)在acme.sh中设置gcp账号和项目密钥
```

eab-kid是[申请到的 keyId]，eab-hmac-key是[申请到的 b64MacKey]

邮箱可以随意写，但是每一个邮箱代表一个acme.sh账户，谷歌每一个项目只能认证100个此账户

```bash
acme.sh --register-account -m myemail@example.com --server google \\
--eab-kid xxxxxxx \\
--eab-hmac-key xxxxxxx
```

### 第三部：申请证书

申请证书为了方便全都使用dns令牌的方式，以在cloudflare做dns为例

可以把dns令牌写入 `~/.acme.sh/account.conf`,但我推荐以下方式

其他常见dns供应商配置，参见笔记末尾表格

```bash
export CF_Token="your cf token"
```

申请证书（通配符证书）

```bash
[acme.sh](<http://acme.sh/>) --issue --dns dns_cf \\
-d *.example.com \\
-d example.com \\
--keylength ec-256
```

申请证书（单个二级域名证书）

```bash
[acme.sh](<http://acme.sh/>) --issue --dns dns_cf \\
-d exp[.example.com](<http://api.example.com/>) \\
--keylength ec-256
```

### 第四步：安装证书并部署

默认情况下证书存放在 ~/.acme.sh/example.com/目录下

申请好证书以后就该安装证书了，我一般用nginx反向代理使用证书，就以nginx为例

使用以下命令将证书与密钥安装到指定目录（sslFile是我自己在nginx目录下创建的）：

```bash
acme.sh --install-cert -d *.example.com --ecc \\
--key-file /etc/nginx/sslFile/example.com/example.key \\
--fullchain-file /etc/nginx/sslFile/example.com/fullchain.cer \\
--reloadcmd "systemctl reload nginx"
```

### 番外步：疑难杂症

如果在通过EAB密钥认证账户或者申请证书的时候遇到了这些报错：

```bash
Please refer to <https://curl.haxx.se/libcurl/c/libcurl-errors.html> for error code: 28
Please refer to <https://curl.haxx.se/libcurl/c/libcurl-errors.html> for error code: 35
```

那么使用以下命令调试一下，这可能是vps的问题，需要修复

```bash
# 查看当前 MTU
ip link show
# 临时设置较小 MTU
sudo ip link set dev eth0 mtu 1400
# 测试是否解决
curl -vI <https://acme.zerossl.com/v2/DV90> --tlsv1.2
```

如果你的证书需要用在CDN平台或者其他的非本地服务的平台，比如国内的七牛云CDN那么可以使用acme.sh的官方远程部署脚本：deployhooks。 [具体操作参见官方文档](https://github.com/acmesh-official/acme.sh/wiki/deployhooks)

### 一些acme.sh常用的命令：

手动强制更新证书

```bash
acme.sh --renew -d example.com -d *.example.com --force --ecc
acme.sh --renew -d example.com -d *.example.com --force        //非ECC证书使用此命令
```

查询域名申请证书的信息

```bash
acme.sh --list
```

主要命令参数列表

```bash
-d ：后面跟的是域名凡是需要写域名的地方，之前一定要写这个参数

--help 或 -h ：显示帮助信息

--version 或 -v ：显示版本信息

--upgrade ：检查更新

--list ：列出证书

--set-default-ca --server：设置默认的证书颁发机构

--server ：单次申请设置证书颁发机构

--issue ：申请证书

--renew 或 -r ：更新证书

--use-wget ：使用wget代替curl获取证书（不常用，当你正常申请证书出现error code:35的时候用）

--revoke ：吊销证书

--remove ：删除证书

--install-cert ：安装证书

--uninstall ：卸载acme.sh
```

### 常见DNS供应商 配置

| 机构               | dns参数       | token令牌命令（必须在申请证书之前操作）                      |
| ------------------ | ------------- | ------------------------------------------------------------ |
| **Cloudflare**     | dns_cf        | CF_Token="your cf token"                                     |
| 腾讯云             | dns_dp        | export DP_Id="123456"<br />export DP_Key="abcdef"            |
| 阿里云             | dns_ali       | export Ali_Key="123456"<br />export Ali_Secret="abcdef"      |
| 华为云             | dns_huawei    | export HUAWEICLOUD_Username="<Your IAM Username>"<br />export HUAWEICLOUD_Password="<Your Password>"<br />export HUAWEICLOUD_DomainName="<Your DomainName>" |
| GoDaddy            | dns_gd        | export GD_Key="<key>"<br />export GD_Secret="<secret>"       |
| DigitalOcean       | dns_dgon      | export DO_API_KEY="<key>”                                    |
| netcup             | dns_netcup    | export NC_Apikey="<Apikey>"<br />export NC_Apipw="<Apipassword>"<br />export NC_CID="<Customernumber>” |
| Vultr DNS          | dns_vultr     | export VULTR_API_KEY="<Your API key>"                        |
| AWS Route53        | dns_aws       | export AWS_ACCESS_KEY_ID="<key id>"<br />export AWS_SECRET_ACCESS_KEY="<secret>" |
| Google Cloud DNS   | dns_gcloud    | export CLOUDSDK_ACTIVE_CONFIG_NAME=default                   |
| Namecheap          | dns_namecheap | export NAMECHEAP_USERNAME="..."<br />export NAMECHEAP_API_KEY="..."<br />export NAMECHEAP_SOURCEIP="..." *# 必须添加* |
| Hurricane Electric | dns_he        | export HE_Username="<yourusername>"<br />export HE_Password="<password>" |
| Porkbun            | dns_porkbun   | export PORKBUN_API_KEY="..."<br />export PORKBUN_SECRET_API_KEY="..." |
