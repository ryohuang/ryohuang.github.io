---
title: "我给Openwrt造盔甲"
date: 2021-02-23T12:55:40+08:00
draft: false
author: "阿黄"
authorLink: "https://ryohuang.github.io"
description: "不会做盔甲的码农不是一个合格的矿工"

tags: ["openwrt"]
categories: ["document"]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "/img/armor.jpg"
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: '<a rel="license external nofollow noopener noreffer" href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank">CC BY-NC 4.0</a>'
---


<!--more-->

## 缘起

那一晚我不该看手机的。

不看手机就不会知道CDN矿工；不当CDN矿工就不会嫌弃家里的AC86U；不嫌弃AC86U就不会花巨资买个软路由；不买软路由就不会把软路由系统刷个遍；刷了一遍发现我就是想自己做（改）一个软路由系统——立刻马上。

## 现状

说到路由器系统，特别是软路由，现在热门的无外乎闭源的如爱快、高恪，开源的Openwrt和它的子孙们。至于ROS,pfsense之类在国内稍显冷门的并不在讨论范围之内。对于路由器系统没有什么高低贵贱之分，适合自己，满足自己的需求就好。如果一定要争论个高低的话，就好像争论Debian、CentOS、Manjaro、OpenSUSE、和Ubuntu哪个更好一样，毫无意义。

我的使用体验是，想给各个软路由系统发好人卡——你很好，但是我们不适合。

评判之前先说下我的网络环境，尽管是主观感受但也要有客观前提。

* 上海电信千兆宽带+2路IPTV+固定电话
* 电信光猫拨号并设置DMZ给家庭网关
* 一台GEN8，更换E3 CPU,内存12G
* 一台TS-551，内存10G
* 一台AC 86U
* 一台四网口工控机，N4200+4G内存
* 若干智能设备，总计约30个设备接入

按照刷机顺序先说爱快。国内论坛经常听说的一个系统。对于普通家庭用户来说很多功能是多余的，一屏幕的菜单我需要用到的没几个。最近支持了kvm和docker。对于这两个功能，尤其是kvm，我颇有腹诽。一个路由器的系统竟然跑去作为kvm host?有这个硬件条件我肯定让爱快成为虚拟机中的guest!开头在Gen8上尝试虚拟机安装的时候，开头就让我郁闷了——X64版本强制要求4G内存才可以安装。再看看不完善的docker,而且因为闭源无法对它扩展和修改，当时就想换了它。翻翻网上的风评，之前竟然还干过坑用户的事情？还是回到开源世界吧，惹不起。

再来Openwrt。大名鼎鼎的Openwrt，国内很多路由器系统的爹，包括小米路由器的系统也是由它修改而来的。安装以后第一感受就是`瘦`。想要个Web UI都需要自己安装。官方原版如果直接让普通用户用，普通用户立刻自闭。而且，它不支持Fullcone NAT，upnp也支持不好。这两个功能对CDN挖矿是致命的，是我财富自由路上的拦路虎。换个符合国情的看看去！

很容易就找到了Lean's Openwrt，国内大名鼎鼎的Openwrt修改版,github上start数有15k，fork数13.1k。由于加入了很多中国特色的功能，国内社区流行程度很高。我自行编译了一个版本用了一段时间后发现了一些问题。Lean的代码感觉已经离Openwrt官方越来越远。Lean内核版本已经5.x，但是软件源还是18.x。而官方19.x的版本还在用4.x的内核版本。这就导致安装ipk的时候很容易发生依赖冲突。Lean那边看着也没有跟随官方版本升级的意思。眼看这今年Openwrt要出21.02了，我等不起啊。

最后我看了看老朋友Gargoyle的代码。稍显另类的一个Openwrt衍生版本。它基于Openwrt的系统但是没有采用luci作为界面。曾经因为它的QoS功能而闻名，也作为我之前的路由器固件运行了好久。今时今日，很多人认为Openwrt上最好的QoS是SQM而不是它了，但是至少他的代码组织方式和编译过程给了我很大的启发。

> 以上我只列举了我认为的缺陷和离开某个固件的理由，并没有任何指责和批评的意思。所有的开源项目都是在无私奉献，向作者们致敬。

## 理想

发完好人卡，我们来谈谈我理想的软路由系统。

Lean的系统很好，功能丰富，社区活跃。只是他的做法是直接在Openwrt上做修改，修改越来越多，也就离官方源越来越远。Openwrt现在已经在筹备Openwrt 21.02的发布，并且在未来luci将被luci2所替代。我很担心到时候Lean和Openwrt完全分道扬镳。之前Openwrt分支出去的LEDE，后来两者又合并的故事想必大家都有耳闻吧？

因为Lean对Openwrt的改动直接触及其筋骨，改出来的版本还很利害，这个过程像极了金刚狼的诞生。而我在想，能不能尽量保持Openwrt的原样，但是给他打造一幅盔甲。这套盔甲现在的Openwrt能够穿，稍微改动下将来的Openwrt(例如即将到来的21.02版本)也能穿。对，就像钢铁侠的盔甲一样。

## 盔甲

