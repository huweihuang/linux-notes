---
title: "Linux常用命令"
weight: 5
catalog: true
date: 2025-03-2 10:50:57
subtitle:
header-img:
tags:
- Linux
catagories:
- Linux
---

# 创建用户名和密码

```bash
# 通过写文件创建用户，/home/mybot是用户目录
echo "mybot:x:1001:1001::/home/mybot:/bin/bash" >> /etc/passwd
# 给用户创建密码
echo "mybot:{password}" | chpasswd
# 指定文件系统根目录创建密码，例如：/mnt/dev/sda，一般为外挂文件系统，并没有chroot
echo "mybot:{password}" | chpasswd -R /mnt/dev/sda
# 给用户分配sudo权限，NOPASSWD表示不需要root密码执行sudo
echo "mybot ALL=(root) NOPASSWD: ALL" > /etc/sudoers.d/mybot

# 设置允许ssh用户密码登录
sudo sed -i 's/^#\?PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
# 设置不允许ssh密码登录
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
# 不重启ssh服务使得配置修改生效
sudo systemctl reload sshd
```


