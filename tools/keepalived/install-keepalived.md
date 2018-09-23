# 1. Keepalived的安装

## 1.1. yum install方式

```bash
yum install -y keepalived
```

## 1.2. 安装包编译方式

更多安装包参考：http://www.keepalived.org/download.html

```bash
wget http://www.keepalived.org/software/keepalived-2.0.7.tar.gz
tar zxvf keepalived-2.0.7.tar.gz
cd keepalived-2.0.7
./configure --bindir=/usr/bin --sbindir=/usr/sbin --sysconfdir=/etc --mandir=/usr/share
make && make install
```

# 2. 常用配置

keepalived配置文件路径：`/etc/keepalived/keepalived`。

## 2.1. MASTER（主机配置）

```shell
global_defs {
    router_id proxy-keepalived
}

vrrp_script check_nginx {
    script "/etc/keepalived/scripts/check_nginx.sh" 
    interval 3  
    weight 2   
}

vrrp_instance VI_1 {
    state MASTER
    interface eth2
    virtual_router_id 15
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass xxx
    }
    track_script {
        check_nginx 
    }
    virtual_ipaddress {
        180.101.115.139
        218.98.38.29
    }
    
	nopreempt
	
	notify_master "/etc/keepalived/keepalived_notify.sh master"
	notify_backup "/etc/keepalived/keepalived_notify.sh backup"
	notify_fault "/etc/keepalived/keepalived_notify.sh fault"
	notify_stop "/etc/keepalived/keepalived_notify.sh stop"
}

```

## 2.2. BACKUP（备机配置）

```shell
global_defs {
    router_id proxy-keepalived
}

vrrp_script check_nginx {
    script "/etc/keepalived/scripts/check_nginx.sh" 
    interval 3  
    weight 2   
}

vrrp_instance VI_1 {
    state BACKUP 
    interface eth2
    virtual_router_id 15
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass xxx
    }
    track_script {
        check_nginx 
    }
    virtual_ipaddress {
        180.101.115.139
        218.98.38.29
    }
    
	nopreempt
	
	notify_master "/etc/keepalived/keepalived_notify.sh master"
	notify_backup "/etc/keepalived/keepalived_notify.sh backup"
	notify_fault "/etc/keepalived/keepalived_notify.sh fault"
	notify_stop "/etc/keepalived/keepalived_notify.sh stop"
}

```

## 2.3. 注意事项

1、指定Nginx健康检测脚本：/etc/keepalived/scripts/check_nginx.sh
2、主备配置差别主要为（建议这么配置）：

- 主机:(state MASTER;priority 100)
- 备机：(state BACKUP;priority 99)
- 非抢占：nopreempt
  或者：
- 主机:(state BACKUP;priority 100)
- 备机：(state BACKUP;priority 100)
- 默认抢占

3、指定VIP

```
    virtual_ipaddress {
        180.101.115.139
        218.98.38.29
    }
```

4、可以指定为非抢占：nopreempt，即priority高不会抢占已经绑定VIP的机器。
5、制定绑定IP的网卡： interface eth2
6、可以指定keepalived状态变化通知

```
	notify_master "/etc/keepalived/keepalived_notify.sh master"
	notify_backup "/etc/keepalived/keepalived_notify.sh backup"
	notify_fault "/etc/keepalived/keepalived_notify.sh fault"
	notify_stop "/etc/keepalived/keepalived_notify.sh stop"
```

7、virtual_router_id 15值，主备值一致，但建议不应与集群中其他Nginx机器上的相同

# 3. 常用脚本

## 3.1. Nginx健康检测脚本

在Nginx配置目录下（/etc/nginx/conf.d/）增加health.conf的配置文件,该配置文件用于配置Nginx health的接口。

```shell
server {
    listen       80  default_server;
    server_name  localhost;
    default_type text/html;
    return 200 'Health';  
}
```

Nginx健康检测脚本：/etc/keepalived/scripts/check_nginx.sh

### 3.1.1. 检查接口调用是否为200

```shell
#!/bin/sh
set -x

timeout=30 #指定默认30秒没返回200则为非健康，该值可根据实际调整
 
if [ -n ${timeout} ];then
	httpcode=`curl -sL -w %{http_code} -m ${timeout} http://localhost -o /dev/null`
else
	httpcode=`curl -sL -w %{http_code}  http://localhost -o /dev/null`
fi

if [ ${httpcode} -ne 200 ];then
        echo `date`':  nginx is not healthy, return http_code is '${httpcode} >> /etc/keeperalived/keepalived.log
        killall keepalived
        exit 1
else
        exit 0
fi

```

### 3.1.2. 检查Nginx进程是否运行

```shell
#!/bin/sh
if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        echo "$(date) nginx pid not found">>/etc/keepalived/keepalived.log
        killall keepalived
fi

```

## 3.2. Keepalived状态通知脚本

```shell
#!/bin/bash
set -x

