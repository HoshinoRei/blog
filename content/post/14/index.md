---
title: Docker 下的 Traefik 上手教程
author: 星野玲
date: 2023-01-22T00:00:00+08:00
description: 今天小玲来给大家介绍小玲目前在服务器上用的负载均衡器——Traefik。
draft: false
tags:
  - Docker
  - Docker Compose
  - Traefik

---

本来是今天要发布新文章的，但今天正好是大年初一，小玲就顺便祝大家农历新年快乐，万事如意！

今天小玲来给大家介绍小玲目前在服务器上用的负载均衡器——Traefik。

## Traefik 介绍

什么是 Traefik？你如果没听说过 Traefik，那你可能听说过 Nginx、Caddy。Traefik 的功能跟它们差不多，但是用起来比它们更优雅，它可以通过在 `docker-compose.yml` 文件里写一些 label 完成服务发现。小玲很久以前就用过 Nginx 当反向代理，后来又改为 Caddy，最后因为服务器上的服务都用 Docker 装了，就改用 Traefik 了。

## 安装

安装 Traefik 前，需要先安装 Docker。Docker 的安装方法请看 [这里]({{< ref "5.md#安装-docker" >}})。因为小玲写这篇文章时 Traefik 3.0 已经发布 Beta 版了，所以咱们采用 Traefik 3.0 版本。

安装完 Docker 后，假设咱们现在在家目录下，咱们新建一个 `docker-compose` 文件夹用于存放各种服务的 `docker-compose.yml` 文件。

```bash
mkdir -p ~/docker-compose
```

在 `docker-compose` 文件夹下创建 `traefik` 文件夹。

```bash
mkdir -p ~/docker-compose/traefik
```

在 `traefik` 文件夹下创建 `docker-compose.yml` 文件。

```bash
vim ~/docker-compose/traefik/docker-compose.yml
```

写入以下内容。

```yaml
version: "3"
services:
  traefik:
    container_name: traefik
    command:
      # 启用 Docker 服务提供者，这样 Traefik 就会监听来自 Docker 的配置
      - --providers.docker
      # 开启一个 8080 端口，并在这个端口上暴露 Traefik API 和一个 web 面板
      - --api.insecure
    extra_hosts:
      "host.docker.internal": "172.18.0.1"
    image: traefik:v3.0
    ports:
      - 80:80
      - 8080:8080
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
networks:
  default:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: "172.18.0.0/24"
```

这里的 `172.18.0.0/24` 网段是 Traefik 和需要与 Traefik 通信的 Docker 容器所在的网段。你也可以改成你喜欢的网段。`172.18.0.1` 是 Docker 容器内访问 Docker 容器外的主机的需要连接的 IP 地址。通常是网段的第一个可用地址。

启动 Traefik。

```bash
sudo docker compose -f ~/docker-compose/traefik/docker-compose.yml up -d
```

启动后，浏览器访问 `http://<你的服务器IP>:8080`，理论上你就能看到 Traefik 的 web 面板了。

{{< figure src="Snipaste_2023-01-21_01-22-16.png" position="center">}}

## 将一个 web 服务暴露在 80 端口上并绑定域名

这里咱们就以 `whoami` 为例，来讲解如何将一个 web 服务暴露在 80 端口上并绑定域名。

在 `docker-compose` 文件夹下创建 `whoami` 文件夹。

```bash
mkdir -p ~/docker-compose/whoami
```

在 `whoami` 文件夹下创建 `docker-compose.yml` 文件。

```bash
vim ~/docker-compose/whoami/docker-compose.yml
```

写入以下内容。

```yaml
version: "3"
services:
  whoami:
    image: traefik/whoami
    labels:
      # 指定 Traefik 所在的网络为 "traefik_default"
      - traefik.docker.network=traefik_default
      # 声明一个名为 whoami 的 service
      # 在名为 whoami 的 service 下，设置端口为容器内的 80 端口
      - traefik.http.services.whoami.loadbalancer.server.port=80
      # 声明一个名为 whoami 的 route
      # 在名为 whoami 的 route 下，将 rule 值为 "Host(`whoami.domain`)"
      - traefik.http.routers.whoami.rule=Host(`whoami.domain`)
      # 在名为 whoami 的 route 下，将 service 设置为 "whoami"
      - traefik.http.routers.whoami.service=whoami
    networks:
      # 这个服务需要连接的网络有 traefik_default
      - traefik_default
networks:
  # 声明这个 docker-compose.yml 里的服务需要使用的网络，这个网络名为 traefik_default
  traefik_default:
    # 这个网络是在这个文件外部声明的，所以咱们需要将 external 设为 true
    external: true
```

`` Host(`whoami.domain`) `` 的意思是绑定到域名 `whoami.domain` 上。

`treafik_default` 这个名字是怎么来的呢？咱们可以通过 `sudo docker network ls` 命令查看。

```text
NETWORK ID     NAME              DRIVER    SCOPE
9f0f43903e7a   bridge            bridge    local
4875654d80b2   host              host      local
c4fab9dcea28   none              null      local
117f3dd6ec15   traefik_default   bridge    local
```

