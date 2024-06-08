---
title: 使用 Docker 搭建求生之路2的服务器
author: 星野玲
date: 2022-08-07T07:11:00+08:00
description: 求生之路（Left 4 Dead 2）是小玲平常玩的游戏之一，这款游戏虽然发售了 13 年，但至今依然有不小的热度。但是小玲在游玩时发现，官服哪怕是在香港的官服，延迟都挺大的，而且官服不支持第三方图，虽然游戏可以开本地服务器，但是经常会出现房主不卡，其他人都卡的情况。正好小玲有一个国内的服务器，所以小玲决定搭建一个求生之路的服务器，这样小玲和别人玩第三方图，大家的延迟都不会那么高。
draft: false
tags:
  - Debian
  - Docker
  - DockerCompose
  - Left4Dead2
  - 求生之路2
---

求生之路（Left 4 Dead 2）是小玲平常玩的游戏之一，这款游戏虽然发售了 13 年，但至今依然有不小的热度。但是小玲在游玩时发现，哪怕是在香港的官服，延迟都挺大的，而且官服不支持第三方图，虽然游戏可以开本地服务器，但是经常会出现房主不卡，其他人都卡的情况。正好小玲有一个国内的服务器，所以小玲决定搭建一个求生之路的服务器，这样小玲和别人玩第三方图，大家的延迟都不会那么高。

小玲在网上搜索教程时发现，网上大部分教程都是在裸机内直接部署服务器。这样部署有一个缺点，就是步骤多，更新麻烦。正好小玲会 Docker，何不把它做成一个 Docker 镜像？部署的话直接启动容器就可以了，更新也是一个命令的事。于是小玲自己写了一个 Dockerfile 文件，并把它开源在 [Github](https://github.com/HoshinoRei/l4d2server_docker)，Docker Hub 也有已经构建好的 [镜像](https://hub.docker.com/r/hoshinorei/l4d2server)。

## 使用教程

要想使用小玲做的 Docker 镜像，你必须会使用 Linux、Docker 和 Docker Compose。教程这里采用的系统是 Debian 11，Docker 版本为 20.10.17，Dockers Compose 版本为 2.9.0。

### 安装 Docker

```bash
sudo wget -qO- https://get.docker.com/ | sh
```

### 安装 Docker Compose

```bash
sudo wget "https://github.com/docker/compose/releases/download/v2.9.0/docker-compose-$(uname -s)-$(uname -m)" -O /usr/local/bin/docker-compose
```

国内服务器可能会无法下载 Github 的文件，咱们可以通过镜像站下载。

```bash
sudo wget "https://ghproxy.com/https://github.com/docker/compose/releases/download/v2.9.0/docker-compose-$(uname -s)-$(uname -m)" -O /usr/local/bin/docker-compose
```

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

### 准备 server.cfg 文件

`server.cfg` 是求生之路2服务器的配置文件，它的写法可以参考这个 [教程](https://steamcommunity.com/sharedfiles/filedetails/?id=276173458)。小玲这里就给一个非常简单的配置。

```bash
mkdir ~/l4d2server
vim ~/l4d2server/server.cfg
```

```cfg
// 服务器名
hostname "Left 4 Dead 2 dedicated server"
// hidden 用于在服务器列表里隐藏你的服务器，防止被人发现并 DDOS
sv_tags hidden
```

### 准备 host.txt 文件

`host.txt` 文件是玩家进入服务器后，右上角显示的横幅的内容, 它的内容就在这里定义。你也可以留空，让它为一个空白的文本文件。


### 准备 motd.txt 文件

MOTD 的全称为 Message of the Day，作用是玩家刚进入服务器时，会显示一个欢迎消息，可以为空。

```bash
touch ~/l4d2server/motd.txt
```

### 编写 docker-compose.yml 文件

```bash
vim ~/l4d2server/docker-compose.yml
```

```yml
version: "3"
services:
  l4d2server:
    command: "-secure +exec server.cfg +map c1m1_hotel -port 27015"
    container_name: l4d2server
    image: hoshinorei/l4d2server
    ports:
      - 27015:27015
      - 27015:27015/udp
    # 你也可以使用 host 的网络模式，减少 Docker 映射端口带来的性能损耗，不过这点损耗可以忽略不计。
    # network_mode: host
    restart: unless-stopped
    stdin_open: true
    tty: true
    volumes:
      # 这里主机上的要映射到容器的路径请根据你的实际情况调整。
      # 如果主机上没有 addons 这个文件夹，创建容器时会自动在主机上创建这个文件夹。
      - ./addons/:/home/steam/l4d2server/left4dead2/addons/
      - ./cfg/server.cfg:/home/steam/l4d2server/left4dead2/cfg/server.cfg:ro
      - ./host.txt:/home/steam/l4d2server/left4dead2/host.txt:ro
      - ./motd.txt:/home/steam/l4d2server/left4dead2/motd.txt:ro
```

最后一个命令创建并启动容器。

```bash
sudo docker-compose -f ~/l4d2server/docker-compose.yml -d
```

### 进入服务器的控制台

你可以通过以下命令进入服务器的控制台。

```bash
sudo docker attach l4d2server
```

这样，一个求生之路2的原版纯净服务器就安装好了，你也可以像其他教程一样安装 sourcemod 插件，这里就不赘述了。
