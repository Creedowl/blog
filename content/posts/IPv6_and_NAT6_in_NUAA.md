---
title: 南航校园网OpenWRT配置IPv6 NAT6
date: 2019-04-04T23:29:39+08:00
draft: false
toc: true
tags:
 - network
 - handbook
---
> 当前IPv6越来越流行，不但解决了IPv4地址不够用的问题，还能给接入的设备提供公网访问的支持。但由于校园网的特殊性，虽然很早就有了IPv6支持，大部分学校在IPv6地址分配上却有剧毒，无法向下游设备分配独立的IPv6地址（正常家庭网络环境中路由器及下游设备都能自动分配独立的公网IPv6地址），以南航为例，OpenWRT上虽然获取到了2001开头，/64结尾的公网IPv6地址，但下游设备却无IPv6访问权限。本文介绍南航校园网环境下如何在OpenWRT下获取IPv6并配置NAT6使下游设备拥有IPv6访问权限。

## 0：软硬件需求

**硬件**：支持OpenWRT固件的路由器或软路由（推荐使用软路由，性能更好，支持启用更多服务，且不需要考虑太多兼容性问题，x86版本OpenWRT一把梭）

**软件**：较新版本的OpenWRT或其衍生版本（lede，pandorabox等，但原版OpenWRT最佳）

**OpenWRT内核需高于3.7**， linux kernel在3.7版本开始支持NAT6

~~内核版本不够高且无法升级可以尝试早年由北邮大佬开发的NAT66项目，但不推荐~~

所需软件包：

```shell
ip6tables kmod-ipt-nat6 kmod-ip6tables kmod-ip6tables-extra luci-proto-ipv6 iputils-traceroute6
```

较新版本已经自带IPv6支持，老版本需要自己安装相关软件包，但兼容性较差，或者内核版本不足，建议找较新版本或者自行编译。

## 1：OpenWRT配置获取IPv6

![IPv6_setting_1](http://ww1.sinaimg.cn/large/0071ouepgy1g1pisgc9yhj319v0p2tco.jpg)

### 1. 修改IPv6 ULA前缀为fd开头（默认为dd开头）
### 2. 修改wan6口配置（如下图所示）

![IPv6_setting_2](http://ww1.sinaimg.cn/large/0071ouepgy1g1pivtmkrbj30p00eumyi.jpg)

### 3. 修改lan口DHCP服务器配置（如下图所示）

![IPv6_setting_3](http://ww1.sinaimg.cn/large/0071ouepgy1g1piynr74fj30ta0adwfe.jpg)

![IPv6_setting_4](http://ww1.sinaimg.cn/large/0071ouepgy1g1pizho2fmj30nu0bymy5.jpg)

此时稍等片刻，在概况中应该就能看到获取到的IPv6地址及下游设备DHCPv6分配内网IPv6地址

![IPv6_setting_5](http://ww1.sinaimg.cn/large/0071ouepgy1g1pj4uxq0hj30rd08xaar.jpg)

![IPv6_setting_6](http://ww1.sinaimg.cn/large/0071ouepgy1g1pj5feonfj311806ldgn.jpg)

现在你的路由器已经可以使用IPv6了，但下游设备并不行，就需要配置NAT6。

## 2：配置NAT6

虽然NAT6并不符合IPv6的设计哲学，但却能解决一些问题，如安全隔离，或者像我们这样通过单一地址转发给多个设备使用。

### 1. 修改防火墙

![IPv6_setting_7](http://ww1.sinaimg.cn/large/0071ouepgy1g1pjfj32f5j30su0gvn00.jpg)

加入以下自定义规则

```shell
WAN6=pppoe-wan
LAN=br-lan
ip6tables -t nat -A POSTROUTING -o $WAN6 -j MASQUERADE
ip6tables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
ip6tables -A FORWARD -i $LAN -j ACCEPT
```

**注意这里WAN6和LAN要改为外网IPv6和内网网卡名字，在终端使用ifconfig查看**

![IPv6_setting_8](http://ww1.sinaimg.cn/large/0071ouepgy1g1pjk4cddbj30ll0jwq71.jpg)

### 2. 配置网关

   在终端执行 `ip -6 r` 查看默认网关（default开头），如果是类似

   `default from 2001:xxxx:xxxx:xxxx::/64 via fe80::ded2:fcff:fe98:64f7 dev pppoe-wan proto static metric 512 pref medium`

这样蛋疼的网关，需要去掉`from 2001:xxxx:xxxx:xxxx::/64` 这段再添加默认网关，例如

`ip -6 route add default via fe80::ded2:fcff:fe98:64f7 dev pppoe-wan proto static metric 512`

可以建立一个`/etc/hotplug.d/iface/99-ipv6`实现自动配置

```shell
#!/bin/sh
[ "$ACTION" = ifup ] || exit 0
iface=wan6
[ -z "$iface" -o "$INTERFACE" = "$iface" ] || exit 0
ip -6 route add `ip -6 route show default|sed -e 's/from [^ ]* //'`
logger -t IPv6 "Add IPv6 default route."
```

最后别忘了修改权限

`chmod +x /etc/hotplug.d/iface/99-ipv6`

重启路由器就可以愉快的玩耍了。



~~本来还想折腾IPv6端口转发，但一直无法配置成功。后来突然发现南航校园网下没有对IPv4做严格隔离（除了80端口基本上都能直接访问），就直接做IPv4端口转发配合DDNS，连接校园WIFI（nuaa.portal，**付费版本**）就可以直接使用，速度还挺快。。。就真香了。。不过配置IPv6端口转发还是有意义，比如可以从非校园网访问，校园网访问IPv6无需认证（不要充钱了），相关教程有空再补上（咕咕咕）有需求可以参考[ipv6 NAT后配置端口转发](https://shura.eu.org/2018/12/06/ipv6-NAT%E5%90%8E%E9%85%8D%E7%BD%AE%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/)~~



## 参考资料

> [清华OpenWRT 路由器作为 IPv6 网关的配置](<https://github.com/tuna/ipv6.tsinghua.edu.cn/blob/master/openwrt.md>)