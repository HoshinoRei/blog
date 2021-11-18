---
title: 使用 Debian 作为路由器
author: 星野玲
date: 2022-02-19T22:35:00+08:00
description: 在这篇文章里，小玲将教你如何将一个装着干净的 Debian 系统的设备变成一个能让局域网内设备上网的软路由。
draft: false
tags:
  - Armbian
  - Debian
  - Nanopi R4S
  - 路由器
  - 软路由

---

在这篇文章里，小玲将教你如何将一个装着干净的 Debian 系统的设备变成一个能让局域网内设备上网的软路由。小玲使用一台 Nanopi R4S 作为演示用的设备，刷入的系统为 Armbian 21.08.1，Debian 版本为 11。你也可以用 X86 的设备，设置过程基本上没有区别。

将 Armbian 系统刷入 TF 卡后，咱们将卡插入 R4S 的 TF 卡槽里，使用一条网线接在原本的路由器的 LAN 口上，然后给 R4S 通电。

找到 R4S 通过 DHCP 获取的 IP 地址。用默认用户名 `root`，默认密码 `1234` 登录 SSH。在完成 Armbian 系统的初始设置后，咱们现在得到了一个干净的 Armbian 系统。下面的步骤都是用在 Armbian 系统的初始设置里创建的非 root 用户操作的。小玲不建议大家直接用 root 用户操作。

## 设置密钥

安装完后第一步是要确保路由器安全，因为当设置好后路由器是要暴露在公网上的。有人会扫描 IPv4 的 IP 段，然后再扫描能连通的 IP 地址的端口。如果发现是 SSH 端口，还会爆破。所以咱们要使用密钥文件登录，而且要把密码登录关闭。

创建存放密钥的文件夹，默认路径是家目录下的 `.ssh` 文件夹。

```bash
mkdir ~/.ssh
```

写入密钥。这里密钥请替换成你自己的。

```bash
echo 'ssh-rsa <private_key>' >> ~/.ssh/authorized_keys
```

编辑 sshd 配置文件。

```bash
sudo vim /etc/ssh/sshd_config
```

将 `PermitRootLogin` 设为 `no` 以关闭 root 账号登录；将 `PubkeyAuthentication` 设为 `yes` 以开启密钥登录；`AuthorizedKeyFile` 为密钥路径；将 `PasswordAuthentication` 设为 `no` 以关闭密码登录。

```text
PermitRootLogin no
PubkeyAuthentication yes
AuthorizedKeyFile .ssh/authorized_keys .ssh/authorized_keys2
PasswordAuthentication no
```

保存后，重启 SSH 服务。

```bash
sudo systemctl restart ssh
```

现在咱们可以在不关闭当前 SSH 连接的情况下新建一个用密钥登录的 SSH 连接。如果登录成功了，就可以断开用密码登录的 SSH 连接了。如果不出意外，现在就不能用密码登录了。

## 更新系统

如果是 Armbian 系统，请先阻止 Armbian 系统的更新。经过小玲实测，截止至 2022 年 12 月 27 日，Nanopi R4S 的 Armbian 系统最稳定的版本是 21.08.1。一些更新的版本都会有一些 bug，例如 LAN 口在系统里消失，WAN 口的接口名称在 `eth1` 和 `eth0` 之间反复横跳。

在 Armbian 系统下，编辑 `/etc/apt/sources.list.d/armbian.list` 文件。

```bash
sudo vim /etc/apt/sources.list.d/armbian.list
```

在这一行前面加个 `#`，注释这一行。

```text
#deb http://apt.armbian.com bullseye main bullseye-utils bullseye-desktop
```

再把 Debian 更新到最新，如果不是 Armbian 系统，直接做这一步就可以了。

```bash
sudo apt update
sudo apt upgrade -y
```

## 设置网络

查看一下网络配置文件

```bash
cat /etc/network/interfaces
```

如果是 Armbian 系统，默认内容如下。

```text
source /etc/network/interfaces.d/*
# Network is managed by Network manager
auto lo
iface lo inet loopback
```

可以看到现在网络的配置是由 NetworkManager 接管的，原本的 Debian 系统是不预装 NetworkManager 的，为了和原本的 Debian 系统设置方法统一，Armbian 系统要把 NetworkManager 卸载掉。这个步骤只是 Armbian 系统才有的，Debian 系统不需要做此步骤。