对着电脑一顿输出，盔甲造好了在[这里:slim-wrt](https://github.com/ryohuang/slim-wrt)。Slim-wrt意为给小小的Openwrt穿上盔甲，盔甲型号暂且算它是Mark I。

* 完全开源
* 不对Openwrt进行入侵式修改
* 基于Openwrt最新稳定版
* 优化Upnp
* Fullcone NAT
* 可以为特别设备设置特别的网关和DNS
* 明确的自定义项
* github workflows

### 完全开源

这个不用多说，代码全在github上了。

### 不对Openwrt进行入侵式修改

我没有直接fork一份Openwrt的代码修改。我把要改动的地方都分别做了补丁保存在以feature命名的文件夹中了，要用到的应用也在仓库中保存。在编译的时候对Openwrt的补丁被打入，自定义的应用通过自定义feed的方式合入整个版本中。

### 基于Openwrt最新稳定版

现在的工作分支是按照19.07执行的，之后会新增21.02。我相信大多数补丁和应用还能沿用。写完这一篇之后会新建branch去尝试Openwrt的新版本。

### 优化Upnp

upnp功能一直被用户诟病，Lean也对它采取了一些特殊处理，据说是停留在旧版本没有升级。我发现的问题是，在DMZ后一些CDN挖矿软件认为Upnp功能没有开启。对此我写了个简单的程序，就是仓库中的`slimapps/application/luci-app-boostupnp`。这部分之后会撰文说明。

### Fullcone NAT

Fullcone NAT默认不被Openwrt支持，而CDN挖矿需要它。相关的patch在Openwrt的论坛上早有讨论，但是人家不合啊！Fullcone NAT的补丁在[这里](https://github.com/Chion82/netfilter-full-cone-nat.git),Lean也是利用了这一份patch。

### 可以为特别设备设置特别的网关和DNS

看着很拗口的一个功能，因为一些无法说出口的原因。假设我家里有两个网关和DNS(为什么要有两个网关？不许问)。A直通国内，B直通世界。家里几乎所有设备要连A,少数设备连接B。这个app就是解决这个问题的——如何制定某些设备连接B。这个功能在`slimapps/application/luci-app-taghost`。
说到这里插一句，很多博主都吹双软路由，主路由的默认网关设置为副路由地址，副路由的默认网关设置为主路由的地址，DHCP由主路由提供。这神奇的玩法有没有什么不对劲？这哪里是主副路由啊，这是两个路由器串联，其中一个路由器挂了就全完了！

{{< figure src="/drawio/dual-routers.drawio.png" title="现在流行的双软路由下的数据走向" >}}

### 明确的自定义项

对于自定义功能，我做了两方面的努力。

1. 将功能相关的patch放在各个以功能名称命名的目录下。Openwrt的功能可能会涉及各个模块。这里带一下，Openwrt的很多功能都是依赖第三方`feeds`的（这也是国内编译麻烦的地方，很多第三方下不到啊！）所以我人为地以git仓库为依据将openwrt分为了openwrt和feeds。每个功能命名的目录下可以包含openwrt目录，用于对openwrt部分打补丁，feeds/*对feeds打补丁，除此之外如果有特殊需求可以在目录下建立custom.sh做一些额外动作。这样做的好处是清晰，易定制。再也不用到commit里的海洋里去大海捞针了。 

2. Openwrt的编译配置在.config里面，这个文件需要make menuconfig生成。对于默认配置和自定义功能，Lean的做法是修改target的makefile，而那个makefile里面又带了很多我用不到的东西（比如迅雷加速）。Gargoyle的做法是直接提供了.config文件，如果想自定义的话还是需要make menuconfig。我的做法是使用/profiles/slim/config这样的配置文件，再执行make defconfig完成.config的生成。这样的配置文件简单易读易配置，也容易传播。

所以，当你想自定路由器功能的时候，可以看看patch-modules下的目录然后再看看config文件,自定义的工作就完成了。

### github workflows

是的，我另外写了workflows用于固件的编译和版本的发布。我再也不想在本地编译了，而是利用美国大公司的服务器编。当完成自定义功能后，提交上去，打个tag，boom，你要的固件好了。很快啊！

## 原理

所谓一图胜千言，我放两张图。

{{< figure src="/drawio/openwrt.drawio.png" title="openwrt build process" >}}

上面是Openwrt原始的编译过程

{{< figure src="/drawio/slimbuild.drawio.png" title="slim build process" >}}

上面是slim-wrt的编译过程，原理都在图里了。流程都在下面了，代码都在github上了。

1. 加入自定义的feed，指向本项目的slimapps目录。具体每个app做什么会在其目录下另写文档，在此不再赘述。
2. 对openwrt应用补丁。遍历`patches-modules`下所有目录中，如果包含`openwrt`则应用其中的patch文件。例如`patches-modules/0001-fullcone-nat/openwrt`
3. 对`feeds`进行补丁。遍历`patches-modules`下所有目录中，如果包含`feeds`则应用其中的patch文件。
4. 配置每个项目的profile，拷贝到openwrt根目录下之后执行defconfig就会生成相应配置。
5. 配置生成后再额外补充一些内容到.config文件。
6. make，等待之后固件就生出来了。

## 尾声

终于，Openwrt穿上盔甲后CDN挖矿的网络环境满足了。看着100M上传带宽换来的每天4块钱感觉离财富自由越来越近了。不，其实这个根本不重要了。事实上，电费加上设备的损耗，我很可能是亏的。

重要的是有个想法，找找方法，然后真的做到了，自high一番再接着找下一个目标。slim-wrt并不想取代任何的路由器固件，只是有这么一种玩法我给公开了出来。

## 致谢

查阅资料的时候参考了各位大佬的内容，在此一并感谢。排名不分先后，全看复制黏贴。

* [Openwrt](https://openwrt.org)
* [gargoyle](https://www.gargoyle-router.com)
* [Lean](https://github.com/coolsnowwolf/lede)
* [jerrykuku](https://github.com/jerrykuku/luci-theme-argon.git)