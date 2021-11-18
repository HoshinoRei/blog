---
title: Windows 上小狼毫输入法的上手教程
author: 星野玲
date: 2021-10-17T14:58:00+08:00
description: 小玲以前在使用 Windows 系统时，一直都是用系统自带的输入法——微软拼音的，而且小玲没有那种安装第三方输入法的习惯。但是有一天，小玲在网上看到好多人推荐的 Rime 输入法，抱着试一下的想法，小玲试用了一下这款输入法。没想到这款输入法的可定制性真的高。小玲从此就离不开这款输入法了。
draft: false
tags:
  - Rime
  - Windows
  - Weasel
  - 小狼毫

---

小玲以前在使用 Windows 系统时，一直都是用系统自带的输入法——微软拼音的，而且小玲没有那种安装第三方输入法的习惯。但是有一天，小玲在网上看到好多人推荐的 Rime 输入法，抱着试一下的想法，小玲试用了一下这款输入法。没想到这款输入法的可定制性真的高。小玲从此就离不开这款输入法了。现在小玲也来向大家传教。

## 安装

在 Windows 上，这款输入法有一个专属的名字——小狼毫（Weasel）。

小玲是直接采用 [winget]({{< ref "1.md" >}}) 安装的，当然你也可以在 Github [下载](https://github.com/rime/weasel/releases/latest) 这款输入法，小狼毫是开源的。

```powershell
winget install --id=Rime.Weasel
```

## 设置用户文件夹

安装完成后，小玲强烈建议你把小狼毫的“用户资料夹”改成一个 C 盘以外的位置。这样重装系统后，只需把“用户资料夹”的位置设置为上一次设置的位置，输入法配置就全部回来啦。

你可以在开始菜单里的“【小狼毫】安装选项”设置位置。

## 删除原来的输入法

假设你原来原来的中文输入法只有微软拼音。这里以 Windows 10 为例，在你安装好后，系统设置里并没有添加小狼毫这个输入法，你需要在“设置”→“时间和语言”→“语言”→“首选语言”→“中文（简体，中国）”→“选项”→“键盘”里把小狼毫给添加进去，然后就可以把微软拼音删除啦。

## 设置输入方案

从现在开始咱们将通过编写配置文件的方式设置小狼毫。虽然小狼毫有一个“输入法设定”的功能。不过那个能设置的东西太少了，小玲也不会教你那么简单的东西。小玲将教你通过编写配置文件的方式设置小狼毫。不过在这之前，你需要熟悉 YAML 文件的语法。

首先，咱们需要打开小狼毫的“用户文件夹”然后在里面新建一个文件，命名为 `default.custom.yaml`，然后写入以下内容。

```yaml
patch:
  schema_list:
    - schema: luna_pinyin
```

切换系统输入法到小狼毫，右击任务栏里的“中”字图标，点击“重新部署”。咱们写的配置就生效啦。

这里的 `schema` 就是输入方案的意思。`luna_pinyin` 是一个输入方案的 ID。你可以在小狼毫的“程序文件夹”里的 `data` 文件夹里找到输入法自带的所有输入方案，以 `schema.yaml` 为文件名的结尾的文件就是一个输入方案定义文件。咱们打开它，里面的 `schema` 对象下的 `schema_id` 对象的值就是它的 ID。

这些是小狼毫 0.14.3 自带的所有输入方案及其 ID。

|        方案ID         |     方案名称      |
| :-------------------: | :---------------: |
|     `luna_pinyin`     |     朙月拼音      |
|      `bopomofo`       |       注音        |
|  `bopomofo_express`   |   注音-快打方式   |
|     `bopomofo_tw`     |   注音-台湾正体   |
|      `cangjie5`       |     仓颉五代      |
|  `cangjie5_express`   | 仓颉五代-快打模式 |
| `luna_pinyin_fluency` |  朙月拼音·语句流  |
|  `luna_pinyin_simp`   |  朙月拼音·简化字  |
|   `luna_pinyin_tw`    | 朙月拼音·台湾正体 |
|    `luna_quanpin`     |       全拼        |
|       `stroke`        |      五笔画       |
|    `terra_pinyin`     |     地球拼音      |

你可以把 `luna_pinyin` 替换成你想要的输入方案的 ID。如果你是全拼用户，那么使用“朙月拼音”就可以啦。如果你是双拼或五笔用户，你还需要下载输入方案，因为小狼毫并没有自带双拼和五笔的输入方案。

这里小玲以微软双拼和五笔为例，介绍小狼毫如何添加双拼和五笔输入方案。

咱们打开 [双拼仓库](https://github.com/rime/rime-double-pinyin)，然后下载 `double_pinyin_mspy.schema.yaml` 文件，将它放到“用户文件夹”里，然后在 `default.custom.yaml` 里添加微软双拼的输入方案 ID。

```yaml
patch:
  schema_list:
    - schema: luna_pinyin
    - schema: double_pinyin_mspy
```

如果你要使用五笔，你需要从 [五笔仓库](https://github.com/rime/rime-wubi) 下载 2 个文件，分别是 `wubi86.dict.yaml` 和 `wubi86.schema.yaml`，将它们放到“用户文件夹”里，然后在 `default.custom.yaml` 里添加五笔的输入方案 ID。当然你也可以替换原来的输入方案 ID。

```yaml
patch:
  schema_list:
    - schema: wubi86
```

最后请不要忘记“重新部署”小狼毫。

## 切换输入方案和简繁体

如果你在使用 2 个或以上的输入方案，你可以通过按 Ctrl + \`键（\`  就是数字键 1 左边的那个键），然后通过数字键或方向键来切换输入方案和简繁体。这个快捷键是可以改的。在 `default.custom.yaml` 文件的 `patch` 对象里增加 `switcher` 对象，然后在 `switcher` 对象里增加 `hotkeys` 对象。`hotkeys` 对象的值是应该是一个数组，里面的一个元素对应一个快捷键，默认是 `Control+grave`（也就是 Ctrl + \`）和 `F4`，小玲自己在用的是 `Control+Shift+space`，也就是 Ctrl + Shift + 空格，小玲觉得默认的有点不太好按。你可以在 [这里](https://github.com/rime/home/wiki/CustomizationGuide#%E4%B8%80%E4%BE%8B%E5%AE%9A%E8%A3%BD%E5%96%9A%E5%87%BA%E6%96%B9%E6%A1%88%E9%81%B8%E5%96%AE%E7%9A%84%E5%BF%AB%E6%8D%B7%E9%8D%B5) 找到更具体的设置介绍。

```yaml
patch:
  switcher:
    hotkeys:
      # - Control+grave
      # - F4
      - Control+Shift+space
```

## 设置皮肤

小狼毫默认自带的皮肤有 36 套，它们都在“程序文件夹”的 `data` 文件夹里的 `weasel.yaml` 文件里的 `preset_color_schemes` 对象里被定义。想要使用哪款皮肤，只需找到它的对象名就可以了。例如下面的 `aqua` 和 `azure` 就是一个皮肤的对象名。

```yaml
preset_color_schemes:
  aqua:
    name: 碧水／Aqua
    author: 佛振 <chen.sst@gmail.com>
    text_color: 0x000000
    back_color: 0xeceeee
    border_color: 0xe0e0e0
    hilited_text_color: 0x000000
    hilited_back_color: 0xd4d4d4
    hilited_candidate_text_color: 0xffffff
    hilited_candidate_back_color: 0xfa3a0a

  azure:
    name: 青天／Azure
    author: 佛振 <chen.sst@gmail.com>
    text_color: 0xffe8ca
    candidate_text_color: 0xfff8ee
    back_color: 0x8b4e01
    border_color: 0x8b4e01
    hilited_text_color: 0xfff8ee
    hilited_back_color: 0x8b4e01
    hilited_candidate_text_color: 0x7ffeff
    hilited_candidate_back_color: 0xa95e01
    comment_text_color: 0xc69664
```

然后，咱们需要新建在“用户文件夹”里新建一个文件，命名为 `weasel.custom.yaml`，然后写入以下内容。比如小玲现在想使用“青天／Azure”这个皮肤，小玲就把 `patch` 对象下的 `style/color_scheme` 对象的值设为 `azure`。

```yaml
patch:
  "style/color_scheme": azure
```

这里小玲使用的是上面这种写法，而不是使用下面这种写法。

```yaml
patch:
  style:
    color_scheme: azure
```

因为下面这种写法——会使原本 `style` 里原本已经定义的其他对象变为未定义的。下面是“程序文件夹”的 `data` 文件夹里的 `weasel.yaml` 文件里的 `style` 对象。这些是小狼毫默认的样式设置。

```yaml
style:
  color_scheme: aqua
  font_face: Microsoft YaHei
  font_point: 14
  horizontal: false
  fullscreen: false
  inline_preedit: false
  preedit_type: composition
  display_tray_icon: false
  label_format: "%s."
  layout:
    min_width: 160
    min_height: 0
    border_width: 3
    margin_x: 12
    margin_y: 12
    spacing: 10
    candidate_spacing: 5
    hilite_spacing: 4
    hilite_padding: 2
    round_corner: 4
```

如果咱们使用下面这种写法，因为咱们写了 `color_scheme` 对象，它会覆盖输入法默认的 `color_scheme` 对象的值：`aqua`，但是咱们并没有写 `font_face`、`font_size` 等对象，所以它们会变成未定义的，而不是会继承下来。如果写成上面这种写法，会被覆盖的对象就会只有 `style` 对象里的 `color_scheme` 对象。`style` 对象里的其他对象将会被继承下来。

如果把 `weasel.custom.yaml` 写成这个样子，也是可以的。它会覆盖 `style` 对象里的所有对象。

```yaml
patch:
  style:
    color_scheme: azure
    font_face: Microsoft YaHei
    font_point: 14
    horizontal: false
    fullscreen: false
    inline_preedit: false
    preedit_type: composition
    display_tray_icon: false
    label_format: "%s."
    layout:
      min_width: 160
      min_height: 0
      border_width: 3
      margin_x: 12
      margin_y: 12
      spacing: 10
      candidate_spacing: 5
      hilite_spacing: 4
      hilite_padding: 2
      round_corner: 4
```

另外 `style` 对象里最值得一提的是 `horizontal` 对象，它控制着输入法的候选框是竖排的还是横排的。在 `weasel.custom.yaml` 文件里的 `patch` 对象里加入 `style/horizontal` 对象，并把它的值设为 `true`。这样输入法的候选框就变成横排的啦。

```yaml
patch:
  "style/horizontal": true
```

那么如何添加第三方皮肤呢？下面是小玲在网上找到的 2 款第三方皮肤的配置[^1]。把它写进 `weasel.custom.yaml` 文件里的 `patch` 对象就可以啦。

```yaml
patch:
  "preset_color_schemes/placeless":
    author: "jed <placeless@outlook.com>"
    back_color: 0xFFFFFF
    candidate_text_color: 0x000000
    hilited_candidate_back_color: 0xf57c75
    hilited_candidate_text_color: 0xFFFFFF
    name: "秋田／Placeless"
    text_color: 0x000000
  "preset_color_schemes/placeless2":
    author: "jed <placeless@outlook.com>"
    back_color: 0xFFFFFF
    candidate_text_color: 0x666666
    hilited_candidate_back_color: 0xFFFFFF
    hilited_candidate_text_color: 0xf57c75
    name: "荷田／Placeless"
    text_color: 0x000000
```

[^1]: 这些配置来自 《[我的鼠须管配置](https://placeless.net/blog/my-rime-squirrel-config)》，为了兼容小狼毫，小玲修改了一点点。

可以看到这里定义皮肤时使用的写法是

```yaml
patch:
  "preset_color_schemes/placeless":
    # 省略
```

而不是

```yaml
patch:
  preset_color_schemes:
    placeless":
      # 省略
```

至于原因，小玲相信你也明白了。这是为了防止输入法自带的皮肤配置变成未定义的。

要使用这个第三方皮肤，只需把 `weasel.custom.yaml` 文件里的 `patch` 对象里的 `style/color_scheme` 对象的值设为 `placeless` 就可以啦。

```yaml
patch:
  "style/color_scheme": placeless
```

## 设置快捷键

如果你用过微软拼音，你可能知道 Ctrl + 空格是切换中英文的快捷键。但是使用了小狼毫后再按 Ctrl + 空格会导致输入法被禁用。那么如何解决这个问题呢？

你可以将下面的代码保存成一个 reg 文件，将它导入到你的注册表里[^2]，然后重启电脑。这样按 Ctrl + 空格就不会导致输入法被禁用啦。小玲实测 Windows 10 和 Windows 11 都可以使用这个方法。

[^2]: 方法参考 [keyboard shortcuts - CTRL-Space always toggles Chinese IME (Windows 7) - Super User](https://superuser.com/questions/327479/ctrl-space-always-toggles-chinese-ime-windows-7/480723#480723)。

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Control Panel\Input Method\Hot Keys\00000010]
"Key Modifiers"=hex:00,c0,00,00
"Virtual Key"=hex:ff,00,00,00

[HKEY_CURRENT_USER\Control Panel\Input Method\Hot Keys\00000070]
"Key Modifiers"=hex:00,c0,00,00
"Virtual Key"=hex:ff,00,00,00
```

现在按 Ctrl + 空格还不能切换中英文，咱们需要在 `default.custom.yaml` 文件里的 `patch` 对象里增加 `key_binder` 对象。具体配置如下。

```yaml
patch:
  key_binder:
    bindings:
      - accept: Control+space
        toggle: ascii_mode
        when: always
```

经过添加上面的配置，现在咱们可以通过 Ctrl + 空格切换中英文啦。

小玲还自己设置了一些快捷键，比如下面这些。

```yaml
patch:
  key_binder:
    bindings:
      - accept: Control+h
        send: Up
        when: composing
      - accept: Control+j
        send: Page_Down
        when: composing
      - accept: Control+k
        send: Page_Up
        when: composing
      - accept: Control+l
        send: Down
        when: composing
```

这 4 个快捷键的作用是在出现候选框时，可以通过按 Ctrl + H 和 Ctrl + L 移动选中的词，通过按 Ctrl + J 和 Ctrl + K 向下或向上翻页。如果把候选框变成横排的将会更直观。这可是微软拼音做不到的哦。因为小玲打字时是使用标准指法的（也就是左右手的食指分别放在 F 和 J 上）。以前用微软拼音时，要用左手手指伸很远去按数字键选词，现在只需通过左手小指按住左边的 Ctrl 键，然后右手几乎不用移动就可以选词，感觉方便多啦。

## 输入符号和短语

小狼毫默认的符号配置在“程序文件夹”的 `data` 文件夹里的 `symbols.yaml` 文件里。你可以打开这个文件查看你能输入哪些符号和短语。比如要输入一些特殊符号，就在中文输入模式下打 `/fh`，候选框就会出现特殊符号了。

## 结尾

好了，通过上面的学习，相信你已经可以掌握小狼毫的基本使用方法了。如果还有什么不懂的，可以查看小狼毫的 [官方文档](<https://github.com/rime/home/wiki>)，这篇文档比小玲写的要好多了。