warn_receiver=$1
ip=$(ifconfig bond0|grep inet |awk '{print $2}')
warningInfo="${ip}_keepalived_changed_status_to_$1"
warn-report --user admin --key=xxxx --target=${warn_receiver} ${warningInfo}
echo $(date)  $1 >> /etc/keepalived/status

```

**说明：**

1. ip获取本机IP，本例中IP获取是bond0的IP，不同机器网卡名称不同需要修改为对应网卡名称。
2. 告警工具根据自己指定。

# 4. 常用命令

## 4.1. 查看当前VIP在哪个节点上

```bash
# 查看VIP是否在筛选结果中
ip addr show|grep "scope global"

# 或者 
ip addr show|grep {vip}
```

## 4.2. 查看keepalived的日志

```bash
tail /var/log/messages
```

## 4.3. 抓包命令

```bash
# 抓包
tcpdump -nn vrrp
# 可以用这条命令来查看该网络中所存在的vrid
tcpdump -nn -i any net 224.0.0.0/8
```

## 4.4. VIP操作

```bash
# 解绑VIP
ip addr del  dev 
# 绑定VIP
ip addr add  dev 
```

## 4.5. keepalived 切 VIP

例如将 A 机器上的 VIP 迁移到B 机器上。

### 4.5.1. 停止keepalived服务

停止被迁移的机器（A机器）的keepalived服务。

```bash
systemctl stop keepalived
```

### 4.5.2. 查看日志

解绑 A机器 VIP的日志

```bash
Sep 19 14:28:09 localhost systemd: Stopping LVS and VRRP High Availability Monitor...
Sep 19 14:28:09 localhost Keepalived[45705]: Stopping
Sep 19 14:28:09 localhost Keepalived_vrrp[45707]: VRRP_Instance(twemproxy) sent 0 priority
Sep 19 14:28:09 localhost Keepalived_vrrp[45707]: VRRP_Instance(twemproxy) removing protocol VIPs.
Sep 19 14:28:09 localhost Keepalived_healthcheckers[45706]: Stopped
Sep 19 14:28:10 localhost Keepalived_vrrp[45707]: Stopped
Sep 19 14:28:10 localhost Keepalived[45705]: Stopped Keepalived v1.3.5 (03/19,2017), git commit v1.3.5-6-g6fa32f2
Sep 19 14:28:10 localhost systemd: Stopped LVS and VRRP High Availability Monitor.
Sep 19 14:28:10 localhost ntpd[1186]: Deleting interface #10 bond0, 192.168.99.9#123, interface stats: received=0, sent=0, dropped=0, active_time=6755768 secs
```

绑定 B 机器 VIP的日志

```bash
Sep 17 17:20:25 localhost systemd: Starting LVS and VRRP High Availability Monitor...
Sep 17 17:20:26 localhost Keepalived[34566]: Starting Keepalived v1.3.5 (03/19,2017), git commit v1.3.5-6-g6fa32f2
Sep 17 17:20:26 localhost Keepalived[34566]: Opening file '/etc/keepalived/keepalived.conf'.
Sep 17 17:20:26 localhost Keepalived[34568]: Starting Healthcheck child process, pid=34569
Sep 17 17:20:26 localhost Keepalived[34568]: Starting VRRP child process, pid=34570
Sep 17 17:20:26 localhost Keepalived_vrrp[34570]: Registering Kernel netlink reflector
Sep 17 17:20:26 localhost Keepalived_vrrp[34570]: Registering Kernel netlink command channel
Sep 17 17:20:26 localhost Keepalived_vrrp[34570]: Registering gratuitous ARP shared channel
Sep 17 17:20:26 localhost Keepalived_vrrp[34570]: Opening file '/etc/keepalived/keepalived.conf'.
Sep 17 17:20:26 localhost Keepalived_vrrp[34570]: Truncating auth_pass to 8 characters
Sep 17 17:20:26 localhost Keepalived_vrrp[34570]: VRRP_Instance(twemproxy) removing protocol VIPs.
Sep 17 17:20:26 localhost Keepalived_vrrp[34570]: Using LinkWatch kernel netlink reflector...
Sep 17 17:20:26 localhost Keepalived_vrrp[34570]: VRRP_Instance(twemproxy) Entering BACKUP STATE
Sep 17 17:20:26 localhost Keepalived_vrrp[34570]: VRRP sockpool: [ifindex(4), proto(112), unicast(0), fd(10,11)]
Sep 17 17:20:26 localhost systemd: Started LVS and VRRP High Availability Monitor.
Sep 17 17:20:26 localhost kernel: IPVS: Registered protocols (TCP, UDP, SCTP, AH, ESP)
Sep 17 17:20:26 localhost kernel: IPVS: Connection hash table configured (size=4096, memory=64Kbytes)
Sep 17 17:20:26 localhost kernel: IPVS: Creating netns size=2192 id=0
Sep 17 17:20:26 localhost kernel: IPVS: Creating netns size=2192 id=1
Sep 17 17:20:26 localhost kernel: IPVS: ipvs loaded.
Sep 17 17:20:26 localhost Keepalived_healthcheckers[34569]: Opening file '/etc/keepalived/keepalived.conf'.
```

