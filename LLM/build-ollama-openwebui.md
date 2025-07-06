---
title: "基于Ollama构建本地大模型"
weight: 1
catalog: true
date: 2025-2-16 10:50:57
subtitle:
header-img:
tags:
- 大模型
catagories:
- 大模型
---

本文主要介绍如何通过`Ollama`和`OpenWebUI`来搭建一个本地私有化运行的大模型工具。私有化大模型的构建主要用于解决`数据的安全性问题`，对于大部分私有数据不适合通过外部的大模型网站来上传和分析。

# 1. Ollama 与 OpenWebUI 介绍

## 1.1. Ollama简介

Ollama 是一个 **本地运行的 AI 大模型管理工具**，可以让你在本地 **快速拉取、管理和运行** 各种开源大语言模型（如 LLaMA、Mistral、deepseek 等），而无需依赖云端 API。它的主要特点包括：

- **简易安装**：支持 macOS、Linux 和 Windows（WSL）。
- **本地推理**：在本地设备上直接运行 LLM，保护数据隐私。
- **模型管理**：可以像使用 Docker 一样 `ollama run llama2` 轻松拉取和运行模型。
- **自定义模型**：支持通过 `Modelfile` 进行微调和定制。
- **支持 API**：可以通过 Python、Node.js 等语言调用 Ollama 提供的本地 REST API。

Ollama 适用于本地 AI 代理、嵌入式 AI 应用、隐私保护的智能助手等场景。你可以用它来运行大语言模型，而无需自己搭建复杂的推理环境。

## 1.2. OpenWebUI简介

**Open-WebUI** 是一个 **开源的 Web 用户界面**，用于管理和使用本地或远程的大语言模型（LLM），比如 Ollama、OpenAI、Gemini 等。它的主要特点包括：

- **友好的 Web 界面**：提供 ChatGPT 类似的对话 UI，方便交互。
- **支持多种后端**：可以连接 **Ollama、OpenAI API、本地 LLM** 等。
- **多用户支持**：适用于团队协作。
- **对话历史管理**：可保存和管理聊天记录。
- **插件和自定义功能**：支持扩展，适用于不同应用场景。

它可以让本地 LLM 变得更加易用，适合个人、企业部署本地 AI 助手。

# 2. 部署ollama

## 2.1. 脚本安装`ollama`

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

输出

```bash
>>> Installing ollama to /usr/local
>>> Downloading Linux amd64 bundle
######################################################################## 100.0%
>>> Creating ollama user...
>>> Adding ollama user to render group...
>>> Adding ollama user to video group...
>>> Adding current user to ollama group...
>>> Creating ollama systemd service...
>>> Enabling and starting ollama service...
Created symlink /etc/systemd/system/default.target.wants/ollama.service -> /etc/systemd/system/ollama.service.
>>> The Ollama API is now available at 127.0.0.1:11434.
>>> Install complete. Run "ollama" from the command line.
WARNING: No NVIDIA/AMD GPU detected. Ollama will run in CPU-only mode.
```

默认服务监听的地址为：`127.0.0.1:11434`

## 2.2. 查看`ollama`服务状态

```bash
systemctl status ollama
* ollama.service - Ollama Service
     Loaded: loaded (/etc/systemd/system/ollama.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-02-07 17:21:55 +08; 23s ago
   Main PID: 53472 (ollama)
      Tasks: 10
     Memory: 10.3M
     CGroup: /system.slice/ollama.service
             `-53472 /usr/local/bin/ollama serve
```

查看`ollama`命令

```bash
# ollama --help
Large language model runner

Usage:
  ollama [flags]
  ollama [command]

Available Commands:
  serve       Start ollama
  create      Create a model from a Modelfile
  show        Show information for a model
  run         Run a model
  stop        Stop a running model
  pull        Pull a model from a registry
  push        Push a model to a registry
  list        List models
  ps          List running models
  cp          Copy a model
  rm          Remove a model
  help        Help about any command

Flags:
  -h, --help      help for ollama
  -v, --version   Show version information

Use "ollama [command] --help" for more information about a command.
```

## 2.3. 拉取一个大模型

可以在 https://ollama.com/search 网站上，选择一个所需要的大模型，例如`deepseek-r1:7b`。

```bash
# 下载指定的大模型，例如deepseek
# ollama pull deepseek-r1:7b
pulling manifest
pulling 96c415656d37... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏ 4.7 GB
pulling 369ca498f347... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏  387 B
pulling 6e4c38e1172f... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏ 1.1 KB
pulling f4d24e9138dd... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏  148 B
pulling 40fb844194b2... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏  487 B
verifying sha256 digest
writing manifest
success

