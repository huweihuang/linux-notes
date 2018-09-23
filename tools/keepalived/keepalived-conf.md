# 详细配置说明

keepalived只有一个配置文件`/etc/keepalived/keepalived.conf`。

里面主要包括以下几个配置区域，分别是:

- global_defs
- static_ipaddress
- static_routes
- vrrp_script
- vrrp_instance
- virtual_server

## 1. global_defs区域

主要是配置故障发生时的通知对象以及机器标识。

```bash
global_defs {
    notification_email {        # notification_email 故障发生时给谁发邮件通知
        a@abc.com
        b@abc.com
        ...
    }
    notification_email_from alert@abc.com  # notification_email_from 通知邮件从哪个地址发出
    smtp_server smtp.abc.com    # smpt_server 通知邮件的smtp地址
    smtp_connect_timeout 30     # smtp_connect_timeout 连接smtp服务器的超时时间
    enable_traps                # enable_traps 开启SNMP陷阱（Simple Network Management Protocol）
    router_id host163 # router_id 标识本节点的字条串，通常为hostname，但不一定非得是hostname。故障发生时，邮件通知会用到。
}
```

## 2. static_ipaddress和static_routes区域[可忽略]

static_ipaddress和static_routes区域配置的是是本节点的IP和路由信息。如果你的机器上已经配置了IP和路由，那么这两个区域可以不用配置。其实，一般情况下你的机器都会有IP地址和路由信息的，因此没必要再在这两个区域配置。

```bash
static_ipaddress {
    10.210.214.163/24 brd 10.210.214.255 dev eth0
    ...
}
static_routes {
    10.0.0.0/8 via 10.210.214.1 dev eth0
    ...
}
```

## 3. vrrp_script区域

用来做健康检查的，当时检查失败时会将vrrp_instance的priority减少相应的值。

```bash
vrrp_script chk_http_port {   
    script "</dev/tcp/127.0.0.1/80"       #一句指令或者一个脚本文件，需返回0(成功)或非0(失败)，keepalived以此为依据判断其监控的服务状态。
    interval 1    #健康检查周期
    weight -10   # 优先级变化幅度，如果script中的指令执行失败，那么相应的vrrp_instance的优先级会减少10个点。
}
```

## 4. vrrp_instance和vrrp_sync_group区域

vrrp_instance用来定义对外提供服务的VIP区域及其相关属性。

vrrp_rsync_group用来定义vrrp_intance组，使得这个组内成员动作一致。

```bash
vrrp_sync_group VG_1 {  #监控多个网段的实例
    group {
        inside_network   # name of vrrp_instance (below)
        outside_network  # One for each moveable IP.
        ...
    }
    notify_master /path/to_master.sh      # notify_master表示切换为主机执行的脚本
    notify_backup /path/to_backup.sh      # notify_backup表示切换为备机师的脚本
    notify_fault "/path/fault.sh VG_1"    # notify_fault表示出错时执行的脚本
    notify /path/notify.sh  # notify表示任何一状态切换时都会调用该脚本，且在以上三个脚本执行完成之后进行调用
    smtp_alert  # smtp_alert 表示是否开启邮件通知（用全局区域的邮件设置来发通知）
}

vrrp_instance VI_1 {
    state MASTER # state MASTER或BACKUP，当其他节点keepalived启动时会将priority比较大的节点选举为MASTER，因此该项其实没有实质用途。
    interface eth0  # interface 节点固有IP（非VIP）的网卡，用来发VRRP包
    use_vmac    dont_track_primary # use_vmac 是否使用VRRP的虚拟MAC地址，dont_track_primary 忽略VRRP网卡错误（默认未设置）
    track_interface {# track_interface 监控以下网卡，如果任何一个不通就会切换到FALT状态。（可选项）
        eth0
        eth1
    }
    #mcast_src_ip 修改vrrp组播包的源地址，默认源地址为master的IP
    mcast_src_ip    lvs_sync_daemon_interface eth1 #lvs_sync_daemon_interface 绑定lvs syncd的网卡
    garp_master_delay 10  # garp_master_delay 当切为主状态后多久更新ARP缓存，默认5秒
    virtual_router_id 1   # virtual_router_id 取值在0-255之间，用来区分多个instance的VRRP组播， 同一网段中virtual_router_id的值不能重复，否则会出错
    priority 100 #priority用来选举master的，根据服务是否可用，以weight的幅度来调整节点的priority，从而选取priority高的为master，该项取值范围是1-255（在此范围之外会被识别成默认值100）
    advert_int 1 # advert_int 发VRRP包的时间间隔，即多久进行一次master选举（可以认为是健康查检时间间隔）
    authentication { # authentication 认证区域，认证类型有PASS和HA（IPSEC），推荐使用PASS（密码只识别前8位）
        auth_type PASS  #认证方式
        auth_pass 12345678  #认证密码
    }

    virtual_ipaddress { # 设置vip
        10.210.214.253/24 brd 10.210.214.255 dev eth0
        192.168.1.11/24 brd 192.168.1.255 dev eth1
    }

    virtual_routes { # virtual_routes 虚拟路由，当IP漂过来之后需要添加的路由信息
        172.16.0.0/12 via 10.210.214.1
        192.168.1.0/24 via 192.168.1.1 dev eth1
        default via 202.102.152.1
    }

    track_script {
        chk_http_port
    }

    nopreempt # nopreempt 允许一个priority比较低的节点作为master，即使有priority更高的节点启动
    preempt_delay 300 # preempt_delay master启动多久之后进行接管资源（VIP/Route信息等），并提是没有nopreempt选项
    debug
    notify_master|    notify_backup|    notify_fault|    notify|    smtp_alert
}
```

