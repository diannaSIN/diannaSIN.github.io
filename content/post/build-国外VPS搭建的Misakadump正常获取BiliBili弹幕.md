---
title: build-国外VPS搭建的Misakadump正常获取BiliBili弹幕
description: 在你的vps中搭建snell节点
#slug: snellbuild1
date: 2025-10-08T18:00:00+08:00
image: https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage20251008012849943.png
categories:
    - VPS配置
tags:
    - VPS配置
    - 服务器相关
#weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

#  在非中国大陆地区的VPS搭建Misaka_Dump并且获取BiliBili等国内平台的弹幕

假设你已经在国外的VPS搭建好了Misaka_Dump，现在没有办法获取国内平台的弹幕。这是由于版权等原因导致的IP限制。

### 配置前准备

1. 一个Tailscale账号。
2. 有一台国内的VPS、或者家里本地有一个可以打开终端的可以长期打开终端的设备如：istoreos，nas，PVE直接安装的debian12，或者有条件一直开着的设备都可以。
3. 注意：国内的家里设备可以不需要公网IP
4. 由于我是使用的家里的设备不是国内的VPS，所以以下均以家里设备代指国内设备

### 开始配置

#### 设备配置

1. 在[tailscale](https://tailscale.com)注册一个账号。

2. 进入控制台之后点击右边的Add device，并选择对应的平台。

   ![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage20251008005627314.png)

3. 如果你选择的是其他平台，包含你的Tailscale账号key的安装链接会发送到你的注册邮箱，如果你和我一样直接选择了linux，那么进来之后直接点击最下面的Generate install script。

   ![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage20251008005941897.png)

4. 之后你会获得一个安装命令，直接在你的家里的设备直接安装就可以了。

5. 之后在你家里的设备上配置家里设备允许作为网络出口，如果你和我一样是使用Linux debian12，那么直接输入以下命令就可以了

   ```
   sudo tailscale up --advertise-exit-node
   ```

6. 之后来到你网页中的Tailscale控制台，就可以看到你家里的这台设备出现在了控制台。然后按照下图提示在控制台中设置你的家里设备作为出口。

   ![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage20251008010537548.png)

   ![](https://raw.githubusercontent.com/diannaSIN/blogImage/image/BlogImage20251008010706949.png)

   上面就是家里设备的配置。   国外VPS和家里设备的配置一模一样但是不需要做第五步骤和第六步。同样也是在你的Tailscale控制台能看到你的国外VPS的设备就可以了。

做到这里，你在你的家里设备或者国外的VPS使用以下命令，

```
tailscale status
```

获取两台设备由Tailscale分配的虚拟内网IP，然后测试一下，如果可以ping通没有问题的话就可以进行下一步了。输出的结果就比如这样：

```
100.1.1.1   my-home-vm   linux   active
100.1.1.2   us-vps      linux   active
```

### 在家里设备搭建Socks5代理

如果你完成了上面的步骤，现在国内外两台设备已经可以通过Tailscale，现在只需要在家里设备搭建一个Socks5代理就可以了。（Misaka_dump系统设置里面可以添加socks5代理）

**如果你自己会搭建Socks5代理，那你自己搭建就可以了，在Misaka_dump设置中添加代理，并且在BiliBili设置中启用代理就行了。如果你不会搭建，那你就看一下以下的几个简单步骤**

注意确认好，是在你家里的设备安装Socks5，我这里以我家里的debian12为例

1. 下载一键脚本

   ```
   wget --no-check-certificate https://raw.github.com/Lozy/danted/master/install.sh -O install.sh
   ```

2. 执行安装

   ```
   bash install.sh --port=端口 --user=用户名 --passwd=密码
   ```

3. 这个过程可能比较慢需要几分钟当你的终端出现`Dante Server Install Successfuly`的时候就成功了，并且会输出你的连接配置。

到这里还没有结束，由于一键脚本的限制，它错误的把Tailscale分配给你的虚拟IP当成了公网访问IP，需要做一下修改。

在你的家里设备输入以下命令，查看你的网络接口

```
ip a
```

结果会给你列出几个网络接口，选择有你内网或者公网IP的那个接口，比如192.168.3.1，注意不是127.0.0.1的那个lo接口，一般是ens1、eth0等名称。复制下这个接口名称。 比如我的是ens18那我就复制ens18

然后输入以下命令修改你的socks5配置

```
vim /etc/danted/sockd.conf
```

在你的配置文件中 修改第四行的`external` ，把冒号后面那个Tailscale分配给你的虚拟ip删掉直接改成你的接口比如我的ens18. 然后保存退出

之后使用以下命令重启socks5服务。

```
service sockd restart
```



然后就能在misaka_dump来输入socks5代理信息并且使用了。注意在misaka_dump中要输入的socks代理的ip地址是你的Tailscale分配给你的虚拟ip，也就是通过`tailscale status`查询到的100开头的ip

