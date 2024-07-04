---
title: "配置Nginx免费证书"
weight: 4
catalog: true
date: 2024-04-06 10:50:57
subtitle:
tags:
- Nginx
catagories:
- Nginx
---

# 1. 介绍

网站的 SSL/TLS 加密会为您的用户带来更靠前的搜索排名和更出色的安全性。但是最大障碍是证书获取成本高昂和所涉人工流程繁琐。

Let’s Encrypt 是一家免费、开放、自动化的证书颁发机构 (CA)。本文介绍了如何使用 Let’s Encrypt 客户端生成证书，以及如何自动配置 NGINX 开源版和 NGINX Plus 以使用这些证书。

# 2. 安装certbot

```bash
apt-get update
sudo apt-get install certbot
apt-get install python3-certbot-nginx
```

# 3. 为域名生成证书

执行以下命令会生成一个`90天到期`的证书文件。

```bash
sudo certbot --nginx -d www.example.com
```

以上命令会在`/etc/letsencrypt/live/`生成证书文件。

```bash
cd /etc/letsencrypt/live/www.example.com
ls
README  cert.pem  chain.pem  fullchain.pem  privkey.pem
```

如果配置成功会生成以下信息：

```bash
Congratulations! You have successfully enabled https://example.com and https://www.example.com 

-------------------------------------------------------------------------------------
IMPORTANT NOTES: 

Congratulations! Your certificate and chain have been saved at: 
/etc/letsencrypt/live/example.com/fullchain.pem 
Your key file has been saved at: 
/etc/letsencrypt/live/example.com//privkey.pem
Your cert will expire on 2017-12-12.
```

并且certbot会自动为**domain‑name.conf**文件自行修改证书路径。

```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    server_name  example.com www.example.com;

    listen 443 ssl; # managed by Certbot

    # RSA certificate
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot

    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

    # Redirect non-https traffic to https
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    } # managed by Certbot
}
```

# 3. 重启Nginx

```bash
nginx -t && nginx -s reload
```

# 4. 自动更新证书

Let’s Encrypt 证书将在 90 天后到期， 因此设置定时任务自动更新证书。

```bash
crontab -e

# 将以下信息写入到crontab文件中
0 12 * * * /usr/bin/certbot renew --quiet
```


参考：

- [更新：为 NGINX 配置免费的 Let’s Encrypt SSL/TLS 证书](https://www.nginx-cn.net/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/)