启动 `whoami`。

```bash
sudo docker compose -f ~/docker-compose/traefik/docker-compose.yml up -d
```

想要正常访问 `whoami` 服务需要做好域名解析，这里咱们就通过改本地的 `hosts` 文件了，修改方法小玲不再赘述。

通过浏览器访问 `whoami.domain` 域名，可以看到这时候已经可以访问到 `whoami` 服务了。

```text
Hostname: de4ebf3ea06e
IP: 127.0.0.1
IP: 172.18.0.3
RemoteAddr: 172.18.0.2:49166
GET / HTTP/1.1
Host: whoami.domain
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,en-US;q=0.7,en;q=0.3
Dnt: 1
Sec-Gpc: 1
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 192.168.3.16
X-Forwarded-Host: whoami.domain
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: da8861ae460f
X-Real-Ip: 192.168.3.16
```

在 web 面板里也能看到多了一个 route。

{{< figure src="Snipaste_2023-01-21_01-52-16.png" position="center">}}

## 通过 Traefik 负载均衡 web 服务

为什么说 Traefik 是负载均衡器呢？通过下面这个例子告诉你。

咱们把 `whoami` 服务扩展到 2 个容器。

```bash
sudo docker compose -f ~/docker-compose/whoami/docker-compose.yml up -d --scale whoami=2
```

然后不断地刷新 `http://whoami.domain`，你就会发现它的 `Hostname` 和 `IP` 一直在变化，这就说明 Traefik 已经将请求分摊到了 2 个容器上。而且这个过程咱们只用了 1 条命令，可以说是非常优雅了。

## Traefik 与 HTTPS

现在的 web 服务一般都采用 HTTPS 协议了。所以小玲必须讲如何配置 HTTPS。

首先需要在 Traefik 里定义 2 个 entrypoint，一个名为 `web`，绑定在 Traefik 容器的 `80` 端口上，另一个名为 `websecure`，绑定在 Traefik 容器的 `443` 端口上。

编辑 Traefik 的 `docker-compose.yml` 文件。

```bash
vim ~/docker-compose/traefik/docker-compose.yml
```

```yaml
version: "3"
services:
  traefik:
    container_name: traefik
    command: 
      - --providers.docker
      - --api.insecure
      # 下面未注释的 3 行是新增的内容
      # 启用文件服务提供者，并设置配置文件的目录为 "/etc/traefik/config"
      - --providers.file.directory=/etc/traefik/config
      # 声明一个名为 web 的 entrypoint，地址为任意地址的 80 端口
      - --entryPoints.web.address=:80
      # 声明一个名为 websecure 的 entrypoint，地址为任意地址的 443 端口
      - --entrypoints.websecure.address=:443
    extra_hosts:
      "host.docker.internal": "172.18.0.1"
    image: traefik:v3.0
    ports:
      - 80:80
      # 下面 1 行是新增的内容
      - 443:443
      - 8080:8080
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      # 下面 2 行是新增的内容
      - ../../docker_config/traefik:/etc/traefik/config
      - ../../docker_data/traefik/certs:/etc/traefik/certs
networks:
  default:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: "172.18.0.0/24"
```

在家目录下创建 `docker_config/traefik` 文件夹用于存放 Traefik 的配置文件。

```bash
sudo mkdir -p ~/docker_config/traefik
```

在家目录下创建 `docker_data/traefik/certs` 文件夹用于存放 Traefik 的证书文件。

```bash
sudo mkdir -p ~/docker_data/traefik/certs
```

重新启动 Traefik。

```bash
sudo docker compose -f ~/docker-compose/traefik/docker-compose.yml up -d
```

### 自定义证书

下面用一个例子讲解如何配置自定义证书。假设咱们要将 `whoami` 服务绑定在 `whoami.bling.moe` 域名上，并且可以被 HTTP 和 HTTPS 协议同时访问。咱们现在有 `*.bling.moe` 泛域名证书。

首先要将证书存放在 `~/docker_data/traefik/certs` 文件夹下。1 个证书一般有 2 个文件，pem 文件和 key 文件。这里咱们就将 pem 文件命名为 `*.bling.moe.pem`，将 key 文件命名为 `*.bling.moe.key`。

然后在 `~/docker_config/traefik` 文件夹下创建一个 `bling.moe.yml` 文件，这里的文件名可以随便取。

```bash
vim ~/docker_config/traefik/bling.moe.yml
```

写入以下内容。

```yaml
tls:
  certificates:
    - certFile: /etc/traefik/certs/*.bling.moe.pem
      keyFile: /etc/traefik/certs/*.bling.moe.key
```

然后编辑 `whoami` 的 `docker-compose.yml` 文件。

```bash
vim ~/docker-compose/whoami/docker-compose.yml
```

