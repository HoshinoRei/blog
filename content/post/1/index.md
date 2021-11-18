---
title: Windows 包管理器——winget 上手教程
author: 星野玲
date: 2021-10-14T15:13:05+08:00
description: Windows程序包管理器（英语：Windows Package Manager，也称 winget）是微软为 Windows 10 开发的一款自由开源的软件包管理器。它由一个命令行实用程序（CLI）和一组安装应用程序的服务组成。独立软件供应商可以将其作为软件包的分发渠道。
draft: false
tags:
  - winget
  - Windows
  - Windows 10
  - Windows 11

---

> Windows程序包管理器（英语：Windows Package Manager，也称 winget）是微软为 Windows 10 开发的一款自由开源的软件包管理器。它由一个命令行实用程序（CLI）和一组安装应用程序的服务组成。独立软件供应商可以将其作为软件包的分发渠道。[^1]

[^1]: 选自 《[Windows程序包管理器 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/Windows%E7%A8%8B%E5%BA%8F%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8)》。

## 为什么要使用 winget

如果你是 Windows 使用者，你可能会经历过需要大量安装 win32 程序的痛苦：一个一个点开这些 exe 或 msi 文件，然后不断地点“下一步”，等待安装完成，再点“完成”。如果只是在一台电脑上安装还好，但是如果需要给 10 台，100 台电脑装软件，这个动作应该会让人抓狂吧？下载软件的时候也很麻烦，咱们需要去那个软件的官网，把它的安装包下载进来。如果不小心从某些乱七八糟的网站上下载，还有中毒的危险。如果你用过 Linux，你应该会体验过包管理器的便捷。想要什么软件，一个命令就把软件及其依赖给装好了。所以咱们需要一个 Windows 的包管理器，正好微软开发了 winget。咱们就来学习一下 winget 吧。

## 安装

### Windows 10

