---
title: 使用 fnm 管理 Node.js 版本
author: 星野玲
date: 2023-10-15T08:00:00+08:00
description: 一直以来，小玲都是用 Scoop 来安装 Node.js 的。以前小玲只用 main bucket 下的 nodejs 这个包。因为小玲没有使用旧版的 Node.js 的需求（虽然这个问题也能用 Scoop 的 version bucket 解决。），有魔法以后也不用担心下载国外的文件慢的问题，但是小玲决定改变这种安装方式。
draft: false
tags:
  - fnm
  - Node.js
  - Windows

---

一直以来，小玲都是用 [Scoop]({{< ref "11.md" >}}) 来安装 Node.js 的。以前小玲只用 main bucket 下的 `nodejs` 这个包。因为小玲以前没有使用旧版的 Node.js 的需求（虽然这个问题也能用 Scoop 的 version bucket 解决。），有魔法以后也不用担心下载国外的文件慢的问题，但是小玲决定改变这种安装方式。

fnm 是使用 Rust 编写的，意味着咱们可以在 Linux、macOS、Windows 上都能使用 fnm。这里小玲就拿 Windows 作为例子。

## 安装

使用 Scoop 安装 [fnm](https://github.com/Schniz/fnm)。

```powershell
scoop install main/fnm
```

安装好后，需要往 `$profile` 文件加一行命令使得 fnm 在 PowerShell 中生效，小玲就用 VSCode 编辑 `$profile` 了。如果没有安装 VSCode，可以通过 `scoop install extras/vscode` 安装。

```powershell
code $profile
```

在最后一行添加下面的命令。

```powershell
fnm env --use-on-cd | Out-String | Invoke-Expression
```

## 设置镜像

fnm 默认是从 Node.js 官网下载的，有时候下载会特别慢。咱们可以使用下面的命令设置环境变量，将源设置成国内阿里云的。

```powershell
[Environment]::SetEnvironmentVariable('FNM_NODE_DIST_MIRROR', 'https://mirrors.aliyun.com/nodejs-release/', 'User');
```

使用上面的命令设置了环境变量之后，请重启终端使环境变量生效。

## 版本号

在 fnm 里，Node.js 的版本号是可以省略小版本号的，省略了小版本号之后，剩下大版本号是指这个大版本下的最新一个小版本。

例如最新的 Node.js 版本是 `20.8.1`，`20.8` 就等同于 `20.8.1` 而不是 `20.8.0`。`20` 就等同于 `20.8` 而不是 `20.7`、`20.6` 以及更低的版本。

## 安装最新的 Node.js

```powershell
fnm install --latest
```

## 安装任意一个版本的 Node.js

```powershell
fnm install <version>
```

例如

```powershell
fnm install 20
```

## 卸载任意一个版本的 Node.js

```powershell
fnm uninstall <version>
```

卸载时，如果执行 `fnm uninstall 20`，但是已经安装了两个或以上的 `20` 大版本的 Node.js，这时候就需要把版本号写更精确一些。

## 查看当前安装了哪些版本

```powershell
fnm ls
```

标有 `default` 的版本是当前正在使用的版本。

## 查看可以安装哪些版本

```powershell
fnm ls-remote
```

## 使用某个版本

```powershell
fnm use <version>
```

## 将当前使用的版本添加到 Path 环境变量

虽然咱们能在 PowerShell 使用 `node` 命令了，但是咱们并没有把 `node` 命令所在的文件夹加入 `Path` 环境变量，这样 WebStorm 就没有识别到使用 fnm 安装的 Node.js。

{{< figure src="Snipaste_2023-10-15_07-43-34.png" position="center">}}

解决方法也很简单，使用下面的命令把路径添加到 `Path` 环境变量就可以了。

```powershell
[Environment]::SetEnvironmentVariable('Path', [Environment]::GetEnvironmentVariable('Path', 'User') + ";$env:APPDATA\fnm\aliases\default", 'User')
```

如果 WebStorm 里还是没有识别到，可以试试重启系统。