在卸载 NetworkManager 前，需要先把 `/etc/network/interfaces` 文件的配置写上。咱们先查看一下接口名称。

```bash
ip a
```

返回如下。小玲自己的 IP 地址和 MAC 地址已打码。

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether ██:██:██:██:██:██ brd ff:ff:ff:ff:ff:ff
    inet 192.168.███.███/24 brd 192.168.███.███ scope global dynamic noprefixroute enp1s0
       valid_lft 2338sec preferred_lft 2338sec
    inet6 240█:████:████:████:████:████:████:████/64 scope global dynamic noprefixroute
       valid_lft 3455sec preferred_lft 3455sec
    inet6 fe80::362f:579d:8c86:105b/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: eth1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether ██:██:██:██:██:██ brd ff:ff:ff:ff:ff:ff
```

现在咱们可以看到 LAN 口的接口名称是 `enp1s0`，WAN 口的接口名称是 `eth1`，LAN 口的 IPv6 本地链路地址是 `fe80::362f:579d:8c86:105b`，这个地址请记下来，待会可以用到。

编辑 `/etc/network/interfaces`

```bash
sudo vim /etc/network/interfaces
```

将文件改写成现在这个样子。小玲将 IPv4 网段设成了 `192.168.3.0/24`。当然网段你也可以改成你喜欢的。

```text
source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

auto enp1s0
allow-hotplug enp1s0
iface enp1s0 inet static
    address 192.168.3.1/24

auto eth1
allow-hotplug eth1
iface eth1 inet dhcp
iface eth1 inet6 auto
```

保存好以后就可以卸载 NetworkManager 了。

```bash
sudo apt remove -y network-manager
sudo apt autoremove -y
```

## 开启转发

编辑 `/etc/sysctl.conf` 文件。

```bash
sudo vim /etc/sysctl.conf
```

在 `/etc/sysctl.conf` 的末尾添加以下内容。

```text
net.ipv4.ip_forward=1
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
net.ipv6.conf.all.forwarding=1
net.ipv6.conf.default.forwarding=1
net.ipv6.conf.default.accept_ra=2
net.ipv6.conf.default.use_tempaddr=1
```

应用配置。

```bash
sudo sysctl -p
```

因为之前咱们更新了系统。现在最好重启一下，顺便应用新的网络设置。

```bash
sudo reboot
```

重启之后 R4S 的 LAN 口的 IP 地址已经变成了 `192.168.3.0/24` 这个网段了，不能再通过将 R4S 的 LAN 口连接原来的路由器的 LAN 口来获取原来的路由器的 IP 地址了。所以咱们需要使用一条网线连接着 R4S 的 LAN 口和电脑，一条网线连接的原本路由器的 LAN 口和 R4S 的 WAN 口。因为 R4S 上现在还没有 DHCP 服务，所以咱们需要把电脑的 IPv4 地址设为与 R4S 的 LAN 口的 IPv4 地址同一网段的 IPv4 地址。这里咱们就设成 `192.168.3.2`，子网前缀长度是 `24`，网关是 R4S 的 IP 地址 `192.168.3.1`。IPv6 先不用管。

设置完电脑的 IP 地址后，通过 SSH 连接到 `192.168.3.1` 来继续设置 R4S。

## 安装必要的软件

这些软件都是接下来要使用的。

```bash
sudo apt install -y nftables pppoeconf
```

使用 Docker 能减少软件的部署的难度。所以也安装上。

```bash
sudo wget -qO- https://get.docker.com/ | sh
```

安装 Docker Compose。

```bash
sudo wget "https://github.com/docker/compose/releases/download/v2.14.2/docker-compose-$(uname -s)-$(uname -m)" -O /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 配置 Dnsmasq

创建一个文件夹来存放 Dnsmasq 的配置文件。

```bash
mkdir -p ~/config/dnsmasq
```

编写 `dnsmasq.conf`。

```bash
vim ~/config/dnsmasq/dnsmasq.conf
```

写入以下内容。

