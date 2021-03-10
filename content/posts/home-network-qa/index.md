---
title: "家庭组网时我所面对的是非题、选择题以及简答题"
subtitle: ""
date: 2021-02-25T22:08:19+08:00
lastmod: 2021-02-25T22:08:19+08:00
draft: true
author: ""
authorLink: "https://ryohuang.github.io"
description: "slim-wrt的应用场景"

tags: ["openwrt","slim-wrt"]
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "img/network-2402637.jpg"
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: '<a rel="license external nofollow noopener noreffer" href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank">CC BY-NC 4.0</a>'

---

<!--more-->

## 前言

打完洞，布完线，光猫上象征性地测完速，在纸上留下联系方式和账号密码，宽带师傅和我说，好了，搞定了。但是，事实上，家庭网络还有一堆问题在等着我。

我的网络环境和设备如下：

- 上海电信十全十美千兆套餐(升级千兆时光猫换成了SDN设备)
- 2路电信IPTV
- 1路固定电话
- 1台AC 86u
- 1台威联通TS-551
- 1台HP Microserver Gen8
- 1台多网口小主机
- iMac/Laptop/智能设备若干
- 有非固定公网IP

## 宽带要不要桥接？

许多人第一要务是把光猫变桥接，由路由器拨号。如果你一个人住，IPTV和固话功能无所谓，不怕折腾，那我支持你把光猫桥接了。

事实是家里有老人爱看个电视，还连着固话，孩子还要时不时上网课，我还是不要桥接了。

光猫拨号至少有几个优点：
- 路由器和光猫独立，路由器的变化不会干扰到电视的收看和电话的使用。我那么爱折腾，还老是尝试新的路由器固件（最近自己在弄slim-wrt）,时不时会影响到家庭网络。如果我是桥接状态的话，我重启路由器家里的电视立刻就没法看了。
- 责任的划分更清晰。电信局端的问题和我家庭内部的问题可以很清晰地划分开。如果我发现上不去网了，只要连接光猫看看网络还是不是通的。如果光猫不通立刻就打电话保修就好了；如果光猫上的设备正常那就是我家庭内部网络的问题。让电信小哥清闲一点，在真的要让他解决问题的时候他才能马力全开——之前被变内网IP找小哥解决第二天就好了。
- 之后的升级可以不受干扰。之后如果我换其他运营商或者光猫设备升级，我的家庭网络可以保持原有的结构不变。

## 不桥接怎么做端口映射？


光猫拨号，下面再接路由器的话，家庭网络就有两级NAT了(第一次NAT在电信光猫，第二次NAT在家中主路由)。通常情况下，有几次NAT就要做几次端口映射。

{{< figure src="img/network.drawio.png" title="光猫拨号下的家庭网络" >}}

假如我想要在外网能够访问图中群晖的管理页面的话，需要做两次端口映射。
1. 在电信光猫上，开启tcp 5000，转发到IP 192.168.1.111
2. 在主路由上，开启tcp 5000，转发到IP 192.168.2.200

很显然，这太麻烦了。每个端口都要设置两次，之后改动的话也要改两个地方。我的做法是设置DMZ。

> 在一些家用路由器中，DMZ是指一部所有端口都暴露在外部网络的内部网络主机，除此以外的端口都被转发。——摘自维基百科

上海电信估计也是让搞光猫桥接的大爷们给整烦了，直接在配套的手机应用中加了设置**DMZ主机**的功能。

{{< figure src="img/dmz.jpeg" title="上海电信 DMZ主机设置" >}}

设置完DMZ后，家庭网络变成了下图的样子。（对，看起来和原来没有任何差别！）

{{< figure src="img/network-dmz.drawio.png" title="光猫拨号并设置DMZ的家庭网络" >}}

通俗的来讲，被设置为DMZ主机的那台设备会代替光猫成为家庭网络的对外代表。原来从外部而来的访问都是光猫处理，但是现在光猫直接就交给DMZ主机去处理了。
所以，现在我想要在外网能够访问图中群晖的管理页面的话，需要做一次端口映射就好了。

1. 在主路由上，开启tcp 5000，转发到IP 192.168.2.200

总结一下，开启DMZ给家里的主要路由器，使用体验上和光猫桥接路由器拨号没有太大差异。但是省去了光猫桥接模式下配置IPTV/固定电话的麻烦。

## DMZ下的upnp感觉有问题？

之前用Merlin系统的时候，一些CDN挖矿的软件经常报告我说Upnp没有开启。换了软路由后，刷了iKuai和Openwrt干脆一直说我Upnp没有开启了。

