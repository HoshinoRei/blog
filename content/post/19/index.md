---
title: 使用 NetworkManager 配置 Debian 路由器
author: 星野玲
date: 2023-10-01T08:00:00+08:00
description: 大约一年半前，小玲写了《使用 Debian 作为路由器》这篇文章。由于 Armbian 自带了 NetworkManager 而 Debian 没有自带 NetworkManager，所以当时小玲一开始就把 NetworkManager 卸载了，不过现在小玲打算用上 NetworkManager。
draft: false
tags:
  - Debian
  - 路由器
  - 软路由

---

大约一年半前，小玲写了《[使用 Debian 作为路由器]({{< ref "3.md" >}})》这篇文章。由于 Armbian 自带了 NetworkManager 而 Debian 没有自带 NetworkManager，所以当时小玲一开始就把 NetworkManager 卸载了，不过现在小玲打算用上 NetworkManager。

这次小玲使用的系统是通过虚拟机安装的 Debian 12，默认没有安装 NetworkManager。

## 安装 NetworkManager

首先安装 NetworkManager，如果需要 PPPoE 拨号，还需要安装 `ppp`。

```bash
sudo apt install -y network-manager ppp
```

## nmcli 命令的使用

### 查看网络情况

通过 `nmcli` 命令查看当前网络情况。

可以看到有两个连接外部的网络设备——`ens33` 和 `ens34`。`ens34` 设备已经通过 DHCP 获取了 IP 地址。`ens33` 还没有被设置。小玲将 `ens34` 设备当作 WAN 口，`ens33` 设备当作 LAN 口。

```text
ens34: connected to Wired connection 1
        "Intel 82545EM"
        ethernet (e1000), 00:0C:29:BD:87:34, hw, mtu 1500
        ip4 default, ip6 default
        inet4 192.168.█.9/26
        route4 192.168.█.0/26 metric 100
        route4 default via 192.168.█.1 metric 100
        inet6 240█:████:████:████:████:████:████:████/64
        inet6 fe80::90ec:3e41:215c:ac29/64
        route6 fe80::/64 metric 1024
        route6 240█:████:████:████::/64 metric 100
        route6 default via fe80::████:████:████:████ metric 100

lo: connected (externally) to lo
        "lo"
        loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536
        inet4 127.0.0.1/8
        inet6 ::1/128

ens33: unmanaged
        "Intel 82545EM"
        ethernet (e1000), 00:0C:29:BD:87:2A, hw, mtu 1500

DNS configuration:
        servers: 192.168.█.1
        interface: ens34

        servers: fe80::████:████:████:████
        interface: ens34

Use "nmcli device show" to get complete information about known devices and
"nmcli connection show" to get an overview on active connection profiles.

Consult nmcli(1) and nmcli-examples(7) manual pages for complete usage details.
```

### 查看网络设备

使用 `nmcli d` 命令查看网络设备。

现在 `ens33` 设备并没有被 NetworkManager 管理，待会会通过 NetworkManager 创建一个连接，使其被 NetworkManager 管理。

```text
DEVICE  TYPE      STATE                   CONNECTION
ens34   ethernet  connected               Wired connection 1
lo      loopback  connected (externally)  lo
ens33   ethernet  unmanaged               --
```

### 查看网络连接

使用 `nmcli c` 命令查看网络连接。

只有 `Wired connection 1` 这个 NetworkManager 自动创建的连接。

```text
NAME                UUID                                  TYPE      DEVICE
Wired connection 1  7d5979e9-c222-3fa0-a86e-9c1ea3f330b5  ethernet  ens34
lo                  609bc8f3-b78b-40c6-9836-9f67f0dab964  loopback  lo
```

### 创建以太网连接

在 `ens33` 设备上创建一个名为 `lan` 的以太网连接，这个连接的 IPv4 地址为 `192.168.3.1/24`。注意创建时需要 `sudo` 给权限。

命令里带了双引号的是参数的值，小玲这里是为了让大家看得更明白。