```text
interface=enp1s0
port=53
server=8.8.8.8
server=8.8.4.4
enable-ra
log-dhcp
dhcp-range=192.168.3.2,192.168.3.254,1h
dhcp-range=::,constructor:enp1s0,ra-stateless,1h
dhcp-option=option:router,192.168.3.1
dhcp-option=option:dns-server,192.168.3.1
dhcp-option=option6:dns-server,[fe80::362f:579d:8c86:105b]
```

`interface` 是 LAN 口的接口名称。

`port` 是 R4S 上的 DNS 服务的端口，一般都要设为 `53`。

`server` 是上游 DNS 服务器的地址，这里就以谷歌的 DNS 服务器作为例子。

`enable-ra` 是用于向局域网内分发 IPv6 地址的。

`log-dhcp` 用于在日志中记录 DHCP 信息。

`dhcp-range` 是 DHCP 分发 IP 地址的范围，后面的 `1h` 是过期所需要的时间。

`dhcp-option=option:router` 是向局域网内宣告的 IPv4 网关地址。

`dhcp-option=option:dns-server` 是向局域网内宣告的 IPv4 DNS 服务器地址。

`dhcp-option=option6:dns-server` 是向局域网内宣告的 IPv6 DNS 服务器地址。

编辑 `docker-compose.yml` 文件，咱们使用 Dnsmasq 的 Docker 版来提供 DNS 和 DHCP 服务。

```bash
vim ~/docker-compose.yml
```

写入以下内容。

```yaml
version: "3"
services:
  dnsmasq:
    container_name: dnsmasq
    cap_add: 
      - NET_ADMIN
    image: hoshinorei/dnsmasq
    network_mode: host
    volumes:
      - ./config/dnsmasq/dnsmasq.conf:/etc/dnsmasq.conf:ro
    restart: always
```

这里的 Dnsmasq 容器必须要使用 `host` 网络模式，`bridge` 是不行的。

启动容器。

```bash
sudo docker-compose -f ~/docker-compose.yml up -d
```

现在把电脑的 IP 地址从静态改回 DHCP 的。咱们可以发现现在已经可以通过 DHCP 获取 IP 地址了。

## PPPoE 拨号

现在咱们可以开始连接外网了，咱们把连接 R4S 的 WAN 口的那条网线连接到光猫上。然后执行

```bash
sudo pppoeconf
```

这时候开始扫描哪个接口能连接 PPPoE。

```text
┌───────────────────┤ SCANNING DEVICE ├────────────────────┐
│ Looking for PPPoE Access Concentrator on enp1s0...       │
│                            66%                           │
└──────────────────────────────────────────────────────────┘
```

这里选择 `Yes`。

```text
┌────────────────────────┤ POPULAR OPTIONS ├─────────────────────────┐
│ Most people using popular dialup providers prefer the options      │
│ 'noauth' and 'defaultroute' in their configuration and remove the  │
│ 'nodetach' option. Should I check your configuration file and      │
│ change these settings where neccessary?                            │
│                  <Yes>                     <No>                    │
└────────────────────────────────────────────────────────────────────┘
```

这里输入宽带用户名。

```text
┌────────────────────┤ ENTER USERNAME ├────────────────────┐
│ Please enter the username which you usually need for the │
│ PPP login to your provider in the input box below. If    │
│ you wish to see the help screen, delete the username and │
│ press OK.                                                │
│                                                          │
│ ________________________________________________________ │
│                          <Ok>                            │
└──────────────────────────────────────────────────────────┘
```

这里输入宽带密码。

```text
┌────────────────────┤ ENTER PASSWORD ├────────────────────┐
│ Please enter the password which you usually need for the │
│ PPP login to your provider in the input box below.       │
│                                                          │
│ NOTE: you can see the password in plain text while       │
│ typing.                                                  │
│                                                          │
│ ________________________________________________________ │
│                          <Ok>                            │
└──────────────────────────────────────────────────────────┘
```

这里选 `No`，因为咱们已经有 Dnsmasq 了，咱们可以将路由器自身的 DNS 请求交给 Dnsmasq 处理。

```text
┌─────────────────────┤ USE PEER DNS ├─────────────────────┐
│ You need at least one DNS IP address to resolve the      │
│ normal host names. Normally your provider sends you      │
│ addresses of useable servers when the connection is      │
│ established. Would you like to add these addresses       │
│ automatically to the list of nameservers in your local   │
│ /etc/resolv.conf file? (recommended)                     │
│               <Yes>                  <No>                │
└──────────────────────────────────────────────────────────┘
```