甜糖检测时候的日志如下：

```xml
2021-01-15 00:35:33.671 GetExternalIpAddress send POST /ctl/IPConn HTTP/1.1
HOST: 192.168.2.1:5000
Content-Length: 274
CONTENT-TYPE: text/xml;charset="utf-8"
SOAPACTION: "urn:schemas-upnp-org:service:WANIPConnection:1#GetExternalIPAddress"

<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
<s:Body>
<u:GetExternalIPAddress xmlns:u="urn:schemas-upnp-org:service:WANIPConnection:1">
</u:GetExternalIPAddress>
</s:Body>
</s:Envelope>

2021-01-15 00:35:33.677 GetExternalIpAddress recv = HTTP/1.1 200 OK
Content-Type: text/xml; charset="utf-8"
Connection: close
Content-Length: 357
Server: OpenWRT/OpenWrt UPnP/1.1 MiniUPnPd/2.0
Ext:

<?xml version="1.0"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"><s:Body><u:GetExternalIPAddressResponse xmlns:u="urn:schemas-upnp-org:service:WANIPConnection:1"><NewExternalIPAddress>192.168.2.1</NewExternalIPAddress></u:GetExternalIPAddressResponse></s:Body></s:Envelope>
2021-01-15 00:35:33.677 router ip = 192.168.2.1,external ip = 192.168.2.1
```
很明显，可以发现，**external ip = 192.168.2.1**，外部IP看着不对啊！

翻看一下miniupnpd的文档，有两项值得注意：

- **external_iface**:External interface. The default is to autodetect the first interface with a default route, which usually is wan.

- **external_ip**:Manually specified external IP - if not specified the default ipv4 address of the external interface is used.

很显然，这是DMZ带来的副作用。
再次回顾一下光猫拨号并设置DMZ的家庭网络结构。

{{< figure src="img/network-dmz.drawio.png" title="光猫拨号并设置DMZ的家庭网络" >}}

桥接后路由器拨号的话，路由器的WAN口可以正常地得到外网地址。但是在DMZ之后的路由器，它的WAN口获得的是光猫给分配的内网地址。被指定为DMZ主机的那个路由器，它接管了几乎所有的网络入口功能，它是事实上对外的唯一网关，但是它自己不知道！它依然认为自己只是一个卑微的二级路由器而已。

我的做法是，写个插件(包含在slim-wrt中，名字叫boostupnp)，让它知道自己多厉害，让它对外宣告事实上的外部IP地址。

- 指定external_iface和internal_iface
- 指定真实的external_ip

{{< figure src="img/d07f05b3c8354471a21f0104991892c8.png" title="boostupnp设置界面" >}}

使用插件之后upnpd配置如下，补全了原来没有的**external_ip**和**external_iface**

```shell
root@Slim:/# cat /etc/config/upnpd 

config upnpd 'config'
        option download '1024'
        option upload '512'
        option internal_iface 'lan'
        option port '5000'
        option upnp_lease_file '/var/run/miniupnpd.leases'
        option igdv1 '1'
        option ext_ip_reserved_ignore '1'
        option external_iface 'wan'
        option uuid '431ba314-25c0-40e2-bf58-b0bc07ebca1c'
        option clean_ruleset_threshold '90'
        option presentation_url 'http://192.168.2.1/'
        option enabled '1'
        option external_ip '124.77.xxx.xxx' #此处隐去真实IP

config perm_rule
        option action 'allow'
        option ext_ports '1024-65535'
        option int_addr '0.0.0.0/0'
        option int_ports '1024-65535'
        option comment 'Allow high ports'

config perm_rule
        option action 'deny'
        option ext_ports '0-65535'
        option int_addr '0.0.0.0/0'
        option int_ports '0-65535'
        option comment 'Default deny'
```

配合boostupnp，我的upnp环境终于得到了CDN挖矿软件的认可。

{{< figure src="img/upnp.png" title="来自挖矿软件的认可" >}}



## NAT type要选择哪种？

在Merlin系统上，NAT type有两种选择，全锥型和对称型。直观来说，任天堂Switch的网络检测里，NAT Type A对应的就是全锥形；B对应的是对称形。AB两种都能进行联机游戏，但是普遍认为A好于B，B好于C和D，C和D几乎没法玩。

{{< figure src="img/a3ea677160d14bc2b0e092398b2768b8.png" title="Switch网络测试" >}}

Play Station,XBox也有类似的要求。要求最严格的是那些CDN挖矿软件，他们希望有全锥型，如果挖矿的机器直接拨号更佳，甚至最好直接把矿机塞到电信机房去。

