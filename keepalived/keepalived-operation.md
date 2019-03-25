# 1. 常用命令

## 1.1. 查看当前VIP在哪个节点上

```bash
# 查看VIP是否在筛选结果中
ip addr show|grep "scope global"

# 或者 
ip addr show|grep {vip}
```

## 1.2. 查看keepalived的日志

```bash
tail /var/log/messages
```

## 1.3. 抓包命令

```bash
# 抓包
tcpdump -nn vrrp
# 可以用这条命令来查看该网络中所存在的vrid
tcpdump -nn -i any net 224.0.0.0/8
```

```bash
# tcpdump -nn -i any net 224.0.0.0/8
# tcpdump -nn vrrp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
14:40:00.576387 IP 192.168.98.57 > 224.0.0.18: VRRPv2, Advertisement, vrid 9, prio 99, authtype simple, intvl 1s, length 20
14:40:01.577605 IP 192.168.98.57 > 224.0.0.18: VRRPv2, Advertisement, vrid 9, prio 99, authtype simple, intvl 1s, length 20
14:40:02.578429 IP 192.168.98.57 > 224.0.0.18: VRRPv2, Advertisement, vrid 9, prio 99, authtype simple, intvl 1s, length 20
14:40:03.579605 IP 192.168.98.57 > 224.0.0.18: VRRPv2, Advertisement, vrid 9, prio 99, authtype simple, intvl 1s, length 20
14:40:04.580443 IP 192.168.98.57 > 224.0.0.18: VRRPv2, Advertisement, vrid 9, prio 99, authtype simple, intvl 1s, length 20
```

## 1.4. VIP操作

```bash
# 解绑VIP
ip addr del  dev 
# 绑定VIP
ip addr add  dev 
```

## 1.5. keepalived 切 VIP

例如将 A 机器上的 VIP 迁移到B 机器上。

### 1.5.1. 停止keepalived服务

停止被迁移的机器（A机器）的keepalived服务。

```bash
systemctl stop keepalived
```

### 1.5.2. 查看日志

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

# 2. 指定keepalived的输出日志文件

## 2.1. 修改 `/etc/sysconfig/keepalived`

将`KEEPALIVED_OPTIONS="-D"`改为`KEEPALIVED_OPTIONS="-D -d -S 0"`。

```bash
# Options for keepalived. See `keepalived --help' output and keepalived(8) and
# keepalived.conf(5) man pages for a list of all options. Here are the most
# common ones :
#
# --vrrp               -P    Only run with VRRP subsystem.
# --check              -C    Only run with Health-checker subsystem.
# --dont-release-vrrp  -V    Dont remove VRRP VIPs & VROUTEs on daemon stop.
# --dont-release-ipvs  -I    Dont remove IPVS topology on daemon stop.
# --dump-conf          -d    Dump the configuration data.
# --log-detail         -D    Detailed log messages.
# --log-facility       -S    0-7 Set local syslog facility (default=LOG_DAEMON)
#

KEEPALIVED_OPTIONS="-D -d -S 0"
```

## 2.2. 修改rsyslog的配置 /etc/rsyslog.conf

在/etc/rsyslog.conf  添加 keepalived的日志路径

```bash
vi /etc/rsyslog.conf
...
# keepalived log
local0.*                                                /etc/keepalived/keepalived.log
```

## 2.3. 重启rsyslog和keepalived

```bash
# 重启rsyslog
systemctl restart rsyslog
 
#  重启keepalived
systemctl restart keepalived
```

# 3. Troubleshooting

## 3.1. virtual_router_id 同网段重复

日志报错如下：

```bash
Mar 09 21:28:28 k8s4 Keepalived_vrrp[8548]: bogus VRRP packet received on eth0 !!!
Mar 09 21:28:28 k8s4 Keepalived_vrrp[8548]: VRRP_Instance(VI-kube-master) ignoring received advertisment...
Mar 09 21:28:43 k8s4 Keepalived_vrrp[8548]: ip address associated with VRID not present in received packet : 192.168.1.10
Mar 09 21:28:43 k8s4 Keepalived_vrrp[8548]: one or more VIP associated with VRID mismatch actual MASTER advert
```

解决方法:

同一网段内LB节点配置的 `virtual_router_id` 值有重复了，选择一个不重复的0~255之间的值，可以用以下命令查看已存在的vrid。

```bash
tcpdump -nn -i any net 224.0.0.0/8
```