这里选 `Yes`。

```text
┌──────────────────────┤ LIMITED MSS PROBLEM ├───────────────────────┐
│                                                                    │
│ Many providers have routers that do not support TCP packets with a │
│ MSS higher than 1460. Usually, outgoing packets have this MSS when │
│ they go through one real Ethernet link with the default MTU size   │
│ (1500). Unfortunately, if you are forwarding packets from other    │
│ hosts (i.e. doing masquerading) the MSS may be increased depending │
│ on the packet size and the route to the client hosts, so your      │
│ client machines won't be able to connect to some sites. There is a │
│ solution: the maximum MSS can be limited by pppoe. You can find    │
│ more details about this issue in the pppoe documentation.          │
│                                                                    │
│ Should pppoe clamp MSS at 1452 bytes?                              │
│                                                                    │
│ If unsure, say yes.                                                │
│                                                                    │
│ (If you still get problems described above, try setting to 1412 in │
│ the dsl-provider file.)                                            │
│                                                                    │
│                  <Yes>                     <No>                    │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

这里问咱们是否要在开机时自动拨号，作为一个路由器当然要在开机时自动拨号啦。所以咱们选 `Yes`。

```text
┌─────────────────────────┤ DONE ├─────────────────────────┐
│ Your PPPD is configured now. Would you like to start the │
│ connection at boot time?                                 │
│               <Yes>                  <No>                │
└──────────────────────────────────────────────────────────┘
```

这里告诉咱们可以使用 `pon dsl-provider` 开启连接；使用 `poff` 关闭连接。然后问咱们是否现在就开启连接？咱们选择 `Yes`。

```text
┌────────────────┤ ESTABLISH A CONNECTION ├────────────────┐
│ Now, you can make a DSL connection with "pon             │
│ dsl-provider" and terminate it with "poff". Would you    │
│ like to start the connection now?                        │
│               <Yes>                  <No>                │
└──────────────────────────────────────────────────────────┘
```

选择 `Ok`，结束配置。

```text
┌─────────────────┤ CONNECTION INITIATED ├─────────────────┐
│ The DSL connection has been triggered. You can use the   │
│ "plog" command to see the status or "ip addr show ppp0"  │
│ for general interface info.                              │
│                          <Ok>                            │
└──────────────────────────────────────────────────────────┘
```

现在可以查看一下所有接口的 IP 地址。

```bash
ip a
```

咱们可以看到多了一个名为 `ppp0` 的接口，并且成功获取了外网的 IP 地址。

```text
4: ppp0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1492 qdisc pfifo_fast state UNKNOWN group default qlen 3
    link/ppp
    inet ███.███.███.███ peer ███.███.███.1/32 scope global ppp0
       valid_lft forever preferred_lft forever
    inet6 240█:████:████:████:████:████:████:████/64 scope global temporary dynamic
       valid_lft 259127sec preferred_lft 85894sec
    inet6 240█:████:████:████:████:████:████:████/64 scope global dynamic mngtmpaddr
       valid_lft 259127sec preferred_lft 172727sec
    inet6 fe80::████:████:████:████ peer fe80::████:████:████:████/128 scope link
       valid_lft forever preferred_lft forever
```

咱们在 R4S 上 ping 一下外网的 IP，已经可以 ping 通。

```text
nanopi-r4s:~:% ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=19.9 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=116 time=19.8 ms
```

但是 ping 域名就不行了。查看一下 `/etc/resolv.conf`

```bash
cat /etc/resolv.conf
```

原来这里的 DNS 还是使用以前的路由器宣告的 DNS。

```text
# Generated by NetworkManager
nameserver 192.168.█.1
nameserver fe80::████:████:████:████%enp1s0
```

编辑 `/etc/resolv.conf`

```bash
sudo vim /etc/resolv.conf
```

将文件内容改为

```text
nameserver ::1
```

保存后，ping 域名也能 ping 通了，这是因为路由器自身的 DNS 请求已经交给自身的 Dnsmasq 处理了。所以为了路由器自身能正常上网，Dnsmasq 要一直运行着。

但是现在电脑还是不能 ping 通外网的 IPv4 地址 ，这是因为咱们没有做 IPv4 的 NAT 转换。

## 开启 NAT 转换

咱们使用 nftables 来开启 NAT 转换。

编辑 `/etc/nftables.conf` 文件。

```bash
sudo vim /etc/nftables.conf
```

里面已经写好了一些规则。

```text
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0;
    }
    chain forward {
        type filter hook forward priority 0;
    }
    chain output {
        type filter hook output priority 0;
    }
}
```

咱们把规则修改成这个样子。

```text
#!/usr/sbin/nft -f