```yaml
version: "3"
services:
  whoami:
    image: traefik/whoami
    labels:
      - traefik.docker.network=traefik_default
      - traefik.http.services.whoami.loadbalancer.server.port=80
      # 下面未注释的 1 行与原来不同
      # 在名为 whoami 的 route 下，将 rule 设置为 "Host(`whoami.bling.moe`)"
      - traefik.http.routers.whoami.rule=Host(`whoami.bling.moe`)
      - traefik.http.routers.whoami.service=whoami
      # 下面未注释的 5 行是新增的内容
      # 在名为 whoami 的 route 下，将 entrypoint 设置为 "web"
      - traefik.http.routers.whoami.entrypoints=web
      # 声明一个名为 whoami-secured 的 route
      # 在名为 whoami-secured 的 route 下，将 rule 设置为 "Host(`whoami.bling.moe`)"
      - traefik.http.routers.whoami-secured.rule=Host(`whoami.bling.moe`)
      # 在名为 whoami-secured 的 route 下，将 service 设置为 "Host(`whoami.bling.moe`)"
      - traefik.http.routers.whoami-secured.service=whoami
      # 在名为 whoami-secured 的 route 下，将 tls 设置 true
      - traefik.http.routers.whoami-secured.tls
      # 在名为 whoami-secured 的 route 下，将 entrypoint 设置为 websecure
      - traefik.http.routers.whoami-secured.entrypoints=websecure
    networks:
      - traefik_default
networks:
  traefik_default:
    external: true
```

重新启动 `whoami`。

```bash
sudo docker compose -f ~/docker-compose/traefik/docker-compose.yml up -d
```

使用浏览器访问 `http://whoami.bling.moe` 和 `https://whoami.bling.moe`，你会发现都可以访问了。

### 使用 Let's Encrypt 的证书

Traefik 内置了获取 Let's Encrypt 的证书的功能。在使用这个功能之前咱们需要先在 Traefik 的设置里新增一个 certificatesresolver。

编辑 Traefik 的 `docker-compose.yml` 文件。

```bash
vim ~/docker-compose/traefik/docker-compose.yml
```

```yaml
version: "3"
services:
  traefik:
    container_name: traefik
    command: 
      - --providers.docker
      - --api.insecure
      - --providers.file.directory=/etc/traefik/config
      - --entryPoints.web.address=:80
      - --entrypoints.websecure.address=:443
      # 下面未注释的 2 行是新增的内容
      # 声明一个名为 myresolver 的 certificatesresolver
      # 在名为 myresolver 的 certificatesresolver 下，将 httpchallenge 的 entrypoint 设置为 "web"
      - --certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web
      # 在名为 myresolver 的 certificatesresolver 下，将 storage 设置为 "/etc/traefik/certs/acme.json"
      - --certificatesresolvers.myresolver.acme.storage=/etc/traefik/certs/acme.json
    extra_hosts:
      "host.docker.internal": "172.18.0.1"
    image: traefik:v3.0
    ports:
      - 80:80
      - 443:443
      - 8080:8080
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ../../docker_config/traefik:/etc/traefik/config
      - ../../docker_data/traefik/certs:/etc/traefik/certs
networks:
  default:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: "172.18.0.0/24"
```

重新启动 Traefik。

```bash
sudo docker compose -f ~/docker-compose/traefik/docker-compose.yml up -d
```

如果以前有使用自定义证书，则需要把自定义证书的配置删除。这里小玲就将 yml 文件加上 `.bak` 后缀，让 Traefik 不把它当成一个配置文件。

```bash
mv ~/docker_config/traefik/bling.moe.yml ~/docker_config/traefik/bling.moe.yml.bak
```

然后编辑 `whoami` 的 `docker-compose.yml` 文件。

```bash
vim ~/docker compose/whoami/docker-compose.yml
```

```yaml
version: "3"
services:
  whoami:
    image: traefik/whoami
    labels:
      - traefik.docker.network=traefik_default
      - traefik.http.services.whoami.loadbalancer.server.port=80
      - traefik.http.routers.whoami.rule=Host(`whoami.bling.moe`)
      - traefik.http.routers.whoami.service=whoami
      - traefik.http.routers.whoami.entrypoints=web
      - traefik.http.routers.whoami-secured.rule=Host(`whoami.bling.moe`)
      - traefik.http.routers.whoami-secured.service=whoami
      - traefik.http.routers.whoami-secured.tls
      - traefik.http.routers.whoami-secured.entrypoints=websecure
      # 下面未注释的 1 行是新增的内容
      # 在名为 whoami-secured 的 route 下，将 certresolver 设置为 myresolver
      - traefik.http.routers.whoami-secured.tls.certresolver=myresolver
    networks:
      - traefik_default
networks:
  traefik_default:
    external: true
```

重新启动 `whoami`。

```bash
sudo docker compose -f ~/docker-compose/traefik/docker-compose.yml up -d
```

使用浏览器访问 `https://whoami.bling.moe`，你会发现已经在使用 Let's Encrypt 的证书了。`~/docker_data/traefik/certs` 文件夹下也有一个 `acme.json` 文件，里面有 Let's Encrypt 的证书数据。

## 结尾

好了，有了以上的知识，你应该可以开始上手 Traefik 了。还有一些进阶的知识因为小玲在写这篇文章的时候比较赶，所以只能留到下次再写一篇文章了。你也可以通过查看 Traefik 的 [文档](https://doc.traefik.io/traefik/v3.0/) 来学习更多使用方法。
