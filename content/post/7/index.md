---
title: Debian 软路由 + 华硕AX86U 实现多个 WiFi VLAN 隔离
author: 星野玲
date: 2022-08-10T13:29:00+08:00
description: 小玲有这样一个需求，一个无线路由器能发出多个 SSID 的 WiFi，并且连接一个 SSID 的 WiFi 的设备与连接另一个 SSID 的 WiFi 的设备之间的通信可以隔离。这里的隔离不是“禁止无线用户互通”，而是要 VLAN 隔离。
draft: false
tags:
  - Debian
  - 华硕AX86U
  - 路由器
  - 软路由

---

小玲有这样一个需求，一个无线路由器能发出多个 SSID 的 WiFi，并且连接一个 SSID 的 WiFi 的设备与连接另一个 SSID 的 WiFi 的设备之间的通信可以隔离。这里的隔离不是“禁止无线用户互通”，而是要 VLAN 隔离。

这样的好处是比如家里来了客人的话，可以让客人连接上客人专用的 WiFi。因为客人用的 WiFi 与小玲专用的 WiFi 之间是 VLAN 隔离的，客人就无法访问到小玲的局域网里的 NAS 之类的设备。或者小玲有一些智能家居设备，把这些智能家居设备放进一个局域网里，然后与小玲专用的局域网之间隔离，可以防止一些潜在的隐私窥探问题。

以前小玲用过一个能刷集客 AP 固件的路由器，集客 AP 固件可以在一个路由器里创建多个 SSID，并且还能设置 VLAN ID。不过现在没见过能刷集客 AP 固件的 WiFi 6 路由器了。好在小玲买的 AX86U 能通过一个野路子实现小玲的需求。小玲很庆幸买对了路由器，要是买了 TP-Link 估计就要跟这个功能说拜拜了。

## 设备介绍

首先，先介绍一下要使用的设备。R4S 是小玲目前在用的软路由，刷了 Armbian 系统，它的设置请看 [这篇文章]({{< ref "3.md" >}})。[AX86U]({{< ref "6.md" >}}) 刷了梅林固件，小玲不清楚原版固件能不能实现这个功能，如果不能请像小玲一样刷梅林固件。

## 网段规划

然后规划一下网段。小玲原本在 R4S 里给整个局域网设置的 IPv4 网段为 `192.168.3.0/24`。小玲决定把它拆分成 4 个子网。它们分别如下。

|网段|第一个可用地址|最后一个可用地址|用途|
|:---:|:---:|:---:|:---:|
|`192.168.3.0/26`|`192.168.3.1`|`192.168.3.62`|小玲专用的局域网|
|`192.168.3.64/26`|`192.168.3.65`|`192.168.3.126`|家人专用的局域网|
|`192.168.3.128/26`|`192.168.3.129`|`192.168.3.190`|访客专用的局域网|
|`192.168.3.192/26`|`192.168.3.193`|`192.168.3.254`|智能家居专用的局域网|

上面每个子网的可用 IP 数都是 62，这个数量对小玲来说够用了。