table inet filter
delete table inet filter

table inet filter {
    chain input {
        type filter hook input priority 0;
        iifname "ppp0" tcp dport { 80, 443 } drop
        iifname "ppp0" udp dport { 53 } drop
    }
    chain forward {
        type filter hook forward priority 0;
    }
    chain output {
        type filter hook output priority 0;
    }
}

table ip ipv4-nat
delete table ip ipv4-nat

table ip ipv4-nat {
    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        oifname { eth1, ppp0 } masquerade
    }
}
```

咱们在 `filter` 表里的 `input` 链创建了一条规则 `iifname "ppp0" tcp dport { 80, 443 } drop`。这条规则的意思是从 `ppp0` 接口进来的访问 TCP 端口 80 和 443 的数据包全部禁止。设置这条规则的原因是在中国，家宽是不能私自做 WEB 服务的。咱们提前禁止这两个端口从外部的访问。之后就算在路由器上开了 WEB 服务。因为从外部访问不了，就不会有喝茶的风险了。

`iifname "ppp0" udp dport { 53 } drop` 这条规则的意思是禁止从 `ppp0` 接口进来的对 UDP 端口 53 的访问。因为咱们的 Dnsmasq 只是用来对局域网提供 DNS 服务，没有必要对外网提供 DNS 服务，所以顺便禁止了。

这里咱们创建了一个表 `ipv4-nat`。表名前面的 `ip` 意思是这个表只对 IPv4 有效。然后在表里新建了链 `postrouting`。`type nat` 的意思是这条链的类型是 `nat`；`hook postrouting` 的意思是这条链会被挂到 `postrouting` 钩子上；`priority srcnat` 的意思是这条链的优先级是 `srcnat`；`policy accept` 的意思是这条链的默认规则是“接受”。小玲这么说你可能不太理解，如果你理解 iptables，那么现在的 `ipv4-nat` 表中的 `postrouting` 链的功能就和 iptables 中的 `nat` 表的 `POSTROUTING` 链的功能差不多。

`oifname { eth1, ppp0 } masquerade` 的意思是从 `eth1` 和 `ppp0` 出去的数据包做 NAT 转换。

`table inet filter` 和 `delete table inet filter` 这两句的作用是在使用 `sudo systemctl reload nftables` 命令重载 nftables 规则时先把已有的表删除再添加，防止规则重复添加。

启动 nftables 服务，这样咱们写的规则就会被加载了，顺便将 nftables 设为自启。

```bash
sudo systemctl enable nftables
sudo systemctl start nftables
```

## 给内网分配 IPv6 地址

这时候虽然路由器有公网 IPv6 地址了，可是局域网的设备并没有公网 IPv6 地址。现在咱们要使用 wide-dhcpv6-client 来获取 PD 前缀，这样局域网的设备就能被分配到公网 IPv6 地址了。

创建一个文件夹来存放 wide-dhcpv6-client 的配置文件。

```bash
mkdir -p ~/config/wide-dhcpv6-client
```

编写 `dhcp6c.conf`。

```bash
vim ~/config/wide-dhcpv6-client/dhcp6c.conf
```

写入以下内容。

```text
interface ppp0 {
    send ia-pd 0;
};
id-assoc pd 0 {
    prefix-interface enp1s0 {
        sla-len 0;
        ifid 0;
    };
};
```

`interface ppp0` 这里的 `ppp0` 是连接外网的接口名称。

`send ia-pd 0` 的意思是发送获取 PD 的请求，并把这个 PD 的 ID 设为 `0`。

`id-assoc pd 0` 的意思是对 ID 为 `0` 的 PD 进行处理。

`prefix-interface enp1s0` 的意思是将 PD 前缀绑定到 `enp1s0` 接口上。这里的接口名称请替换成你的 LAN 口的接口名称。

`sla-len` 这个值要根据你能获取到的 PD 前缀长度来决定。公式是 `64 - 你能获取到的 PD 长度`[^1]。小玲只能获取到 `/64` 的前缀。所以小玲的 `sla-len` 的值为 `64-64=0`。

`ifid` 这个值可以用来固定 LAN 口接口的主机号，把 `ifid` 设为 `0` 并且 `sla-len` 是 `0` 的话，那么 LAN 口的公网 IPv6 地址将会是 `240█:████:████:████::`。如果不希望这个地址固定，还可以用 `if-id-random` 来替换这行。

[^1]: 来源：[X86 软路由配置 IPv6 踩坑小记](https://www.v2ex.com/t/724060)。

编辑 `docker-compose.yml` 文件，加入 wide-dhcpv6-client 的配置。

```bash
vim ~/docker-compose.yml
```

```yaml
version: "3"
services:
  dnsmasq:
    container_name: dnsmasq
    cap_add: 
      - NET_ADMIN
    image: hoshinorei/dnsmasq
    network_mode: host
    volumes:
      - ./config/dnsmasq/dnsmasq.conf:/etc/dnsmasq.conf:ro
    restart: always
  # 以下是新增的配置。
  wide-dhcpv6-client:
    cap_add:
      - NET_ADMIN
    container_name: wide-dhcpv6-client
    environment:
      # 这里的 INTERFACE 是连接外网的接口名称。
      - INTERFACE=ppp0
    image: hoshinorei/wide-dhcpv6-client
    network_mode: host
    restart: always
    volumes:
      - ./config/wide-dhcpv6-client/dhcp6c.conf:/etc/wide-dhcpv6/dhcp6c.conf:ro