# ollama list
NAME              ID              SIZE      MODIFIED
deepseek-r1:7b    0a8c26691023    4.7 GB    12 minutes ago
```

## 2.4. 运行大模型

```bash
# ollama run deepseek-r1:7b
>>> 你是谁
<think>

</think>

您好！我是由中国的深度求索（DeepSeek）公司开发的智能助手DeepSeek-R1。如您有任何任何问题，我会尽我所能为您提供帮助。

>>> /bye

# 查看正在运行的模型
# ollama ps
NAME              ID              SIZE      PROCESSOR    UNTIL
deepseek-r1:7b    0a8c26691023    5.5 GB    100% CPU     3 minutes from now
```

## 2.5. 修改ollama服务地址和目录

### 2.5.1. 修改ollama服务地址

ollama服务默认监听127.0.0.1, 如果要修改监听地址，则可以添加`Environment="OLLAMA_HOST=0.0.0.0:11434"`。

```bash
vi /etc/systemd/system/ollama.service

[Unit]
Description=Ollama Service
After=network-online.target

[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"   # 增加环境变量
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"

[Install]
WantedBy=default.target

# 重启服务
systemctl daemon-reload
systemctl restart ollama
systemctl status ollama
```

### 2.5.2. 修改ollama数据目录

参考：[ollama/docs/faq.md](https://github.com/ollama/ollama/blob/main/docs/faq.md#where-are-models-stored)

默认存储目录

- macOS: `~/.ollama/models`
- Linux: `/usr/share/ollama/.ollama/models`
- Windows: `C:\Users\%username%\.ollama\models`

以linux系统为例，修改默认的存储目录：

```bash
dir="/data/ollama/models"

# 创建目录并分配权限
mkdir -p /data/ollama/models
sudo chown -R ollama:ollama /data/ollama/models

# 添加环境变量OLLAMA_MODELS
vi /etc/systemd/system/ollama.service

[Service]
Environment="OLLAMA_MODELS=/data/ollama/models"

# 重启服务
systemctl daemon-reload
systemctl restart ollama
systemctl status ollama

# 迁移数据
cp -fr /usr/share/ollama/.ollama/models/* /data/ollama/models
sudo chown -R ollama:ollama /data/ollama/models
```

# 3. 部署open-webui

## 3.1. 单独部署open-webui

如果已经部署了ollama服务，可以通过以下命令单独部署open-webui，修改`OLLAMA_BASE_URL`为ollama的服务地址。如果使用host-network，默认服务监听端口为`8080`。

```bash
docker run -d --network=host -e OLLAMA_BASE_URL=http://OLLAMA_HOST:11434 -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

环境变量

- `OLLAMA_BASE_URL:http://OLLAMA_HOST:11434` : 设置ollama服务的地址
- `HF_HUB_OFFLINE: "1"`：设置模型为离线的环境
- `ENABLE_OPENAI_API: "false"`：设置关闭openai的接口

**访问open-webui服务：**

访问`http://服务器IP:8080`，注册用户名密码然后登录。就可以使用本地的大模型服务。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1739864579/article/linux/llm/open-webui.png)

## 3.2. 部署open-webui和ollama服务

如果不想单独部署ollama，可以通过open-webui:ollama镜像，同时部署open-webui和ollama，两个服务集成在同一个镜像中。

```bash
# 下载镜像
docker pull ghcr.io/open-webui/open-webui:ollama
# 运行open-webui:ollama
docker run -d -p 3000:8080 -v ollama:/root/.ollama -v ollama-open-webui:/app/backend/data --name ollama-open-webui --restart always ghcr.io/open-webui/open-webui:ollama
```

查看服务

```bash
# docker images
REPOSITORY                      TAG       IMAGE ID       CREATED        SIZE
ghcr.io/open-webui/open-webui   ollama    29d60b4958c8   4 days ago     8.02GB

# docker ps
CONTAINER ID   IMAGE                                  COMMAND           CREATED          STATUS                    PORTS                              NAMES
3175fc20c608   ghcr.io/open-webui/open-webui:ollama   "bash start.sh"   16 minutes ago   Up 16 minutes (healthy)   0.0.0.0:3000->8080/tcp             ollama-open-webui
```

登录容器下载大模型文件

```bash
# 登录容器
# docker exec -it 3175fc20c608 bash

# 下载指定的大模型
# ollama pull deepseek-r1:7b
pulling manifest
pulling 96c415656d37... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏ 4.7 GB
pulling 369ca498f347... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏  387 B
pulling 6e4c38e1172f... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏ 1.1 KB
pulling f4d24e9138dd... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏  148 B
pulling 40fb844194b2... 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████▏  487 B
verifying sha256 digest
writing manifest
success

# ollama list
NAME              ID              SIZE      MODIFIED
deepseek-r1:7b    0a8c26691023    4.7 GB    12 minutes ago
```

则可以访问所部属服务器的地址和端口来访问open-webui的服务。

## 3.3. 构建本地知识库

### 3.3.1. 自定义文件分析

可以通过页面上传本地的知识库文件，让AI回答关于自定义文件中的内容。

例如：我通过文件自定义了内容，提问张飞的电话号码，则可以通过文章中的内容来回答。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1739871922/article/linux/llm/phone-chat.png)

其中自定义文档的内容如下：

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1739869289/article/linux/llm/ollama-docs.png)

同样可以上传其他文件来构建一个本地大模型知识库。然后借助大模型来查询和分析数据内容。

### 3.3.2. 本地化数据存储

其中open-webui的本地化数据存储在容器内的`/app/backend/data/`目录下。

```bash
/app/backend/data# ls -l
total 236
drwxr-xr-x 7 root root   4096 Feb 11 10:48 cache
drwxr-xr-x 2 root root   4096 Feb 18 06:11 uploads
drwxr-xr-x 3 root root   4096 Feb 18 06:11 vector_db
-rw-r--r-- 1 root root 229376 Feb 18 06:38 webui.db

# 可以从uploads目录看到上传的本地文件
/app/backend/data/uploads# cat 117e6f99-0657-40d1-ab6f-1bea81e78053_ollama-docs.md
张飞的电话号码是u987438274

曹操的电话号码是123456

关羽的电话号码是5352345
```

## 3.4. FAQ

### 1）open-webui页面无法选择模型

**问题：**

当单独部署open-webui，可能会遇到open-webui页面无法选择模型具体的现象如下：

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1739864579/article/linux/llm/open-webui-error.png" title="" alt="" width="709">

open-webui日志报错：

```bash
INFO  [open_webui.routers.ollama] get_all_models()
ERROR [open_webui.routers.ollama] Connection error: Cannot connect to host 1.1.1.1:11434 ssl:default [Connect call failed ('1.1.1.1', 11434)]
```

**原因：**

按官网命令使用`端口映射`的网络模式，如果OLLAMA_BASE_URL配置为127.0.0.1则访问不到单独部署的ollama服务，如果改用具体的ollama的IP也可能存在访问失败的问题。

```bash
docker run -d -p 3000:8080 -e OLLAMA_BASE_URL=https://example.com -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

**解决方案：**

docker网络模式改为`host-network`的网络模式

```bash
docker run -d --network=host -e OLLAMA_BASE_URL=http://OLLAMA_HOST:11434 -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

### 2）数据目录没权限permission denied

如果用户修改了ollama的models的存储目录，出现ollama服务重启失败，或者pull model数据报错

```bash
# 修改ollama的model目录后ollama服务重启报错
Error: mkdir /data/ollama: permission denied

# 迁移model数据后出现没权限，因为使用了root命令执行
cp -fr /usr/share/ollama/.ollama/models/* /data/ollama/models
# ollama pull deepseek-r1:70b
writing manifest
Error: open /data/ollama/models/manifests/registry.ollama.ai/library/deepseek-r1/70b: permission denied
```

**原因：**

 ollama默认使用的用户名是 ollama，因此需要给目录添加用户的权限，例如：目录创建和model文件迁移是通过root或其他用户执行的。

```bash
sudo chown -R ollama:ollama /data/ollama/models
```

# 4. 总结

本文主要介绍了ollama和open-webui的部署，从而搭建一个`本地化私有的大模型工具`，`所有的数据都存储在本地`。可以通过上传文件来分析本地的数据，类似构建`本地大模型知识库`。

不过本地大模型的响应速度依赖于大模型本身和本地的资源，包括cpu和gpu，没有gpu资源也可以运行。在资源较小的情况下，大模型回答问题的速度比较慢。如果完全需要离线的大模型分析数据，在资源受限的情况下需要再进一步做优化才能得到比较好的体验。

参考：

- https://ollama.com/download/linux
- https://github.com/ollama/ollama/blob/main/docs/faq.md
- https://docs.openwebui.com/getting-started/quick-start
- https://github.com/open-webui/open-webui#troubleshooting
