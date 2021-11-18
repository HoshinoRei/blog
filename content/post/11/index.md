---
title: Windows 包管理器——Scoop 上手教程
author: 星野玲
date: 2023-01-01T23:00:00+08:00
description: 今天小玲来介绍 Windows 上的又一个包管理器——scoop，这也是小玲在 Windows 系统上最喜欢也是用的最多的包管理器。
draft: false
tags:
  - Scoop
  - Windows
  - Windows 10
  - Windows 11
 
---

今天小玲来介绍 Windows 上的又一个包管理器——Scoop，这也是小玲在 Windows 系统上最喜欢也是用的最多的包管理器。

## 安装 Scoop

安装前请确保已经安装了 Powershell 5.1 或更新的版本。现在的 Windows 10 在安装完系统后都已经自带了上面这两个了。所以使用 Windows 10 的你一般情况下不用特地去再安装了。

第一次安装时，咱们先要指定 Scoop 的安装路径，因为这个路径还决定了使用 Scoop 安装的软件的安装路径，所以请务必考虑好安装在哪。小玲的建议是不要安装在 C 盘，这样如果你重装系统后，你使用 Scoop 安装的软件就不会被删除，可以很快地把软件恢复回来而不用再一个一个下载。下面的 `Scoop` 变量的值 `D:\Users\DemoUser\Scoop` 和 `SCOOP_GLOBAL` 变量的值 `D:\Scoop` 是小玲为了演示使用的路径。请根据你的实际情况修改为适合你的值。

`SCOOP` 变量的值是 Scoop 在当前用户安装的位置。

`SCOOP_GLOBAL`变量的值是是 Scoop 在全局安装的位置。

```powershell
[Environment]::SetEnvironmentVariable('SCOOP', 'D:\Users\DemoUser\Scoop', 'User');
[Environment]::SetEnvironmentVariable('SCOOP_GLOBAL', 'D:\Scoop', 'Machine');
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser;
irm get.scoop.sh | iex
```

启动 Powershell，把上面的命令一行行地复制进终端执行就可以安装 Scoop 了。注意 `[Environment]::SetEnvironmentVariable('SCOOP_GLOBAL', 'D:\Scoop', 'Machine');` 这条命令需要管理员权限。所以你需要打开有管理员权限的 Powershell 才能执行这条命令。

## 开始使用

Git 是每个使用 Scoop 的人必须装的 app，有了 Git，咱们才能添加 bucket。所以安装完 Scoop 的第一件事就是安装 Git。

```powershell
scoop install git
```

## Bucket

bucket 是一个 app 元数据的集合，一个 bucket 存放在 Git 仓库里。安装完 Scoop 后默认只有 main Bucket。

下面是 Scoop 社区维护的 bucket。

你可以通过 `scoop bucket known` 查看这个列表。

通过 `scoop bucket list` 查看已添加的 bucket。

通过 `scoop bucket add <bucket>` 添加 bucket。

通过 `scoop bucket rm <bucket>` 移除 bucket。