```

修改完成后，启动容器。

```bash
sudo docker-compose -f ~/docker-compose.yml up -d
```

不出意外的话，现在 LAN 口应该能获取到公网 IPv6 地址了。因为咱们前面已经将 Dnsmasq 设置好了，所以局域网的设备应该也能被分配公网 IPv6 地址。

```text
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether ██:██:██:██:██:██ brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.1/24 brd 192.168.3.255 scope global enp1s0
       valid_lft forever preferred_lft forever
    inet6 240█:████:████:████::/64 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::362f:579d:8c86:105b/64 scope link
       valid_lft forever preferred_lft forever
```

你可以试试在内网 ping 外网的 IPv6 地址，在外网 ping 内网设备的公网 IPv6 地址。只要没被设备上的防火墙挡住，应该都是可以 ping 通的。

关于 PD 前缀还有一个问题，因为小玲这里的 PPPoE 连接偶尔会自己断一下，虽然 PPPoE 会自动重连，但是断开前获取的 PD 前缀会在重连后无法使用。所以咱们要写一个脚本，在 PPPoE 连接断开时自动删除已有的 PD 前缀，然后 PPPoE 连接成功后自动获取新的 PD 前缀。

在 `/etc/ppp/ipv6-down.d/` 文件夹下创建一个脚本，这个脚本小玲就命名为 `del_ipv6_pd_prefix`。

```bash
sudo vim /etc/ppp/ipv6-down.d/del_ipv6_pd_prefix
```

写入以下内容。

```bash
#!/usr/bin/bash

# 这里请替换成你的 LAN 口的接口名称。
INTERFACE=enp1s0

IPV6_ADDRESS=$(ip a show dev $INTERFACE scope global | awk '/inet6/{print $2}' | awk 'NR==1')

# 这里判断 IPV6_ADDRESS 这个变量是否不为空，不为空则说明有 PD 前缀。
if [[ -n "$IPV6_ADDRESS" ]]; then
  # 不为空就把 PD 前缀删除。
  ip addr del $IPV6_ADDRESS dev $INTERFACE
fi

exit 0
```

在 `/etc/ppp/ipv6-up.d/` 文件夹下创建一个脚本，这个脚本小玲就命名为 `restart_wide-dhcpv6-client`。

```bash
sudo vim /etc/ppp/ipv6-up.d/restart_wide-dhcpv6-client
```

写入以下内容。

```bash
#!/bin/sh

