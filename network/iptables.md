## 添加iptables规则

```bash
# 单个端口
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 多个端口
iptables -A INPUT -p tcp -m multiport --dports 6443,8443,2379,2380,10250 -j ACCEPT
```

## 删除iptables规则

```bash
# 显示iptables规则行号
iptables -nL --line-numbers

# 删除某行规则
iptables -D INPUT 11
```

## 持久化iptables（重启仍生效）

持久化iptables规则，添加规则到文件中/etc/sysconfig/iptables

```bash
# vi /etc/sysconfig/iptables

-A INPUT -p vrrp -j ACCEPT
-A OUTPUT -p vrrp -j ACCEPT
```

或者

```bash
apt-get install iptables-persistent

netfilter-persistent save
netfilter-persistent reload

# 生成的规则存储在以下文件
/etc/iptables/rules.v4
/etc/iptables/rules.v6
```
