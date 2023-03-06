---
title: "vscode远程开发"
catalog: true
date: 2022-12-1 17:26:24
subtitle:
header-img: "https://res.cloudinary.com/dqxtn0ick/image/upload/v1542285471/header/building.jpg"
tags:
- Linux
catagories:
- Linux
---

# 1. 本地文件传远程开发

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

# 2. 远程文件传本地开发

在远程设备上git clone代码，然后在vscode上安装remote的插件，通过remote插件连接远程开发机并打开代码目录。即可进行远程开发。

通过vscode在远程机器上面安装相关的代码插件即可实现代码跳转等操作。