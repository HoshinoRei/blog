---
title: 分享一个 TranslucentTB 的配置
author: 星野玲
date: 2022-10-15T22:00:00+08:00
description: 众所周知，TranslucentTB 是 Windows 上最好用的能让任务栏透明的软件。但是怎么调教才能尽可能的漂亮呢？小玲自己调了下 TranslucentTB 的设置，总算调一个比较满意的设置。
draft: false
tags:
  - TranslucentTB
  - Windows
  - Windows10
---

众所周知，[TranslucentTB](https://www.microsoft.com/store/productId/9PF4KZ2VN4W9) 是 Windows 上最好用的能让任务栏透明的软件。但是怎么调教才能尽可能的漂亮呢？小玲自己调了下 TranslucentTB 的设置，总算调一个比较满意的设置。

## 效果

桌面下，小玲设置了背景颜色为白色，并有一点点不透明。之所以不设置成全透明是因为全透明时，白色的字容易看不清。

{{< figure src="Snipaste_2022-10-15_17-19-17.png" caption="桌面" position="center">}}

任务视图下，小玲设置了背景颜色为黑色，并将透明度设置成与任务视图几乎一致。这样的效果就是，任务栏与任务视图融为一体，肉眼几乎看不出任务栏与任务视图的分界线。

{{< figure src="Snipaste_2022-10-15_17-37-08.png" caption="任务视图" position="center">}}

最大化窗口时，任务栏不再是一片纯色。而是壁纸里未被遮挡的部分。

{{< figure src="Snipaste_2022-10-15_17-42-49.png" caption="最大化窗口时" position="center">}}

## 设置方法

首先，请安装 Microsoft Store 里的 TranslucentTB，然后打开资源管理器，在地址栏中粘贴 `%LOCALAPPDATA%\Packages\28017CharlesMilette.TranslucentTB_v826wp6bftszj\RoamingState` 后回车，编辑 `settings.json` 文件。用下面的内容覆盖原来的内容，保存就行了。

```json
// See https://TranslucentTB.github.io/config for more information
{
    "$schema": "https://sylveon.dev/TranslucentTB/schema",
    "desktop_appearance": {
        "accent": "clear",
        "color": "#0000000A",
        "show_peek": false
    },
    "visible_window_appearance": {
        "enabled": false,
        "accent": "clear",
        "color": "#00000000",
        "show_peek": true
    },
    "maximized_window_appearance": {
        "enabled": false,
        "accent": "blur",
        "color": "#00000000",
        "show_peek": true
    },
    "start_opened_appearance": {
        "enabled": false,
        "accent": "normal",
        "color": "#00000000",
        "show_peek": true
    },
    "search_opened_appearance": {
        "enabled": false,
        "accent": "normal",
        "color": "#00000000",
        "show_peek": true
    },
    "task_view_opened_appearance": {
        "enabled": true,
        "accent": "clear",
        "color": "#00000066",
        "show_peek": false
    },
    "battery_saver_appearance": {
        "enabled": false,
        "accent": "opaque",
        "color": "#00000000",
        "show_peek": true
    },
    "ignored_windows": {
        "window_class": [],
        "window_title": [],
        "process_name": []
    },
    "hide_tray": false,
    "disable_saving": false,
    "verbosity": "warn"
}
```
