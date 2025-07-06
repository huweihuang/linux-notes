---
title: "博客问题记录"
weight: 6
catalog: true
date: 2025-07-01 10:50:57
subtitle:
header-img:
tags:
- Blog
catagories:
- Blog
---

> 本文主要介绍博客相关问题

# 1. 安装nodejs

参考官方文档：[Node.js — Download Node.js®](https://nodejs.org/zh-cn/download)，通过nvm管理node的不同版本。

```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 22

# Verify the Node.js version:
node -v # Should print "v22.17.0".
nvm current # Should print "v22.17.0".

# Verify npm version:
npm -v # Should print "10.9.2".
```

# 2. nvm的使用

```bash
# 查看node版本
nvm list
       v10.24.1
->     v12.14.0
       v16.20.2

# 安装node版本
nvm install <版本>

# 卸载node
nvm uninstall <版本>

# 设置默认版本
nvm alias default <版本>
```

# 3. 安装hexo

参考：[Hexo](https://hexo.io/zh-cn/)

```bash
npm install hexo-cli -g
```

由于node的版本过新可能会导致hexo使用异常，例如hexo g 生成空白的html文件等。

以下是本人使用可用的相关版本，其中node的版本是`12.14.0`。

```bash
# hexo -v
hexo: 3.9.0
hexo-cli: 4.3.2
os: darwin 24.3.0 15.3.1

node: 12.14.0
v8: 7.7.299.13-node.16
uv: 1.33.1
zlib: 1.2.11
brotli: 1.0.7
ares: 1.15.0
modules: 72
nghttp2: 1.39.2
napi: 5
llhttp: 1.1.4
http_parser: 2.8.0
openssl: 1.1.1d
cldr: 35.1
icu: 64.2
tz: 2019c
unicode: 12.1
```

node相关版本

```bash
$ nvm -v
0.40.3

$ npm -v
6.13.4

$ node -v
v12.14.0
```

# 4. 安装hugo

hugo是跟hexo类似的博客工具。

安装参考：[hugo macOS](https://gohugo.io/installation/macos/)

```bash
brew install hugo
```

或者通过github下载对应的安装包：https://github.com/gohugoio/hugo/releases

hugo由于版本的不兼容问题也可能带来构建报错，以下是我博客使用的版本`v0.101.0`。

```bash
wget https://github.com/gohugoio/hugo/releases/download/v0.101.0/hugo_0.101.0_macOS-ARM64.tar.gz

# hugo version
hugo v0.101.0-466fa43c16709b4483689930a4f9ac8add5c9f66+extended darwin/arm64 BuildDate=2022-06-16T07:09:16Z VendorInfo=gohugoio
```

hugo下使用的node的版本太低可能会导致博客生成的问题，例如本人使用12.14.0会有问题，而需要切换到node的`v16.20.2`的版本

```bash
# nvm use 16.20.2
# node -v        
v16.20.2
```

# 5. 安装gitbook

如果使用gitbook构建博客，则安装gitbook。

```bash
npm install -g gitbook-cli
```

gitbook 可用版本及node版本。

```bash
# gitbook --version
CLI version: 2.3.2
GitBook version: 3.2.3
 
# node -v                   
v12.14.0
```

# 6. 问题

## 6.1. zsh: bad CPU type in executable: node

执行node相关命令出现以上信息报错：`zsh: bad CPU type in executable: node`。

原因是mac的M1/M4系列版本与node较低的版本不兼容导致，例如node 12 以下的版本可能出现上述的问题。可以执行以下命令修复。

```bash
softwareupdate --install-rosetta
```

## 6.2. hexo g 生成空白的html文件

由于node的版本太高不兼容导致，需要降低node的版本，例如本人使用`12.14.0`是可用的，因此安装12.14.0的版本修复。

## 6.3. hugo: Failed to connect to github.com port 443

问题：使用vpn代理导致hugo命令执行时连接github超时。

```bash
Failed to connect to github.com port 443 after 75018 ms: Couldn't connect to server
```

搜索mac代理，找到代理配置的端口，例如：1087。或者打开“设置 -> 网络和Internet -> 代理”，记录下当前的端口号。

执行以下命令。

```bash
# 使用VPN则设置代理
git config --global http.proxy http://127.0.0.1:1087
git config --global https.proxy https://127.0.0.1:1087

# 未使用VPN则取消代理
git config --global unset https.proxy
git config --global unset http.proxy
```

## 6.4. Error: Cannot find module 'prismjs/components/prism-dockerfile.js

问题：使用gitbook build出现以下报错。

```bash
 Error: Cannot find module 'prismjs/components/prism-dockerfile.js 
```

原因是因为存在某些文档这个错误通常是由于某个 Markdown 文件中使用了 Dockerfile 语法高亮，但 GitBook 的 Prism.js 插件没有正确配置 Dockerfile 语言支持导致的。

因此需要找到哪些文档做了该解析配置，例如以下的文档，把文档中的dockerfile格式解析去掉即可：

```bash
   ```dockerfile
   FROM scratch
   COPY hello.wasm /
   ```
```


