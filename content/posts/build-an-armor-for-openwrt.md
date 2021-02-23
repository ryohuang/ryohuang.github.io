---
title: "我给Openwrt造盔甲"
date: 2021-02-23T12:55:40+08:00
draft: true
# weight: 1
# aliases: ["/first"]
tags: ["openwrt"]
author: "ryohuang"
showToc: false
TocOpen: false
hidemeta: false
comments: false
description: "不会做盔甲的码农不是一个合格的矿工"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
searchHidden: false
cover:
    image: "/img/armor-armor-m-tachaux-armoer-in-chief-of-metropolitan-museum-of-art-new-york-aeb860-1600.jpg"
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page

---

## 缘起

那一晚我不该看手机的。

不看手机就不会知道CDN矿工；不当CDN矿工就不会嫌弃家里的AC86U；不嫌弃AC86U就不会花巨资买个软路由；不买软路由就不会把软路由系统刷个遍；刷了一遍发现我就是想自己做（改）一个软路由系统——立刻马上。

## 现状

说到路由器上系统，特别是软路由，现在热门的无外乎闭源的如爱快、高恪，开源的Openwrt和它的子孙们。至于ROS,pfsense之类在国内稍显冷门的并不在讨论范围之内。对于路由器系统没有什么高低贵贱之分，适合自己，满足自己的需求就好。如果一定要争论个高低的话，就好像争论Debian、CentOS、Manjaro、OpenSUSE、和Ubuntu哪个更好一样，毫无意义。

我的使用体验是，想给各个软路由系统发好人卡——你很好，但是我们不适合。

### 爱快

国内论坛经常听说的一个系统。对于普通家庭用户来说很多功能是多余的。一屏幕的菜单我能用到的没几个。最近支持了kvm和docker。对于这两个功能，尤其是kvm，我颇有腹诽。一个路由器的系统竟然跑去作为kvm host?有这个硬件条件我肯定让爱快成为虚拟机中的guest!

#### 优点

* 背后有公司运作，界面美观，还有手机端
* 功能丰富

#### 缺点

* 闭源导致功能无法扩展和修改
* x64竟然要4G内存才能进行安装
* 曾经有过污点，坑过用户

### Openwrt

大名鼎鼎的Openwrt，国内很多路由器系统的爹，包括小米路由器的系统也是由它修改而来的。

#### 优点

* 开源且社区活跃
* 开源带来了最大的可扩展性

#### 缺点

* 默认系统太精简，太瘦了

### Lean's Openwrt

国内大名鼎鼎的Openwrt修改版,github上start数有15k，fork数13.1k。由于加入了很多中国特色的功能，国内社区流行程度很高。

#### 优点

* 适应国情

#### 缺点

* 代码感觉已经离Openwrt官方越来越远。内核版本已经5.x，但是软件源还是18.x。而官方19.x的版本还在用4.x的内核版本。这就导致安装ipk的时候很容易发生依赖冲突。

### Gargoyle

稍显另类的一个Openwrt衍生版本。它基于Openwrt的系统但是没有采用luci作为界面。曾经因为它的QoS功能而闻名，也作为我之前的路由器固件运行了好久。今时今日，很多人认为Openwrt上最好的QoS是SQM，但是至少他的代码给了我很大的启发。



## 理想

发完好人卡，我们来谈谈我理想的软路由系统。

## 盔甲

## 原理

## 尾声