如果你用的是 Windows 10，请确保它的版本是 1809 及以上，不然是无法使用 winget 的。然后在应用商店里把“应用安装程序”更新到最新版，或者去 Github [下载 winget](https://github.com/microsoft/winget-cli/releases/latest)，winget 的安装包是一个 msixbundle 文件，下载它并打开。完毕之后打开 PowerShell。输入 `winget -v` 并回车。如果终端返回一个版本号（比如 `v1.1.12653`）就表示安装成功啦。

如果你用的是 Windows 10 LTSC，由于 Windows 10 LTSC 不自带应用商店，无法按照上面的教程安装。你可以参考 [这篇教程](https://kms.app/archives/414/)。小玲亲测在 Windows 10 LTSC 2021 上安装成功。

### Windows 11

Windows 11 的安装步骤和 Windows 10 没有什么区别。

## 熟悉 winget

咱们先来查看一个 winget 有哪些命令吧。输入 `winget` 并回车以查看 winget 的命令。你应该会看到下面的输出。

```powershell
PS C:\> winget
Windows Package Manager v1.1.12653
版权所有 (C) Microsoft Corporation。保留所有权利。

WinGet 命令行实用工具可从命令行安装应用程序和其他程序包。

使用情况: winget [<命令>] [<选项>]

下列命令有效:
  install    安装给定的程序包
  show       显示包的相关信息
  source     管理程序包的来源
  search     查找并显示程序包的基本信息
  list       显示已安装的程序包
  upgrade    升级给定的程序包
  uninstall  卸载给定的程序包
  hash       哈希安装程序的帮助程序
  validate   验证清单文件
  settings   打开设置或设置管理员设置
  features   显示实验性功能的状态
  export     导出已安装程序包的列表
  import     安装文件中的所有程序包

如需特定命令的更多详细信息，请向其传递帮助参数。 [-?]

下列选项可用：
  -v,--version  显示工具的版本
  --info        显示工具的常规信息

可在此找到更多帮助： https://aka.ms/winget-command-help
PS C:\Users\Admin>
```

虽然 winget 有那么多命令，但咱们一般会用到的也就只有 6 个，分别是

|             命令             |        作用        |
| :--------------------------: | :----------------: |
|  `winget install --id <id>`  |   安装一个程序包   |
|  `winget upgrade --id <id>`  |   升级一个程序包   |
|    `winget upgrade --all`    |   升级所有程序包   |
| `winget uninstall --id <id>` |   卸载一个程序包   |
|    `winget search <name>`    |   搜索一个程序包   |
|        `winget list`         | 查看已安装的程序包 |

## 使用 winget 安装一个软件

如果想用 winget 安装一个软件，咱们需要知道它的 ID，因为软件的名字可以重复，但 ID 一定是唯一的。咱们需要先用 `winget search` 命令搜索咱们能在 winget 安装哪些软件。比如咱们需要安装一个 Chrome 浏览器。咱们就输入 `winget search chrome`。

```powershell
PS C:\> winget search chrome
名称                       ID                         版本         匹配            源
------------------------------------------------------------------------------------------
Streamer to Chromecast     9MTTSZ74DBRS               Unknown                      msstore
Google Chrome              Google.Chrome              94.0.4606.81 Moniker: chrome winget
Google Chrome Beta         Google.Chrome.Beta         95.0.4638.49 Command: chrome winget
Google Chrome Dev          Google.Chrome.Dev          96.0.4662.6  Command: chrome winget
Stack                      stack.stack                3.32.0       Tag: chrome     winget
Brave                      BraveSoftware.BraveBrowser 95.1.31.84   Tag: Chrome     winget
Chrome Remote Desktop Host Google.ChromeRemoteDesktop 94.0.4606.27 Tag: chrome     winget
Ginger Chrome              Saxo_Broko.GingerChrome    93.0.4529.0                  winget
115电脑版                  115.115Chrome              25.0.0.3                     winget
360极速浏览器              360.360Chrome              13.0.2256.0                  winget
Google Chrome Canary       Google.Chrome.Canary       97.0.4668.0                  winget
```

终端返回了一个列表，里面正好有 Chrome 和它的 ID：`Google.Chrome`，所以咱们就能通过 winget 安装 Chrome。咱们只需要输入 `winget install --id Google.Chrome`。winget 就会帮咱们下载 Chrome 并安装。

```powershell
PS C:\> winget install --id Google.Chrome
已找到 Google Chrome [Google.Chrome] 版本 94.0.4606.81
此应用程序由其所有者授权给你。
Microsoft 对第三方程序包概不负责，也不向第三方程序包授予任何许可证。
Downloading https://dl.google.com/dl/chrome/install/googlechromestandaloneenterprise64.msi
  ██████████████████████████████  78.2 MB / 78.2 MB
已成功验证安装程序哈希
正在启动程序包安装...
已成功安装
```

期间可能会弹出 UAC 的提示框，这时只需选“是”就可以了。如果你不想让安装的过程中弹出 UAC 提示框，还可以在有管理员权限的 PowerShell 窗口里执行命令。winget 默认会把软件安装在 `C:\Program Files` 或 `C:\Program Files (x86)` 等文件夹。

## 使用 winget 卸载一个软件

卸载一个程序包时咱们也需要知道它的 ID。如果你不记得它的 ID，可以使用 `winget list` 命令列出所有已安装的程序包的 ID。然后使用 `winget uninstall --id <id>` 命令卸载它。虽然有些不是通过 winget 安装的软件也会显示出来。小玲相信看到这里的你已经了解了 ID 的概念了，所以这部分小玲就不做演示了。

## 使用 winget 升级软件

咱们可以通过 `winget upgrade --id <id>` 单独升级一个程序包，使用 `winget upgrade --all` 命令升级 winget 能升级的程序包。

## 使用 winget 批量安装软件包

winget 不支持一个命令安装多个程序包，比如 `winget install --id=id1 --id=id2` 是不行的。因此咱们需要写多个 `winget install` 命令，并用 `;` 分开，咱们可以在文本编辑器里先写好命令，再复制到终端里执行。

比如下面的命令就可以一次性安装 Microsoft Virtual C++ 所有版本，注意最后一条命令后面不要写 `;`。

```powershell
winget install --id=Microsoft.VC++2015-2019Redist-x86;
winget install --id=Microsoft.VC++2015-2019Redist-x64;
winget install --id=Microsoft.VC++2013Redist-x86;
winget install --id=Microsoft.VC++2013Redist-x64;
winget install --id=Microsoft.VC++2012Redist-x86;
winget install --id=Microsoft.VC++2012Redist-x64;
winget install --id=Microsoft.VC++2010Redist-x86;
winget install --id=Microsoft.VC++2010Redist-x64;
winget install --id=Microsoft.VC++2008Redist-x86;
winget install --id=Microsoft.VC++2008Redist-x64;
winget install --id=Microsoft.VC++2005Redist-x86;
winget install --id=Microsoft.VC++2005Redist-x64
```

你可以仿照小玲写的例子，写一个适合你自己的命令。

## 注意事项

细心的你可能会注意到咱们通过 winget 安装 Chrome 时是从谷歌的服务器上下载的，因为 winget 没有设计一个服务器用于存放各个软件的安装包，所以你有可能会遇到一个软件的安装包下载不了。小玲相信你一定知道该怎么做了吧？小玲就不再赘述了。

## 结尾

最后小玲推荐给你一个网站——[winstall](https://winstall.app/)。在这个网站你可以找到部分能通过 winget 安装的软件，而且网站会生成安装命令，你只需要复制进终端执行就可以安装啦。