切换到了Openwrt后发现，它根本没有全锥型这个选项。Slim-wrt里把官方没加的Fullcone功能给加上了。

{{< figure src="img/3defe5cf1e1f496caeb27bd6a067b48e.png" title="Slim-wrt的NAT设置" >}}

总结来说，要玩联机游戏，有P2P(BT/PT)需求，尽量上全锥型。没有需求的话，哪种类型都无所谓。

## 有没有必要买软路由？

别买。千万别跟风买。一般路由器够用了。

除非你特别好学，经常要去“国外网站上学习先进技术”，家里正好没NAS，手机游戏机还都有“国外网站上学习先进技术”的需求。

更不要拿着国内这些软路由（其实是杂牌工控机，而且价格炒得很高）企图实现NAS+软路由的功能，根本不靠谱。

不过，我买了一个……只是因为不喜欢市面上所有的路由器固件，想自己做一个。这种需求下选择了一个低功耗，支持虚拟化的工控机。现阶段因为经常要测试新编译的固件，所以软路由系统我跑在虚拟机下。之后Openwrt 21.02发布后会直接裸机跑，并且适量地运行一些lxc/docker来充分利用它的硬件资源。

## 有没有必要做双软路由？

“双软路由”是个被吹得很玄乎的东西。本质上是家庭网络中的两个网关。

- 网关A：日常娱乐用，可以看看网页，打打游戏。
- 网关B：学习用，比如维护我的github页面，查查资料。

其中的`网关A`的角色就是由我们家里的路由器担当了。

而`网关B`的选择可以更多样化。

- 按照系统分可以是Openwrt这种router系统，也可以是Ubuntu/Debian/CentOS这些标准的Linux发行版。
- 按照运行环境分，它可以跑在真机下，kvm下，也可以选择跑在docker/lxc等容器下。

常见的网上教程中的“双软路由”我在《盔甲》一文中已经表述过，总的来讲就是不推荐，不要学。

## 有什么比双软路由更好的方法？

有。

{{< figure src="img/network-gateway.drawio.png" title="我所使用的两个网关" >}}

我的`网关A`就是我的主路由，家里几乎所有设备都连的它。我的`网关B`选择的系统是Ubuntu，并且是威联通上的lxc容器。不选择KVM是因为lxc更节省资源；不选择Openwrt是因为原版的方法更纯粹简单。（由于一些你知道的原因我不会过多描述`网关B`上到底跑了什么，怎么跑的）

在这样的网络下，连接到`网关A`的网络配置会是这样：

* gateway: 192.168.2.1
* dns: 192.168.2.1

需要连接到`网关B`的网络配置会是这样：

* gateway: 192.168.2.100
* dns: 192.168.2.100

那么如何指定某个设备走哪个网关呢？
最“傻”的方法就是手动指定，不用DHCP分配的网关和DNS。比如这样：

{{< figure src="img/static-ip.png" title="手动指定IP" >}}

稍微方便点是利用`dhcp option` （Merlin系统下的dnsmasq中设置）

```
# 指定需要特定网关的设备
dhcp-mac=free,1C:BF:C0:XX:XX:XX
# 指定网关，DNS
dhcp-option=free,3,192.168.2.100
dhcp-option=free,6,192.168.2.100
```

在slim-wrt下，你可以用我写的插件来做图形化的设定。

{{< figure src="img/taghost.png" title="taghost 插件" >}}

设置一个设备名，选择对应的MAC地址和TAG，保存。将这个设备先关闭Wifi再打开，让DHCP重新分配，这个设备就有了你指定的网关和DNS配置了。

## 一个设备怎么方便地在`网关A`和`网关B`之间切换？

我的手机平时常连的是`网关B`，但是偶尔队友们叫我开黑的时候我需要连接`网关A`。因为现在的手机都支持多频段（2.4G/5G）而且他们的MAC地址不同。可以利用这一点打到快速切换网关的目的。我将2.4G的MAC地址通过taghost设定了`网关B`，而5G频段下依然会有默认的`网关A`。

需要注意的是，现在的手机和windows都会有随机MAC和设备MAC的选项。请保证taghost中填入的是正确的设备MAC并且对应的设备上选择**使用设备MAC**
{{< figure src="img/device-mac.png" title="记得使用设备MAC" >}}

## 尾声

这是我在家庭组网时碰到的一些问题，归根到底都是设问句。自问自答推销一番自己的slim-wrt，见笑了😊。项目地址在https://github.com/ryohuang/slim-wrt 有兴趣的来看看啊。