```bash
sudo nmcli c add type "ethernet" con-name "lan" ifname "ens33" ipv4.method "manual" ipv4.addresses "192.168.3.1/24"
```

如果需要设置静态 IPv6 地址，可以这么添加。

```bash
sudo nmcli c add type "ethernet" con-name "lan" ifname "ens33" ipv4.method "manual" ipv4.addresses "192.168.3.1/24" ipv6.method "manual" ipv6.addresses "fddd:afcf::/64"
```

### 查看连接详情

可以通过 `nmcli c show <connection_id|connection_uuid>` 查看连接的详情。

比如要查看刚刚添加的 `lan` 连接，可以通过下面的命令。

```bash
nmcli c show "lan"
```

### 忽略自动下发的 DNS 服务器地址

因为 NetworkManager 会控制 `/etc/resolv.conf` 文件。里面通常是通过 DHCP 获取的 DNS 服务器地址。如果不想被 NetworkManager 控制 `/etc/resolv.conf` 文件，可以通过下面的命令忽略自动下发的 DNS 服务器地址

```bash
sudo nmcli c modify <connection_id|connection_uuid> ipv4.ignore-auto-dns "yes" ipv6.ignore-auto-dns "yes"
```

### 删除一个连接

删除一个连接可以用 `nmcli c delete <connection_id|connection_uuid>` 命令。

如果要把刚刚创建的 `lan` 连接删除，就可以使用下面的命令。

```bash
sudo nmcli c delete "lan"
```

### 添加 PPPoE 连接

如果要通过 `ens34` 设备使用 PPPoE 拨号，可以创建这么一个连接。

`username` 和 `password` 的值，请填写自己的宽带账号和密码。

```bash
sudo nmcli c add type "pppoe" con-name "pppoe" ifname "ens34" username "" password ""
```

添加成功后，通过下面的命令启动刚刚添加的 `pppoe` 连接。

```bash
sudo nmcli c up pppoe
```

值得注意的是，通过 `nmcli c add` 添加的所有连接，都是默认开机自启动的。所以 PPPoE 的连接也是开机自启动的。

注意在 Debian 上使用 PPPoE 拨号的前提是安装 `ppp`。如果还没有安装 `ppp`，请趁着能联网时安装 `ppp`。

```bash
sudo apt install -y ppp
```

### 获取 IPv6 PD 前缀

如果要获取 PD 前缀，并把它添加到 `lan` 连接上，可以使用下面的命令。

```bash
sudo nmcli c modify "lan" ipv6.method "shared"
```

通过 NetworkManager 获取 PD 前缀时，几乎没有可以自定义的选项，例如前缀长度，路由器在 `lan` 连接上的主机地址。不过小玲只能获取一个 `/64` 前缀，所以不能配置也没有多大负面影响。

如果当前没有 PD 前缀，重新启用 `pppoe` 连接以获取 PD 前缀。

```bash
sudo nmcli c down pppoe && sudo nmcli c up pppoe
```

#### 自动重新获取 PD 前缀

有时候，PPPoE 连接会自己断开重连，这时候如果 PD 前缀没有更新，会导致局域网内的设备无法使用 IPv6。

小玲为了解决这个问题，用了个野路子，思路就是写一个脚本，当 PPPoE 连接断开时，自动禁用并启用 PPPoE 连接以达到重新获取 PD 前缀的目的。

```bash
sudo vim /etc/ppp/ipv6-down.d/restart_pppoe
```

```bash
#!/usr/bin/bash

# CONNECTION_NAME 变量的值为 NetworkManager 里用于 PPPoE 拨号的连接名称
CONNECTION_NAME=pppoe
nmcli c down $CONNECTION_NAME && nmcli c up $CONNECTION_NAME
```

### 添加一个带 VLAN ID 的连接

通过下面的命令来添加一个带 VLAN ID 的连接。

`ifname` 的值可以自由设置，这里小玲就设置为 `ens33.10`。

`vlan.parent` 的值是这个 VLAN 连接依附的网络设备的名称。

