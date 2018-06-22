---
title: 【VSCode插件】background添加编辑器背景
date: 2018-06-22 11:48:26
tags:
    - tool
    - vscoode
category:
    - Tool
    - VSCode
toc: true
---

VScode编辑器对中文支持很好，插件丰富，主题也好看，所以目前已经由sublime转投Vscode了。在插件搜集中找到了可以自定义编辑器背景的插件`background`，炫酷的界面又可以优雅的装个叉了，所以立马上手尝试了一下。也对相关设置和过程进行一下记录。

<!--more-->

# 一、安装插件

## 1.下载地址

VsCode插件Background官方介绍：[探个鲜](!https://marketplace.visualstudio.com/items?itemName=shalldie.background)

## 2.安装扩展

(1) 打开扩展列表

1). 快捷键打开

- 快捷键 `Ctrl+Shift+P`
- 键入`install`找到`安装扩展`

2). 直接打开左侧最下方的图标

(2) 搜索`background`

(3)安装并重新加载

# 二、使用方法

## 1.打开配置文件

1). 快捷键打开

- 快捷键 `Ctrl+Shift+P`
- 键入`settings`找到`首选项:打开设置`

> 快捷键`Ctrl+,`可以直接打开(我的这个快捷键无效)

2). 目录栏 `文件 》 首选项 》 设置`

## 2.配置项参数

```json
// 是否开启背景图显示
"background.enabled": true,
// true-显示默认的图片  false-显示用户自定义的图片
"background.useDefault": false,
// 自定义显示的图片，【路径要用双引号】
"background.customImages": [
    // 最多设置三张图片，默认显示最上方的图片，当打开多个侧边栏时再依次显示后面的背景图片
    "D:/UserData/My Documents/My Pictures/man.jpg",
    "D:/UserData/My Documents/My Pictures/geek2.jpg",
    "D:/UserData/My Documents/My Pictures/jizhi.jpg",
],
"background.useFront": false,
// 默认透明度是100%，看起来生硬，不够炫酷
"background.style":{
    // 编辑器显示文字，默认在左上角
    "content":"'HELLO WORLD'",
    "pointer-events":"none",
    // 以下都是CSS显示样式设置
    "position":"absolute",
    "top":"0",
    "right":"0",
    "width":"100%",
    "height":"100%",
    "z-index":"99999",
    "background.repeat":"no-repeat",
    "background-size":"contain",
    // 设置透明度
    "opacity":0.1
}
```

每次修改后都需要重启VSCode使修改生效。