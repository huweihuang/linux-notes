# Linux 学习笔记

> 本系列是 [Linux 学习笔记](https://www.huweihuang.com/linux-notes/)
> 
> 更多的学习笔记请参考：
> - [Kubernetes 学习笔记](https://www.huweihuang.com/kubernetes-notes/)
> - [Golang 学习笔记](https://www.huweihuang.com/golang-notes/)
> - [Linux 学习笔记](https://www.huweihuang.com/linux-notes/)
> - [数据结构学习笔记](https://www.huweihuang.com/data-structure-notes/)
>
> 个人博客：[www.huweihuang.com](https://www.huweihuang.com/)

---

# 目录

## 计算

* [CPU]() 

* [内存]() 

## 存储

* [Linux 文件系统]()
  * [Linux 介绍](file/linux-introduction.md)
  * [文件系统](file/linux-file-system.md)
  * [文件存储结构](file/linux-file-storage.md)
  * [文件权限](file/linux-file-permission.md)
  * [Linux常用命令](file/linux-command.md)
* [磁盘]()
  * [LVM的使用](disk/lvm-usage.md)
  * [磁盘命令](disk/disk-command.md)
  * [Raid介绍](disk/disk-raid.md)
  * [创建硬件Raid](disk/make-hardware-raid.md)

## 网络

* [TCP/IP]()
  * [TCP/IP基础](tcpip/tcpip-basics.md) 
  * [IP协议](tcpip/ip.md) 
  * [TCP协议](tcpip/tcp.md)
  * [UDP协议](tcpip/udp.md)
  * [HTTP协议](tcpip/http.md)
* [网络命令]()    
  * [iptables介绍](network/iptables.md)    
  * [iptables命令](network/iptables-command.md)    
  * [tcpdump命令](network/tcpdump.md)    
  * [wrk压测命令](network/wrk-usage.md)      
  * [网卡Bonding介绍](network/bond.md)
  * [netplan介绍](network/netplan.md)
  * [VLAN介绍](network/vlan.md)

## 程序

* [进程]()

* [Shell 脚本]()
  * [Shell简介](shell/shell-introduction.md) 
  * [Shell变量](shell/shell-var.md) 
  * [Shell运算符](shell/shell-char.md) 
  * [Shell数组](shell/shell-array.md) 
  * [Shell echo命令](shell/shell-echo.md) 
  * [Shell判断语句](shell/shell-if.md) 
  * [Shell循环语句](shell/shell-loop.md) 
  * [Shell函数](shell/shell-function.md) 
  * [Shell重定向](shell/shell-stdout.md) 

---

## 运维工具

* [Ansible的使用](tools/ansible-usage.md)
* [Supervisor的使用](tools/supervisor-usage.md)
* [Confd的使用](tools/confd-usage.md)
* [NFS的使用](tools/nfs-usage.md)
* [ceph-fuse的使用](tools/ceph-fuse.md)
* [ssh tips](tools/ssh-tips.md)

## Git管理

* [Git 介绍](git/git.md) 
* [Git 常用命令](git/git-common-cmd.md) 
* [Git 命令分类](git/git-commands.md) 
* [Git commit规范](git/git-commit-msg.md) 
* [Git 命令别名](git/git-alias-zsh.md) 

## 数据库

* [Mysql]()
  * [系统管理](mysql/system-manage.md) 
  * [数据表操作](mysql/table-operation.md) 
  * [表内容操作](mysql/curd-commands.md) 
  * [Mysql服务部署](mysql/deploy-mysql.md)
* [Redis]()
  * [Redis介绍](redis/redis-introduction.md) 
  * [Redis集群模式部署](redis/redis-cluster.md) 
  * [Redis主从及哨兵模式部署](redis/redis-sentinel.md) 
  * [Redis配置详解(中文版)](redis/redis-conf-cn.md) 
  * [Redis配置详解(英文版)](redis/redis-conf-en.md) 
* [Memcached]()
  * [Memcached的使用](memcached/memcached.md) 
  * [Memcached命令](memcached/memcached-cmd.md) 

## 网络工具

* [Nginx]()
  * [Nginx安装与配置](nginx/install-nginx.md) 
  * [Nginx作为反向代理](nginx/nginx-proxy.md) 
  * [Nginx http服务器](nginx/nginx-http.md) 
  * [配置Nginx免费证书](nginx/config-ssl-for-nginx.md) 
* [Keepalived]()
  * [Keepalived简介](keepalived/keepalived-introduction.md) 
  * [Keepalived的安装与配置](keepalived/install-keepalived.md) 
  * [Keepalived的相关操作](keepalived/keepalived-operation.md) 
  * [Keepalived的配置详解](keepalived/keepalived-conf.md) 
* [Baremetal]()
  * [bmc概念](baremetal/bmc.md) 
  * [格式化磁盘分区](baremetal/format-disk.md)
  * [Redfish API](baremetal/redfish-api.md)

## 工具技巧

* [快捷键]()
  * [vscode快捷键](keymap/vscode-keymap.md)
  * [eclipse快捷键](keymap/eclipse-keymap.md)
  * [chrome快捷键](keymap/chrome-keymap.md)
  * [tmux快捷键](keymap/tmux-keymap.md)
  * [iterm2 rzsz的使用](keymap/iterm2-rzsz.md)
* [IDE工具]()
  * [vscode配置](ide/vscode.md) 
  * [Goland配置](ide/goland.md) 
* [vim]()
  * [vim命令](ide/vim/vim-keymap.md) 
  * [vimrc配置](ide/vim/vimrc-cn.md) 
  * [basic vimrc](ide/vim/basic-vimrc.md) 
* [博客问题记录](keymap/blog.md)

## 大模型

* [基于Ollama构建本地大模型](LLM/build-ollama-openwebui.md)


---

# 赞赏

> 如果觉得文章有帮助的话，可以打赏一下，谢谢！

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1551599963/blog/donate.jpg" width="70%"/>
