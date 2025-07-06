---
title: "vscode使用配置"
weight: 1
catalog: true
date: 2022-12-1 17:26:24
subtitle:
header-img: "https://res.cloudinary.com/dqxtn0ick/image/upload/v1542285471/header/building.jpg"
tags:
- IDE
catagories:
- IDE
---

> 本文主要描述个人使用vscode中的常用插件和配置

# 1. 常用插件

| 插件名称                 | 说明          |
| -------------------- | ----------- |
| Atom One Dark Theme  | 代码风格主题      |
| JetBrains Icon Theme | Icon主题      |
| GitLens              | 显示某行提交记录    |
| Partial Diff         | 通过剪切板diff对比 |
| Go/Python            | 编程语言插件      |

# 2. 常用快捷键

详细快捷键参考：[vscode快捷键](https://blog.huweihuang.com/linux-notes/keymap/vscode-keymap/)

| 功能              | 快捷键                   |
| --------------- | --------------------- |
| 函数跳转和引用         | （command+单击）或者F12     |
| 回到上次跳回原处（跳转前位置） | ctrl + -              |
| 查看接口的实现方法       |                       |
| 打开terminal终端    | Ctrl + `              |
| 多行 块选择编辑        | Shift + option +鼠标选择块 |

# 3. 配置多窗口显示

vscode默认打开一个新项目就需要打开新的窗口，为了避免很多项目同时浏览而打开太多窗口，因此设置在同一个窗口中可以显示和快速切换多个项目。

1、左下角点击配置按钮，选中 settings。

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1747572400/article/linux/ide/vscode-setting.png" title="" alt="" width="413">

2、搜索 `window.native` 选中此选项。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1747572395/article/linux/ide/vscode-windows.native.png)

3、重启并合并窗口。

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1747572942/article/linux/ide/vscode-merge-windows.png" title="" alt="" width="367">

# 4. 配置远程开发

## 4.1. 本地文件传远程开发

1. 安装sftp插件

2. 创建sftp配置文件
   
   创建`.vscode`目录，在目录下创建sftp.json文件，内容如下：
   
   ```json
   {
       "name": "ip",   
       "host": "ip", 
       "protocol": "sftp",
       "port": 22,
       "username": "root",
       "privateKeyPath": "~/.ssh/id_rsa",
       "remotePath": "/home/go/src/projectname",
       "uploadOnSave": true,
       "ignore": [
           ".git",
           ".vscode",
           ".idea",
           ".DS_Store",
           "node_modules"
       ],
       "watcher": {
           "files": "/home/go/src/github.com/projectname/*",
           "autoUpload": true,
           "autoDelete": true
       }
   }
   ```

## 4.2. 远程文件传本地开发

在远程设备上git clone代码，然后在vscode上安装remote的插件，通过remote插件连接远程开发机并打开代码目录。即可进行远程开发。

通过vscode在远程机器上面安装相关的代码插件即可实现代码跳转等操作。