小玲用的宽带只能获取到一个 IPv6 的 `/64` 前缀，小玲打算将这个前缀用于小玲专用的局域网。另外 3 个局域网就无法使用公网 IPv6 地址了。虽然没有公网 IPv6 地址，但是小玲还可以设置 ULA[^1]，然后通过 NAT，让另外 3 个局域网获得访问 IPv6 地址的能力。使用这个 [网站](https://cd34.com/rfc4193/) 可以生成一个 ULA 的网段，小玲生成的网段是 `fddd:afcf::/48`，小玲决定拿出这里面的前 3 个 `/64` 网段来分别给另外 3 个局域网使用。为什么要用 `/64` 这么大的网段呢？因为比 `/64` 更小的网段就无法通过 SLAAC 分配 IP 地址，也就是说安卓设备无法被分配到这个网段的 IP 地址了。[^2]

[^1]: 参考：[为你的 IPv6 局域网配置 ULA 吧](https://www.v2ex.com/t/488116)

[^2]: 参考：[现在运营商分配 IPV6 地址前缀是/64 吗？](https://www.v2ex.com/t/510276)。

|ULA 网段|用途|
|:---:|:---:|
|不需要|小玲专用的局域网|
|`fddd:afcf::/64`|家人专用的局域网|
|`fddd:afcf:0:1::/64`|访客专用的局域网|
|`fddd:afcf:0:2::/64`|智能家居专用的局域网|

每个网段在 R4S 里的地址都被小玲指定为了第一个可用地址，也就是说，R4S 现在拥有 4 个局域网的 IPv4 地址，3 个局域网的 IPv6 地址，它们分别如下。

- 192.168.3.1
- 192.168.3.65
- 192.168.3.129
- 192.168.3.193
- fddd:afcf::
- fddd:afcf:0:1::
- fddd:afcf:0:2::

## VLAN ID 规划

网段规划完了，现在要规划 VLAN ID，VLAN ID 不仅在 R4S 里用到 AX86U 里也要用到。

|VLAN ID|用途|
|:---:|:---:|
|无|小玲专用的局域网|
|10|家人专用的局域网|
|20|访客专用的局域网|
|30|智能家居专用的局域网|

## 网桥规划

默认情况下，AX86U 里只有一个网桥，名字叫 `br0`。`br0` 网桥的成员默认就是所有的接口。小玲因为有 4 个局域网，所以需要 4 个网桥。网桥的名称可以自定义，这里小玲就用 `br1`、`br2` 等作为网桥名吧。

|网桥|用途|
|:---:|:---:|
|`br0`|小玲专用的局域网|
|`br1`|家人专用的局域网|
|`br2`|访客专用的局域网|
|`br3`|智能家居专用的局域网|

## 设置网段与 VLAN ID

网段和 VLAN ID 规划好后，现在就需要在 R4S 里设置网段和 VLAN ID 了。

在 R4S 里编辑 `/etc/network/interfaces`。

```bash
sudo vim /etc/network/interfaces
```

R4S 的 LAN 口的接口名称是 `enp1s0`，在 `enp1s0` 后面加上一个 `.<数字>` 表示这个接口是一个以这个数字为 VLAN ID 的接口。例如 `enp1s0.10` 这个接口的 VLAN ID 是 `10`，`enp1s0.20` 这个接口的 VLAN ID 是 `20`。

```text
auto enp1s0
allow-hotplug enp1s0
iface enp1s0 inet static
    address 192.168.3.1/26

auto enp1s0.10
allow-hotplug enp1s0.10
iface enp1s0.10 inet static
    address 192.168.3.65/26
iface enp1s0.10 inet6 static
    address fddd:afcf::/64

auto enp1s0.20
allow-hotplug enp1s0.20
iface enp1s0.20 inet static
    address 192.168.3.129/26
iface enp1s0.20 inet6 static
    address fddd:afcf:0:1::/64

auto enp1s0.30
allow-hotplug enp1s0.30
iface enp1s0.30 inet static
    address 192.168.3.193/26
iface enp1s0.30 inet6 static
    address fddd:afcf:0:2::/64
```

## 设置 Dnsmasq

还需要设置 Dnsmasq 以给每个网段的设备自动分配 IP 地址。小玲使用的 Dnsmasq 是通过 [这篇文章]({{< ref "3.md" >}}) 里安装的 Dnsmasq。

在 R4S 里编辑 `dnsmasq.conf`。

```bash
vim ~/config/dnsmasq/dnsmasq.conf
```

下面的内容已省略无关配置。

```text
# 监听这 4 个接口
interface=enp1s0
interface=enp1s0.10
interface=enp1s0.20
interface=enp1s0.30

dhcp-range=interface:enp1s0,192.168.3.2,192.168.3.62,1h
dhcp-range=interface:enp1s0,::,constructor:enp1s0,ra-stateless,1h
dhcp-option=interface:enp1s0,option:router,192.168.3.1
dhcp-option=interface:enp1s0,option:dns-server,192.168.3.1 
# 这里的 fe80::362f:579d:8c86:105b 请换成你的本地链路地址
dhcp-option=interface:enp1s0,option6:dns-server,[fe80::362f:579d:8c86:105b]

dhcp-range=interface:enp1s0.10,192.168.3.66,192.168.3.126,1h
dhcp-range=interface:enp1s0.10,::,constructor:enp1s0.10,ra-stateless,1h
dhcp-option=interface:enp1s0.10,option:router,192.168.3.65
dhcp-option=interface:enp1s0.10,option:dns-server,192.168.3.65
dhcp-option=interface:enp1s0.10,option6:dns-server,[fddd:afcf::]

dhcp-range=interface:enp1s0.20,192.168.3.130,192.168.3.190,1h
dhcp-range=interface:enp1s0.20,::,constructor:enp1s0.20,ra-stateless,1h
dhcp-option=interface:enp1s0.20,option:router,192.168.3.129
dhcp-option=interface:enp1s0.20,option:dns-server,192.168.3.129
dhcp-option=interface:enp1s0.20,option6:dns-server,[fddd:afcf:0:1::]

dhcp-range=interface:enp1s0.30,192.168.3.194,192.168.3.254,1h
dhcp-range=interface:enp1s0.30,::,constructor:enp1s0.30,ra-stateless,1h
dhcp-option=interface:enp1s0.30,option:router,192.168.3.193
dhcp-option=interface:enp1s0.30,option:dns-server,192.168.3.193
dhcp-option=interface:enp1s0.30,option6:dns-server,[fddd:afcf:0:2::]
```

## 设置 WiFi

设置完 R4S，接下来就要设置 AX86U 了。先在 AX86U 里设置 WiFi。小玲专用的 WiFi 就在 `无线网络` 里设置，家人专用的 WiFi、访客专用的 WiFi、智能家居专用的 WiFi 在 `访客网络` 里设置。同时请把 AX86U 的操作模式设为 `无线接入点（AP）模式`。

在这里有一个问题需要弄明白，那就是在 AX86U 的系统里，网络接口是以 `eth0`、`eth1` 这样的字符串表示的。咱们需要知道哪个网口和哪个 WiFi 对应哪个接口。好在小玲找到一份对应关系。[^3] 不过这个对应关系不包含访客网络的接口。小玲又自己测试了一下，整理出了以下的对应关系。

[^3]: 来源：[VLANs, Trunk interface, tagged and untagged traffic RT-AX86U and RT-AX88U](https://www.snbforums.com/threads/vlans-trunk-interface-tagged-and-untagged-traffic-rt-ax86u-and-rt-ax88u.78411/)。

|物理网口或 WLAN 接口|系统内的接口名称|
|:---:|:---:|
|WAN|`eth0`|
|LAN4|`eth1`|
|LAN3|`eth2`|
|LAN2|`eth3`|
|LAN1|`eth4`|
|2.5G LAN|`eth5`|
|2.4GHz WLAN|`eth6`|
|5GHz WLAN|`eth7`|
|2.4GHz WLAN 访客网络1|`wl0.1`|
|2.4GHz WLAN 访客网络2|`wl0.2`|
|2.4GHz WLAN 访客网络3|`wl0.3`|
|5GHz WLAN 访客网络1|`wl1.1`|
|5GHz WLAN 访客网络2|`wl1.2`|
|5GHz WLAN 访客网络3|`wl1.3`|

访客网络在下图中有两行三列，从左到右分别以 1、2、3 来表示。例如 2.4GHz 的第一列对应的系统内接口名称是 `wl0.1`，5GHz 的第二列对应的系统内接口名称是 `wl1.2`。

{{< figure src="Snipaste_2022-08-08_18-25-11.png" position="center">}}

## 设置 WiFi 的 VLAN

清楚了对应关系后，咱们就可以通过 SSH 进入 AX86U 的命令行里设置 VLAN 了。首先请确保 AX86U 的 SSH 功能是开启的，在 `系统管理` → `系统设置` → `服务` → `启用 SSH` 里可以设置。

下面的方法小玲参考了 Github Gist 上的[一段代码](https://gist.github.com/Jimmy-Z/6120988090b9696c420385e7e42c64c4)，在此感谢这段代码的作者。

首先增加带 VLAN ID 的接口。这里使用 `eth0` 是因为小玲把连接着 R4S 的网线接到了 AX86U 的 WAN 口上了，而 WAN 口对应的系统内接口名称是 `eth0`。

```bash
ip link add link eth0 name eth0.10 type vlan id 10
ip link add link eth0 name eth0.20 type vlan id 20
ip link add link eth0 name eth0.30 type vlan id 30
```

通过 `ip link` 可以看到咱们刚添加的接口。

```text
29: eth0.10@eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default
    link/ether ██:██:██:██:██:██ brd ff:ff:ff:ff:ff:ff
30: eth0.20@eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default
    link/ether ██:██:██:██:██:██ brd ff:ff:ff:ff:ff:ff
31: eth0.30@eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default
    link/ether ██:██:██:██:██:██ brd ff:ff:ff:ff:ff:ff
```

可以看到咱们刚添加的接口都是 `DOWN` 的。因为光添加还不行，咱们还需要启用这些接口，把它变成 `UP` 的。

```bash
ip link set eth0.10 up
ip link set eth0.20 up
ip link set eth0.30 up
```

通过 `ip link` 可以看到咱们刚添加的接口已经启用了。

```text
29: eth0.10@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether ██:██:██:██:██:██ brd ff:ff:ff:ff:ff:ff
30: eth0.20@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether ██:██:██:██:██:██ brd ff:ff:ff:ff:ff:ff
31: eth0.30@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether ██:██:██:██:██:██ brd ff:ff:ff:ff:ff:ff
```

通过 `brctl show` 查看所有的网桥。

```text
admin@RT-AX86U-████:/tmp/home/root# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.████████████       no              eth0
                                                        eth1
                                                        eth2
                                                        eth3
                                                        eth4
                                                        eth5
                                                        eth6
                                                        eth7
                                                        wl0.1
                                                        wl0.2
                                                        wl0.3
                                                        wl1.1
                                                        wl1.2
```

可以看到默认只有一个网桥 `br0`，还需要添加三个网桥。

```bash
brctl addbr br1
brctl addbr br2
brctl addbr br3
```

添加网桥后也要像上面启用接口一样启用网桥。

```bash
ip link set br1 up
ip link set br2 up
ip link set br3 up
```

再用 `brctl show` 查看，就可以看到已经添加了三个网桥。

```text
admin@RT-AX86U-████:/tmp/home/root# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.████████████       no              eth0
                                                        eth1
                                                        eth2
                                                        eth3
                                                        eth4
                                                        eth5
                                                        eth6
                                                        eth7
                                                        wl0.1
                                                        wl0.2
                                                        wl0.3
                                                        wl1.1
                                                        wl1.2
br1             8000.000000000000       no
br2             8000.000000000000       no
br3             8000.000000000000       no
```

现在需要从 `br0` 网桥中删除不属于它的接口，需要用 `brctl delif` 这个命令，它的语法如下。

```bash
brctl delif <bridge> <device...>
```

```bash
brctl delif br0 wl0.1 wl1.1 wl0.2 wl0.3 wl1.2
```

```text
admin@RT-AX86U-████:/tmp/home/root# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.████████████       no              eth0
                                                        eth1
                                                        eth2
                                                        eth3
                                                        eth4
                                                        eth5
                                                        eth6
                                                        eth7
br1             8000.000000000000       no
br2             8000.000000000000       no
br3             8000.000000000000       no
```

小玲把 `2.4GHz WLAN 访客网络1` 和 `5GHz WLAN 访客网络1` 当作家人专用的 WiFi；`2.4GHz WLAN 访客网络2` 和 `5GHz WLAN 访客网络2` 当作访客专用的 WiFi，`2.4GHz WLAN 访客网络3` 当作智能家居专用的 WiFi。所以小玲需要把 `eth0.10`、`wl0.1`、`wl1.1` 这三个接口添加进 `br1` 网桥；`eth0.20`、`wl0.2`、`wl1.2` 添加进 `br2` 网桥；`eth0.30`、`wl0.3` 添加进 `br3` 网桥。

把接口添加进网桥需要用 `brctl addif` 这个命令，它的语法如下。

```bash
brctl addif <bridge> <device...>
```

```bash
brctl addif br1 eth0.10 wl0.1 wl1.1
brctl addif br2 eth0.20 wl0.2 wl1.2
brctl addif br3 eth0.30 wl0.3
```

```text
admin@RT-AX86U-████:/tmp/home/root# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.████████████       no              eth0
                                                        eth1
                                                        eth2
                                                        eth3
                                                        eth4
                                                        eth5
                                                        eth6
                                                        eth7
br1             8000.████████████       no              eth0.10
                                                        wl0.1
                                                        wl1.1
br2             8000.████████████       no              eth0.20
                                                        wl0.2
                                                        wl1.2
br3             8000.████████████       no              eth0.30
                                                        wl0.3
```

这时候已经成功了，小玲试着连接上家人专用的 WiFi，可以看到被分配的 IPv4 地址正好在 `192.168.3.64/26` 内，连接上访客专用的 WiFi，IPv4 地址正好在 `192.168.3.128/26` 之内，连接上智能家居专用的 WiFi，IPv4 地址正好在 `192.168.3.192/26` 之内。

## 自动化

现在已经实现了想要的功能，接下来就要让 AX86U 自动帮咱们完成上面这一整套步骤。小玲为此写了一个脚本。要想保存这个脚本，请先打开 AX86U 设置中的 `系统管理` → `系统设置` → `Persistent JFFS2 partition` → `Enable JFFS custom scripts and configs`。

AX86U 的系统里没有 vim，所以咱们只能用 vi。

```bash
vi /jffs/scripts/vlan
```

写入以下内容

```bash
#!/bin/sh

# 这个变量的值是连接软路由的那个网口的系统内的接口名称，小玲用的是 WAN 口，所以是 eth0。
lan_ifname=eth0

# 这个函数的作用是增加并启用带有 VLAN ID 的接口，如果要添加的接口已存在就什么也不做。
# 形参1：接口名称
# 形参2：VLAN ID
# 例如 "add_vlan_if eth0 10" 会在 "eth0.10" 这个 VLAN ID 为 10 的接口不存在时添加这个接口。 
function add_vlan_if() {
    ip link | grep $1.$2 > /dev/null
    if [ $? == 1 ]; then
        ip link add link $1 name "${1}.${2}" type vlan id $2
    fi
    ip link show $1.$2 | grep DOWN > /dev/null
    if [ $? == 0 ]; then
        ip link set "${1}.${2}" up
    fi
}

# 这个函数的作用是增加并启用网桥，如果要添加的网桥已存在就什么也不做。
# 形参1：网桥名称
# 例如 "add_br br1" 会在 "br1" 这个网桥不存在时添加这个网桥。 
function add_br() {
    brctl show | grep $1 > /dev/null
    if [ $? == 1 ]; then
        brctl addbr $1 > /dev/null
    fi
    ip link show $1 | grep DOWN > /dev/null
    if [ $? == 0 ]; then
        ip link set $1 up > /dev/null
    fi
}

# 这个函数的作用是把一个接口添加到网桥，如果要添加的接口已经在这个网桥里了就什么也不做。
# 形参1：网桥名称
# 形参2：接口名称
# 例如 "add_if_to_br br1 eth0.10" 会在 "eth0.10" 这个接口不在 "br1" 网桥里时，将 "eth0.10" 接口添加进 "br1" 网桥。 
function add_if_to_br() {
    brctl show $1 | grep $2 > /dev/null
    if [ $? == 1 ]; then
        brctl addif $1 $2 > /dev/null
    fi
}


# 这个函数的作用是把一个接口从一个网桥里删除，如果要删除的接口不在这个网桥里就什么也不做。
# 形参1：网桥名称
# 形参2：接口名称
# 例如 "delete_if_from_br br1 eth0.10" 会在 "eth0.10" 这个接口在 "br1" 网桥里时，将 "eth0.10" 接口从 "br1" 网桥里删除。 
function delete_if_from_br() {
    brctl show $1 | grep $2 > /dev/null
    if [ $? == 0 ]; then
        brctl delif br0 $2 > /dev/null
    fi
}

# 下面的语句都是调用上面的函数了。

# 添加并启用 "eth0.10"、"eth0.20" "eth0.30" 三个接口。
add_vlan_if $lan_ifname 10
add_vlan_if $lan_ifname 20
add_vlan_if $lan_ifname 30

# 添加并启用 "br1"、"br2"、"br3" 三个网桥。
add_br br1
add_br br2
add_br br3

# 把用到的访客网络的接口从 `br0` 网桥里删除。
delete_if_from_br br0 wl0.1
delete_if_from_br br0 wl0.2
delete_if_from_br br0 wl0.3
delete_if_from_br br0 wl1.1
delete_if_from_br br0 wl1.2

# 添加接口到相应的网桥里。
add_if_to_br br1 $lan_ifname.10
add_if_to_br br1 wl0.1
add_if_to_br br1 wl1.1
add_if_to_br br2 $lan_ifname.20
add_if_to_br br2 wl0.2
add_if_to_br br2 wl1.2
add_if_to_br br3 $lan_ifname.30
add_if_to_br br3 wl0.3
```

可惜 AX86U 的 Shell 是那种超简陋的 Shell，连数组都不能定义，不然小玲可以写数组遍历的。

给脚本添加执行权限。

```bash
chmod +x /jffs/scripts/vlan
```

为了让 AX86U 每次开机时都执行一遍这个脚本，咱们需要在 `/jffs/scripts/services-start` 这个文件里添加这么一行。

```bash
/jffs/scripts/vlan
```

这样 AX86U 每次开机时就会自动配置好 VLAN、网桥什么的。不过有时候咱们更改了 AX86U 的一些设置都会导致网桥和接口复原，这时候只能进 SSH 手动执行一下脚本来恢复配置，或者用一个笨办法，那就是用定时任务每分钟执行一遍脚本来自动恢复设置。这个问题小玲暂时找不到很好的解决方法，如果你知道解决方法，不妨在评论区告诉小玲。

## 防火墙配置

现在小玲连接上访客专用的 WiFi（`192.168.3.128/26` 网段），并试图去 ping 处于 `192.168.3.0/26` 的 NAS 时，发现是可以 ping 通的，这样是不行的。小玲的想法是小玲专用的局域网里的设备能主动访问其它三个局域网里的设备，其他三个局域网的设备只能主动访问它自己网内的设备。用表格表示如下。

|访问方\被访问方|小玲专用的局域网|家人专用的局域网|访客专用的局域网|智能家居专用的局域网|
|:---:|:---:|:---:|:---:|:---:|
|小玲专用的局域网|✓|✓|✓|✓|
|家人专用的局域网|✗|✓|✗|✗|
|访客专用的局域网|✗|✗|✓|✗|
|智能家居专用的局域网|✗|✗|✗|✓|

这个需求可以在 R4S 上通过 nftables 实现。在 `filter` 表里的 `forward` 链里添加这么一句就能禁止其它三个局域网里的设备主动访问小玲专用的局域网里的设备。同时小玲专用的局域网里的设备能正常访问其他三个局域网里的设备。

```bash
sudo vim /etc/nftables.conf
```

```text
# 已省略无关内容
table inet filter {
    chain forward {
        type filter hook forward priority 0;
        iifname enp1s0.10 oifname { enp1s0, enp1s0.20, enp1s0.30 } ct state { new, invalid } drop
        iifname enp1s0.20 oifname { enp1s0, enp1s0.10, enp1s0.30 } ct state { new, invalid } drop
        iifname enp1s0.30 oifname { enp1s0, enp1s0.10, enp1s0.20 } ct state { new, invalid } drop
    }
}
```

## IPv6 访问问题

小玲用的宽带只能获取到一个 `/64` 的前缀，这个前缀已经用于小玲专用的局域网，让它里面的设备都有公网 IPv6 地址了。

由于没有更多的前缀，所以其它三个局域网里的设备无法获得公网 IPv6 地址。虽然如此，小玲还是至少要让其它三个局域网里的设备能正常访问 IPv6 地址，这时候就需要在 R4S 里用 nftables 做 IPv6 的 NAT。

```bash
sudo vim /etc/nftables.conf
```

```text
# 已省略无关内容
table ip6 ipv6-nat {
    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        iifname { enp1s0.10, enp1s0.20, enp1s0.30 } oifname ppp0 masquerade
    }
}
```

`ppp0` 是小玲的 R4S PPPoE 拨号用的接口名称，请根据实际情况换成你的接口名称。

## 结尾

好了，现在终于实现小玲的需求了。其实这个需求不用 AX86U 也能实现，一般能刷梅林的路由器应该都能像这篇文章一样操作。小玲并不是什么精通计算机的大佬，可能讲的不是很专业和严谨，如果有错误欢迎在评论区指出，在此感谢。