# 这里的 docker-compose.yml 的路径请改成你自己的，请务必使用绝对路径。
docker-compose -f /home/hoshinorei/docker-compose.yml restart wide-dhcpv6-client

exit 0
```

写入脚本后，赋予它们执行权限。

```bash
sudo chmod +x /etc/ppp/ipv6-down.d/del_ipv6_pd_prefix /etc/ppp/ipv6-up.d/restart_wide-dhcpv6-client
```

理论上，现在就不会因为 PPPoE 连接重连后导致局域网内的设备无法使用 IPv6 了。

## UPnP

咱们最好启用 UPnP 功能，这样电脑上如果有 BT 下载软件，通过 UPnP 映射了端口之后。与别人连接会更容易一点。咱们使用 miniupnpd 来提供 UPnP 服务。

创建一个文件夹来存放 miniupnpd 的配置文件。

```bash
mkdir -p ~/config/miniupnpd
```

编写 `miniupnpd.conf`。

```bash
vim ~/config/miniupnpd/miniupnpd.conf
```

写入以下内容。

```text
# 这三行配置的作用是检测路由器是否有公网 IP 或者在不受限制的 NAT 类型下面。
# 如果检测不通过 UPnP 就不会起作用。 
ext_perform_stun=yes
ext_stun_host=stun.stunprotocol.org
ext_stun_port=3478

# 使用系统时间而不是 miniupnpd 的运行时间。
system_uptime=yes

# 这里的 UUID 请自己生成
uuid=446cd1fe-9b0b-4527-8f2f-a58f2e7e5504

# WAN 口的接口名称
ext_ifname=ppp0

# 是否启用 UPnP
enable_upnp=yes

# 是否启用 NAT-PMP
enable_natpmp=yes

# LAN 口的接口名称
listening_ip=enp1s0

# 这两行的配置的意思是只允许将路由器 WAN 口的 1024-65535 的端口映射到内网任意地址的 1024-65535 端口
allow 1024-65535 0.0.0.0/0 1024-65535
deny 0-65535 0.0.0.0/0 0-65535
```

`allow` 的意思是允许哪些端口被映射，你也可以把 `allow` 改为 `deny` 来禁止哪些端口被映射。格式都是一样的。

第 1 个 `1024-65535` 是路由器本身也就是外网能访问的端口范围。

第 2 个 `1024-65535` 是内网的端口范围。

`0.0.0.0/0` 是内网的 IPv4 网段，这里设成 `0.0.0.0/0` 指的是任意网段。

UUID 可以使用这个 [网站](https://www.uuidgenerator.net/version4) 生成。

编辑 `docker-compose.yml` 文件，加入 miniupnpd 的配置。

```yaml
version: "3"
services:
  dnsmasq:
    container_name: dnsmasq
    cap_add: 
      - NET_ADMIN
    image: hoshinorei/dnsmasq
    network_mode: host
    volumes:
      - ./config/dnsmasq/dnsmasq.conf:/etc/dnsmasq.conf:ro
    restart: always
  wide-dhcpv6-client:
    cap_add:
      - NET_ADMIN
    container_name: wide-dhcpv6-client
    environment:
      - INTERFACE=ppp0
    image: hoshinorei/wide-dhcpv6-client
    network_mode: host
    restart: always
    volumes:
      - ./config/wide-dhcpv6-client/dhcp6c.conf:/etc/wide-dhcpv6/dhcp6c.conf:ro
  # 以下是新增的配置。
  miniupnpd:
    cap_add:
      - NET_ADMIN
    container_name: miniupnpd
    image: hoshinorei/miniupnpd
    network_mode: host
    restart: always
    tmpfs:
      - /run
    volumes:
      - ./config/miniupnpd/miniupnpd.conf:/etc/miniupnpd/miniupnpd.conf:ro
```

修改完成后，启动容器。

```bash
sudo docker-compose -f ~/docker-compose.yml up -d
```

## 结尾

好了，你现在拥有了一个能正常上网的软路由啦。不过软路由的强大之处可不止这么点，小玲以后还会带来软路由的更多玩法。另外文章中出现的所有 Docker 镜像都已经开源在 [Github](https://github.com/HoshinoRei?tab=repositories&q=docker&type=&language=&sort=)，可以放心使用。
