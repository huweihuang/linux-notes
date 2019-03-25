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
    state BACKUP
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

# 3. 注意事项

1、指定Nginx健康检测脚本：/etc/keepalived/scripts/check_nginx.sh

2、主备配置差别主要为（建议这么配置）：

> 以下两种方式的配置，当其中一台机器keepalived挂掉后会自动VIP切到另一台机器，当挂掉机器keepalived恢复后不会抢占VIP，该方式可以避免机器恢复再次切VIP所带来的影响。

- 主机:(state BACKUP;priority 100)
- 备机：(state BACKUP;priority 99)
- 非抢占：nopreempt
  
  或者：

- 主机:(state MASTER;priority 100)
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

7、virtual_router_id 15值，主备值一致，但建议不应与集群中其他Nginx机器上的相同，如果同一个网段配置的virtual_router_id 重复则会报错，选择一个不重复的0~255之间的值，可以用以下命令查看已存在的vrid。

```bash
tcpdump -nn -i any net 224.0.0.0/8
```

# 4. 常用脚本

## 4.1. Nginx健康检测脚本

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

### 4.1.1. 检查接口调用是否为200

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

### 4.1.2. 检查Nginx进程是否运行

```shell
#!/bin/sh
if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        echo "$(date) nginx pid not found">>/etc/keepalived/keepalived.log
        killall keepalived
fi

```

## 4.2. Keepalived状态通知脚本

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