## 5. virtual_server_group和virtual_server区域

virtual_server_group一般在超大型的LVS中用到，一般LVS用不到这东西。

```bash
virtual_server IP Port {
    delay_loop    # delay_loop 延迟轮询时间（单位秒）
    lb_algo rr|wrr|lc|wlc|lblc|sh|dh  # lb_algo 后端调试算法（load balancing algorithm）
    lb_kind NAT|DR|TUN  # lb_kind LVS调度类型NAT/DR/TUN
    persistence_timeout    #会话保持时间
    persistence_granularity  #lvs会话保持粒度 
    protocol TCP  #使用的协议
    ha_suspend
    virtualhost    # virtualhost 用来给HTTP_GET和SSL_GET配置请求header的
    alpha 
    omega
    quorum   
    hysteresis   
    quorum_up|   
    quorum_down|   
    sorry_server  #备用机，所有realserver失效后启用 
    real_server{   # real_server 真正提供服务的服务器
        weight 1 # 默认为1,0为失效       
        inhibit_on_failure #在服务器健康检查失效时，将其设为0，而不是直接从ipvs中删除
        notify_up|       # real server宕掉时执行的脚本
        notify_down|     # real server启动时执行的脚本
        # HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHECK
        TCP_CHECK {
            connect_timeout 3 #连接超时时间
            nb_get_retry 3 #重连次数
            delay_before_retry 3 #重连间隔时间
            connect_port 23  #健康检查的端口的端口
            bindto  
        }
        
        HTTP_GET|SSL_GET {
            url {# 检查url，可以指定多个
                path          # path 请求real serserver上的路径  
                digest        # 用genhash算出的摘要信息       
                status_code   # 检查的http状态码       
                }
            connect_port         # connect_port 健康检查，如果端口通则认为服务器正常     
            connect_timeout      # 超时时长       
            nb_get_retry         # 重试次数
            delay_before_retry   #  下次重试的时间延迟  
         }
       
         SMTP_CHECK {
            host {
              connect_ip
              connect_port #默认检查25端口
              bindto
            }
            connect_timeout 5
            retry 3
            delay_before_retry 2
            helo_name | #smtp helo请求命令参数，可选
         }
         
         MISC_CHECK {
            misc_path | #外部脚本路径
            misc_timeout #脚本执行超时时间
            misc_dynamic #如设置该项，则退出状态码会用来动态调整服务器的权重，返回0 正常，不修改；返回1，
 #检查失败，权重改为0；返回2-255，正常，权重设置为：返回状态码-2
          }
    }
}
```

