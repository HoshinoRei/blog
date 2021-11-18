---
title: 华硕AX86U 鬼灭之刃版伪开箱
author: 星野玲
date: 2022-08-08T15:12:00+08:00
description: WiFi 6 路由器出来已经有 2 年了，但是小玲直到 2022 年 还没有用上 WiFi 6，所以小玲一直很想买个 WiFi 6 路由器。
draft: false
tags:
  - 华硕AX86U
  - 开箱
  - 路由器

---

WiFi 6 路由器出来已经有 2 年了，但是小玲直到 2022 年 还没有用上 WiFi 6，所以小玲一直很想买个 WiFi 6 路由器。

小玲目前选路由器的标准有两个。

1. 价格不能太贵。2000 以上的路由器小玲都没法买，因为小玲家里没矿。
2. 这路由器呀，性能不能太烂。小玲家里没有埋网线，路由器都是放在客厅里的。虽说可以在地上拉一条网线到小玲的房间，但是这样子太丑。所以小玲希望在客厅和小玲的房间之间的网络用无线连接，而且速度是越快越好。几百块的便宜路由器可能速度会很慢。

小玲没买 AX86U 前看中了两款路由器，TP-Link TL-XTR10280 和华硕 RT-AX86U 鬼灭之刃版，小玲还看了好多这两款路由器的评测，小玲根据自己的需求，整理出了这两款路由器的对比。

|对比项\路由器|TP-Link TL-XTR10280|华硕 RT-AX86U 鬼灭之刃版|
|:---:|:---:|:---:|
|价格|较低|较高|
|性价比|较高|较低|
|无线性能|较高，毕竟是三频。如果未来再买一台 XTR10280，或许能实现小玲的房间和客厅之间拥有千兆的无线速度|较低，只有双频|
|可玩性|较低，因为 TP-Link 只给 16M 闪存，没法刷机，而且官方固件没有小玲想要的功能|较高，有梅林固件，几乎可以当作 Openwrt 来玩|
|外观|较差|较好，二次元皮肤加成，而且因为是立式的，占地面积还小|

为什么小玲希望小玲的房间和客厅之间的速度越快越好呢？因为小玲有个 NAS 放在了客厅，小玲希望能高速地存取文件。拉网线的话虽然能达到千兆，但是明线的缺点小玲已经说过了。把 NAS 放在小玲的房间的话电源和噪音是个问题，所以只能寄希望于无线了。

经过激烈的思想斗争，小玲决定买了 AX86U。虽说华硕的产品都很贵，但是小玲相信华硕的产品应该不会差到哪里去，于是就有了这篇文章。

## 开箱

这是外包装的六个面。

{{< figure src="20220703_135357_01.jpeg" caption="外包装正面" position="center">}}

{{< figure src="20220703_135831.jpeg" caption="外包装背面" position="center">}}

{{< figure src="20220703_135759.jpeg" caption="外包装侧面" position="center">}}

{{< figure src="20220703_135901.jpeg" caption="外包装侧面" position="center">}}

{{< figure src="20220703_135928.jpeg" caption="外包装侧面" position="center">}}

{{< figure src="20220703_140002.jpeg" caption="外包装侧面" position="center">}}

拆开外包装后，是一个纯白的纸箱。

{{< figure src="20220703_140116.jpeg" position="center">}}

打开内包装。

{{< figure src="20220703_140246.jpeg" position="center">}}

{{< figure src="20220703_140327.jpeg" position="center">}}

先看看这些说明书。

{{< figure src="20220703_140504.jpeg" caption="App 设置指南" position="center">}}

{{< figure src="20220703_140616.jpeg" caption="高级无线网络设置小帮手" position="center">}}

{{< figure src="20220703_140655.jpeg" caption="快速使用指南" position="center">}}

{{< figure src="20220703_140728.jpeg" caption="快速使用指南背面" position="center">}}

{{< figure src="20220703_140821.jpeg" caption="保修卡" position="center">}}

再来看看配件。

电源是 19.5V 2.31A，大约 45W 的电源。电源线是三孔的，过于普通就不展示了。

{{< figure src="20220703_141332.jpeg" caption="电源" position="center">}}

还有三根天线和一个小鸟的小牌子。

{{< figure src="20220703_141437.jpeg" caption="天线和牌子" position="center">}}

配件看完了，接下来是路由器本体。

{{< figure src="20220703_141533.jpeg" caption="路由器正面" position="center">}}

{{< figure src="20220703_141712.jpeg" caption="路由器顶面" position="center">}}

这一面有一个 WPS 按钮。

{{< figure src="20220703_141754.jpeg" caption="路由器侧面" position="center">}}

这一面有一个 LED 灯光按钮，用于开关路由器正面的指示灯，小玲喜欢让这些灯开着。

{{< figure src="20220703_141859.jpeg" caption="路由器侧面" position="center">}}

