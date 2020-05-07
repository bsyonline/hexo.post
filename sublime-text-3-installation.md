---
title: Sublime Text 3 安装配置
date: 2017-03-28 19:32:34
tags:
 - Sublime
category: 
 - Wiki
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

Windows 上之前一直用 notepad++ ，后来在 linux 上用了一段时间 atom ，git 的插件用起来很顺手，但是由于长期不关机，导致 atom 经常卡死，所以换 Sublime 试试。

<!-- more -->

### 安装
没用过 Sublime ，所以选择了 Sublime Text 3 。下载安装很简单。[https://www.sublimetext.com/3](https://www.sublimetext.com/3)

```shell
sudo dpkg -i sublime-text_build-3126_amd64.deb
```

### 主题
外观控必须先搞个主题，我挑了一个黑色主题 Afteglow ， [https://github.com/YabataDesign/afterglow-theme](https://github.com/YabataDesign/afterglow-theme) 。

文档详细，配置简单，关键还有 markdown 的配色。

最后附一个 user settings
```json
{
    "color_inactive_tabs": true,
    "theme": "Afterglow-green.sublime-theme",
    "folder_no_icon": false,
    "font_face": "Yahei Consolas Hybrid",
    "font_size": 10,
    "ignored_packages":
    [
        "Vintage"
    ],
    "line_padding_bottom": 5,
    "line_padding_top": 5,
    "sidebar_no_icon": false,
    "sidebar_row_padding_large": true,
    "sidebar_size_11": true,
    "status_bar_brighter": true,
    "tabs_padding_small": true,
    "tabs_small": true,
}
```
关于细节调整，可以在 ```Afterglow-green.sublime-theme``` 中修改。

### 插件
网上推荐的很多，选了一些先用着。

package control [https://packagecontrol.io/installation](https://packagecontrol.io/installation) 这个肯定是要装的，欲善其事，先利器器，顺序不能乱。

安装完，后边的插件安装都是小意思啦。

Alignment 
BracketHighlighter
Emmet
Git
Markdown Editing
Side bar
Themr
CSScomb
ConvertToUFT8