|bucket|添加命令|移除命令|
|:---:|:---:|:---:|
|[main](https://github.com/ScoopInstaller/Main)|`scoop bucket add main`|`scoop bucket rm main`|
|[extras](https://github.com/ScoopInstaller/Extras)|`scoop bucket add extras`|`scoop bucket rm extras`|
|[versions](https://github.com/ScoopInstaller/Versions)|`scoop bucket add versions`|`scoop bucket rm versions`|
|[nirsoft](https://github.com/kodybrown/scoop-nirsoft)|`scoop bucket add nirsoft`|`scoop bucket rm nirsoft`|
|[sysinternals](https://github.com/niheaven/scoop-sysinternals)|`scoop bucket add sysinternals`|`scoop bucket rm sysinternals`|
|[php](https://github.com/ScoopInstaller/PHP)|`scoop bucket add php`|`scoop bucket rm php`|
|[nerd-fonts](https://github.com/matthewjberger/scoop-nerd-fonts)|`scoop bucket add nerd-fonts`|`scoop bucket rm nerd-fonts`|
|[nonportable](https://github.com/TheRandomLabs/scoop-nonportable)|`scoop bucket add nonportable`|`scoop bucket rm nonportable`|
|[java](https://github.com/ScoopInstaller/Java)|`scoop bucket add java`|`scoop bucket rm java`|
|[games](https://github.com/Calinou/scoop-games)|`scoop bucket add games`|`scoop bucket rm games`|

Scoop 社区就规定 main bucket 存放着开发者会用到的 app，比如某种语言的开发环境。还有一些知名的 CLI 程序。它的最大特点就是没有 GUI。（当然 7zip 是个例外。）[^1]

[^1]: 来源：[Criteria for including apps in the main bucket](https://github.com/ScoopInstaller/Scoop/wiki/Criteria-for-including-apps-in-the-main-bucket)。

Extras bucket 存放着不符合 main bucket 规定但又知名的程序。比如浏览器这种 GUI 程序，你都可以在 extras bucket 里找到。

Versions bucket 存放着一些知名的 app 的旧版本。

Php bucket 存放着从远古版本到最新版本的 PHP。

Nerd-fonts bucket 存放着 Nerd 字体和其他字体。

Nonportable 存放着非便携软件。也就是重装系统后虽然文件还在硬盘上，但已经不能正常运行的软件。比如通常的第三方输入法就是非便携软件。小玲不建议大家通过 Scoop 安装非便携软件。如果你的 Scoop 是安装在非 C 盘上的，重装系统后通过 Scoop 安装的非便携软件的文件虽然还残留在硬盘上，但已经不能正常使用了，这时你需要先通过 Scoop 卸载再安装才能使用。对于非便携软件，小玲建议通过 [winget]({{< ref "1.md" >}}) 安装。

Java bucket 存放着不同版本的 JRE 和 JDK。

Games bucket 存放着开源游戏和免费游戏。

一般来说 main 仓库和 extras 仓库是每个人必备的仓库。

### 注意事项

如果你在没装 Git 的情况下通过 `scoop bucket rm main` 把 main bucket 移除了。这时候你就会发现你因为没有 Git，无法通过 `scoop bucket add main` 添加 main bucket，而且因为没有 main bucket，无法通过 `scoop install git` 安装 Git。这时候貌似陷入了一个死循环。不过好在 Scoop 可以通过 `scoop install <mainfest_url>` 的方式安装一个 app 。所以咱们只需通过 `scoop install https://raw.githubusercontent.com/ScoopInstaller/Main/master/bucket/git.json` 就可以安装 Git 啦。

## 安装 app

添加完 bucket 后就可以安装 app 了，咱们可以通过 `scoop install` 命令来安装 app，命令格式如下。

```powershell
scoop install [<bucket_name>/]<app_name>[@<version>]
scoop install <manifest_url>
```

例如，安装 Chrome。

```powershell
scoop bucket add extras
scoop install googlechrome
```

但是假如有两个 bucket 都有 `googlechrome` 这个 app。那通过 `scoop install googlechrome` 安装的是哪个 bucket 的 `googlechrome` 呢？

咱们可以通过 `scoop search` 命令来搜索 app，遇到不同 bucket 里有同名的 app 时一般会安装搜索结果里最靠前的那个 bucket 的 app。

如果要指定安装某个 bucket 的 app 就需要这样子写。

```powershell
scoop install extras/googlechrome
```

如果要安装指定版本的 app 就需要这样子写。

```powershell
scoop install extras/googlechrome@107.0.5304.107
```

小玲也建议大家平常安装 app 时，先搜索一下 app 的名字，如果有两个或以上的 bucket 都有同名的 app 时，通过上面的方法精确定位你要安装的 app。

如果想为所有用户安装，就要使用 `-g` 参数来进行全局安装，卸载时也是如此。

## 搜索 app

```powershell
scoop search <app_name>
```

不过 Scoop 自带的搜索命令在 bucket 数量多时会比较慢，小玲建议大家安装 `scoop-search` 这个包。

```powershell
scoop install main/scoop-search
```

通过 `scoop-search` 命令来搜索会比 `scoop search` 快很多。

```powershell
scoop-search <app_name>
```

## 卸载 app

```powershell
scoop uninstall <app_name>
```

一些 app 会产生能被 Scoop 管理的数据，如果你想在卸载 app 时同时删除这些数据，请加入 `-p` 参数。

## 查看已安装的 APP

```powershell
scoop list
```

## 更新 Scoop

```powershell
scoop update
```

这个命令可以更新 Scoop 和 bucket 的元数据。

## 更新 app

```powershell
scoop update <app_name>
```

如果要更新所有 app，可以使用 `scoop update *` 或 `scoop update -a`。

## 禁止更新 app

如果你希望一直使用某个 app 的旧版本，不希望在使用 `scoop update -a` 时将它升级到最新版，可以使用这个命令禁止更新这个 app。

```powershell
scoop hold <app_name>
```

解除禁止更新可以用这个命令。

```powershell
scoop unhold <app_name>
```

## 清理旧版本

```powershell
scoop cleanup <app_name>
```

Scoop 在安装 app 的新版本时，不会删除旧版本。所以咱们需要用这个命令来清理旧版本。

## 清理安装文件

Scoop 在安装 app 时，会保留安装文件。

你可以通过 `scoop cache` 查看所有安装文件。

通过 `scoop cache rm <app_name>` 删除指定 app 的安装文件。

通过 `scoop cache rm *` 或 `scoop cache rm -a` 或 `scoop cache rm --all` 删除所有安装文件。

## 设置 Scoop

你可以通过 `scoop help config` 来查看 Scoop 都有哪些设置。

通过 `scoop config` 查看所有已修改的设置。

通过 `scoop config <config_name>` 查看设置的值。

通过 `scoop config <config_name> <value>` 更新设置。

通过 `scoop config rm <config_name>` 删除设置。

### 给 Scoop 设置代理

如果在使用 Scoop 时遇到下载慢的问题，可以给 Scoop 设置一个代理。这个代理得是 HTTP 代理，SOCKS 代理是不行的。

```powershell
scoop config proxy [<username>:<password>@]host:port
```

### 使用 Aria2 多线程下载

你可以使用 Aria2 多线程下载。不过并不是什么 app 都支持多线程下载的。有时候会有下载失败的情况。小玲的建议是开启代理就行了，如果代理的速度够快，就没有必要使用多线程下载。

安装 Aria2。

```powershell
scoop install main/aria2
```

让 Scoop 使用 aria2

```powershell
scoop config aria2-enabled true
```

让 Scoop 不使用 aria2

```powershell
scoop config aria2-enabled false
```

## 重装系统后恢复 Scoop

重装系统后，只需重新设置一下环境变量，不需要重新下载 Scoop。注意下面的路径要设置成你原来的路径。

```powershell
[Environment]::SetEnvironmentVariable('SCOOP', 'D:\Users\DemoUser\Scoop', 'User')
[Environment]::SetEnvironmentVariable('SCOOP_GLOBAL', 'D:\Scoop', 'Machine')
[Environment]::SetEnvironmentVariable('Path', [Environment]::GetEnvironmentVariable('Path', 'User') + "; " + [Environment]::GetEnvironmentVariable('SCOOP', 'User') + "\shims", 'User')
# 注意下面的命令需要管理员权限。
[Environment]::SetEnvironmentVariable('Path', [Environment]::GetEnvironmentVariable('Path', 'Machine') + "; " + [Environment]::GetEnvironmentVariable('SCOOP_GLOBAL', 'Machine') + "\shims", 'Machine')
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

恢复所有 Scoop 安装的软件。

```powershell
scoop reset *
```

然后你就会发现，原来使用 Scoop 安装的所有软件都恢复了。

## 结尾

好了，小玲已经介绍了最基本的 Scoop 的使用方法了。只要你不是开发者，这些命令应该够你日常使用了。小玲自己平常使用的最多的也就是 `scoop update -a` 而已。如果你想知道 Scoop 还有哪些命令，可以通过 `scoop help` 查看。想知道如何制作一个 app 的元数据，可以查看 Scoop 的 [Wiki](https://github.com/ScoopInstaller/Scoop/wiki)