`vlan.id` 的值可以自由设置，这里小玲就设置为 `10`。

```bash
sudo nmcli c add type "vlan" ifname "ens33.10" vlan.parent "ens33" vlan.id "10" ipv4.method "manual" ipv4.addresses "192.168.3.1/24"
```

### 添加一个网桥并开启 WiFi 热点

如果路由器有多个网络设备，可以通过创建一个网桥将它们桥接起来。比如现在有一个 USB 无线网卡，要把它和 LAN 口桥接起来，LAN 提供有线连接，USB 无线网卡开启热点后提供无线连接。那么可以这么设置。

注意，如果 LAN 口上已经有连接了，需要先把 LAN 口的连接删除。

先用 `nmcli d` 查看 USB 无线网卡的设备名称。

```text
DEVICE                   TYPE      STATE                   CONNECTION
ens34                    ethernet  connected               Wired connection 1
lo                       loopback  connected (externally)  lo
ens33                    ethernet  unmanaged               --
wlx908d78eb1c7b          wifi      disconnected            --
p2p-dev-wlx908d78eb1c7b  wifi-p2p  disconnected            --
```

USB 无线网卡的设备名称为 `wlx908d78eb1c7b`。

通过下面的命令创建一个名为 `bridge` 的网桥，`ifname` 的值可以自由设置，这里小玲就设置为 `br0`。

```bash
sudo nmcli c add type "bridge" con-name "bridge" ifname "br0" ipv4.method "manual" ipv4.addresses "192.168.3.1/24"
```

将 `ens33` 设备加入刚刚创建的网桥。

```bash
sudo nmcli c add type "bridge-slave" ifname "ens33" master "br0"
```

通过下面的命令创建一个无线的连接。

`wifi.band` 是 WiFi 的频段，值为 `a` 或 `bg`。无论哪个都是特别慢的，`nmcli` 并不支持 WiFi 5 甚至 WiFi 6。

`wifi.ssid` 的值请填写自己想设置的 SSID。

`802-11-wireless-security.key-mgmt "wpa-psk"` 和 `802-11-wireless-security.proto "rsn"` 这两个参数将 WiFi 加密方式设置为 WPA2。

`802-11-wireless-security.psk` 的值请填写自己想设置的密码。

```bash
sudo nmcli c add type "wifi" ifname "wlx908d78eb1c7b" master "br0" wifi.band "a" wifi.mode "ap" wifi.ssid "" 802-11-wireless-security.key-mgmt "wpa-psk" 802-11-wireless-security.proto "rsn" 802-11-wireless-security.psk ""
```

小玲并不建议大家通过 NetworkManager 开启 WiFi 热点，因为速度实在是太慢了，如果有这方面的需求，小玲的建议是买一个无线路由器通过网线连接 LAN 口后当作 AP。

如果要使用 Tproxy 透明代理，小玲也不建议设置网桥。因为 nftables 对网桥的数据的处理有些问题。小玲在这里只是介绍这个方法而已，并不是建议大家使用网桥。

### 连接 WiFi

通过 `nmcli d wifi list ifname ""` 命令查看无线网卡，`ifname` 的值请填写自己的无线网卡设备名。如果只有一张无线网卡，那么这个命令可以省略为 `nmcli d wifi`。

通过 `nmcli d wifi connect "" password "" ifname ""` 命令连接 WiFi，`connect` 的值请填写要连接的 WiFi 的 SSID，`password` 的值请填写 WiFi 的密码，`ifname` 的值请填写自己的无线网卡设备名。如果只有一张无线网卡，那么 `ifname` 及其后面的内容可以省略。

## 结尾

这篇文章更多的是一篇补充性而不是教程性的文章，所以没看过小玲之前的文章的话有可能看不懂。如果看不懂请看《[使用 Debian 作为路由器]({{< ref "3.md" >}})》和《[Debian 软路由 + 华硕AX86U 实现多个 WiFi VLAN 隔离]({{< ref "7.md" >}})》。
