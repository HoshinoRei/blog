---
title: Docker 下的 Traefik 上手教程（二）
author: 星野玲
date: 2023-01-29T00:00:00+08:00
description: 上周小玲讲了 Traefik 的基本使用方法。这周小玲来讲一些进阶的知识。
draft: false
tags:
  - Docker
  - Docker Compose
  - Traefik

---

上周小玲讲了 Traefik 的基本使用方法。这周小玲来讲一些进阶的知识。因为看过上一篇文章的你已经知道修改 `docker-compose.yml` 后都要 `docker compose up` 一下了，所以这篇文章就不再提到这个命令了。

## middleware

咱们在使用 Traefik 时，一般都会用到 middleware。比如咱们可以通过中间件让访问者浏览 HTTP 协议的地址时，自动跳转到 HTTPS 协议，或者开启压缩以节省服务器的带宽。下面就用这两个例子来讲解如何使用 middleware。

首先咱们需要定义一个 middleware。在这里需要注意一点的就是在 `docker-compose.yml` 的 labels 里定义的 route 要想使用在文件里定义的 middleware 时，需要在 middleware 的名称后加上 `@file`。反之在文件里定义的 route 要想使用在 `docker-compose.yml` 的 labels 里定义的 middleware 时，需要在 middleware 的名称后加上 `@docker`。[^1]关于这一点，下面会有例子解释。

[^1]: 来源：[Provider Namespace](https://doc.traefik.io/traefik/v3.0/providers/overview/#provider-namespace)。

### HTTP 跳转 HTTPS

如果需要在 `docker-compose.yml` 的 labels 里定义 middleware，咱们可以在 Traefik 的 `docker-compose.yml` 文件里这样子写。

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
      - --certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web
      - --certificatesresolvers.myresolver.acme.storage=/etc/traefik/certs/acme.json
    extra_hosts:
      "host.docker.internal": "172.18.0.1"
    # 下面未注释的 2 行是新增的内容
    labels:
      # 声明一个名为 redirect-to-https 的 middleware
      # 在名为 redirect-to-https 的 middleware 下，将 redirectscheme 的 scheme 设置为 "https"
      - traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https
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

然后修改服务的 labels，让服务使用这个 middleware。

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
      # 下面未注释的 2 行是新增的内容
      # 在名为 whoami 的 route 下，将 middlewares 设置为 "redirect-to-https"
      # 如果有多个 middleware，可以用英文逗号分隔
      - traefik.http.routers.whoami.middlewares=redirect-to-https
      - traefik.http.routers.whoami-secured.rule=Host(`whoami.bling.moe`)
      - traefik.http.routers.whoami-secured.service=whoami
      - traefik.http.routers.whoami-secured.tls
      - traefik.http.routers.whoami-secured.entrypoints=websecure
      - traefik.http.routers.whoami-secured.tls.certresolver=myresolver
    networks:
      - traefik_default
networks:
  traefik_default:
    external: true
```

### 启用压缩

Traefik 3.0 支持 gzip 和 Brotli 压缩。

如果需要在文件里定义 middleware，咱们可以在 Traefik 的 `docker-compose.yml` 文件里这样子写。

```bash
vim ~/docker_config/traefik/middlewares.yml
```

```yaml
http:
  middlewares:
    # 定义一个名为 compression 的 middleware
    compression:
      # 名为 compression 的 middleware 里包含了 compress 这个 middleware
      compress: {}
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
      - traefik.http.routers.whoami.middlewares=redirect-to-https
      - traefik.http.routers.whoami-secured.rule=Host(`whoami.bling.moe`)
      - traefik.http.routers.whoami-secured.service=whoami
      - traefik.http.routers.whoami-secured.tls
      - traefik.http.routers.whoami-secured.entrypoints=websecure
      - traefik.http.routers.whoami-secured.tls.certresolver=myresolver
      # 下面未注释的 1 行是新增的内容
      # 在名为 whoami-secured 的 route 下，将 middlewares 设置为 "compression@file"
      - traefik.http.routers.whoami-secured.middlewares=compression@file
    networks:
      - traefik_default
networks:
  traefik_default:
    external: true
```

### HTTP Basic Authentication

咱们首先要通过一个命令将用户名变成一段 Traefik 可以识别的字符串。

使用这个命令前，咱们需要安装 `apache2-utils`。

```bash
apt install -y apache2-utils
```

```bash
echo $(htpasswd -nB <username>) | sed -e s/\\$/\\$\\$/g
```

咱们以用户名 `test`，密码 `123456` 为例。

```bash
echo $(htpasswd -nB test) | sed -e s/\\$/\\$\\$/g
```

输入两次密码后，返回

```test
test:$$2y$$05$$SkpWa0Cx2rEoAEapv19Mjub.SaEmo7tJ5TqagI7ZqCNEPms5M0zUq
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
      - traefik.http.routers.whoami.middlewares=redirect-to-https
      - traefik.http.routers.whoami-secured.rule=Host(`whoami.bling.moe`)
      - traefik.http.routers.whoami-secured.service=whoami
      - traefik.http.routers.whoami-secured.tls
      - traefik.http.routers.whoami-secured.entrypoints=websecure
      - traefik.http.routers.whoami-secured.tls.certresolver=myresolver
      # 下面未注释的 1 行是修改的内容
      # 在名为 whoami-secured 的 route 下，将 middlewares 设置为 "compression@file,test-auth"
      - traefik.http.routers.whoami-secured.middlewares=compression@file,test-auth
      # 下面未注释的 1 行是新增的内容
      # 声明一个名为 test-auth 的 middleware
      # 在名为 test-auth 的 middleware 下，将 basicauth 的 users 设置为 "test:$$2y$$05$$SkpWa0Cx2rEoAEapv19Mjub.SaEmo7tJ5TqagI7ZqCNEPms5M0zUq"
      # 如果有多个用户，可以用英文逗号分隔
      - traefik.http.middlewares.test-auth.basicauth.users=test:$$2y$$05$$SkpWa0Cx2rEoAEapv19Mjub.SaEmo7tJ5TqagI7ZqCNEPms5M0zUq
    networks:
      - traefik_default
networks:
  traefik_default:
    external: true
```

## 启用 HTTP/3

如果要启用 HTTP/3，只需在 Traefik 的 `docker-compose.yml` 文件的 command 里加上这两行就可以了。

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
      # 在名为 websecure 的 entrypoint 下，开启 HTTP/3
      - --entrypoints.websecure.http3
      # 在名为 websecure 的 entrypoint 下，将 http3 的 advertisedPort 设置为 443
      - --entrypoints.websecure.http3.advertisedPort=443
      - --certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web
      - --certificatesresolvers.myresolver.acme.storage=/etc/traefik/certs/acme.json
    extra_hosts:
      "host.docker.internal": "172.18.0.1"
    labels:
      - traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https
    image: traefik:v3.0
    ports:
      - 80:80
      - 443:443
      # 下面未注释的 1 行是新增的内容
      # 443 端口也要映射 UDP 协议
      - 443:443/udp
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