背面从左到右分别是电源输入口、开关、重置按钮、两个 USB 3.0 接口、一个 2.5G网口、一个千兆 WAN 口和四个千兆 LAN 口。其中 LAN1 口是游戏网口。

{{< figure src="20220703_141954.jpeg" caption="路由器背面" position="center">}}

把天线装上。

{{< figure src="20220703_142740.jpeg" caption="装好天线后的路由器" position="center">}}

## 配置

给路由器通上电、然后 WAN 口连接小玲的 R4S 后，电脑搜到了一个 `ASUS` 开头的 WiFi。

{{< figure src="Snipaste_2022-07-03_15-00-31.png" position="center">}}

浏览器也自动打开了这个网页。

{{< figure src="Snipaste_2022-07-03_15-01-55.png" position="center">}}

点击 `创建新的网络` 后。

{{< figure src="Snipaste_2022-07-03_15-03-17.png" position="center">}}

填自己喜欢的 SSID 和密码。小玲这里就用原来的 SSID。

{{< figure src="Snipaste_2022-07-03_15-03-52.png" position="center">}}

WiFi 6 是必须要开的。

{{< figure src="Snipaste_2022-07-03_15-04-02.png" position="center">}}

设置路由器后台的用户名和密码。

{{< figure src="Snipaste_2022-07-03_15-04-19.png" position="center">}}

还提示有新的固件，升就升吧。

{{< figure src="Snipaste_2022-07-03_15-04-29.png" position="center">}}

{{< figure src="Snipaste_2022-07-03_15-04-39.png" position="center">}}

{{< figure src="Snipaste_2022-07-03_15-06-19.png" position="center">}}

{{< figure src="Snipaste_2022-07-03_15-06-58.png" position="center">}}

经典的华硕固件登录页面。

{{< figure src="Snipaste_2022-07-03_15-08-52.png" position="center">}}

登录后，也是经典的华硕路由器后台页面，只不过有了我妻善逸的画。左边还有一些 `网易 UU 加速器` 之类的功能。不过这些小玲都用不到，因为小玲是要把这款路由器当 AP 来用的。开了 AP 模式后，左边的很多功能都用不了了。

{{< figure src="Snipaste_2022-07-03_15-09-12.png" position="center">}}

看一下系统信息，据说 AX86U 用的是和 AX89X 同款的四核 SoC，1 GB 内存。

{{< figure src="Snipaste_2022-07-03_15-09-35.png" position="center">}}

有意思的是，当你用支持 WiFi 6 的网卡连接了 WiFi 6 的 WiFi 后，Windows 会有这样一个提示。

{{< figure src="Snipaste_2022-07-09_18-52-47.png" position="center">}}

## 刷机并提高信号强度

用华硕路由器的人大概都知道把地区从 `中国` 更改为 `澳大利亚` 能提高信号强度吧？但其实还有个方法能把信号再提高一点，那就是刷 koolshare 的固件后装 `wifi boost` 插件。小玲已经刷好了 koolshare 的固件，刷的过程就不展示了。`wifi boost` 插件是要收费的，价格是 30 元，一个激活码只能给一台路由器用。小玲也买了。

{{< figure src="Snipaste_2022-08-07_19-20-41.png" position="center">}}

`中国` 地区的 WiFi 功率限制是 100mw。据说 `澳大利亚` 地区的功率能达到 300mw，利用 `wifi boost`，小玲直接把功率拉到了 595.66mw。小玲不敢再往上拉了，如果烧坏了路由器就得不偿失了。

小玲的房间距离路由器直线大约有 7 米的距离，中间还隔了半堵墙和一扇木门。就算是这样，小玲在房间里用手机连接 5GHz 的 WiFi，手机显示的信号依然能满格，只不过连接速度没有满。

## 测速

小玲将电脑放在房间里，通过 AX200 网卡连接到客厅的路由器的 5GHz WiFi，并且在路由器里设置了一个周围都没人用的信道，通过 iPerf3 与 R4S 进行测速。测速时只有电脑一台设备连接了 5GHz WiFi，测速结果如下。

|线程数\方向|R4S -> 电脑|电脑 -> R4S|
|:---:|:---:|:---:|
|1|368 Mbits/s|408 Mbits/s|
|2|506 Mbits/s|531 Mbits/s|

这个速度，小玲并不是很满意。虽然这个速度肯定是比小玲以前用 WiFi 5 高的，但还是离无线千兆比较远。

## 结尾

好了，这次开箱就到这里。其实 AX86U 小玲早就买了，也用了一段时间了，所以这不是第一次开箱，小玲姑且把它称作伪开箱吧。这篇文章小玲早就想写了，无奈太忙，终于今天把它写出来了。这也是小玲第一次做开箱，以后还会有更多的开箱的，那么咱们下篇文章见吧。
