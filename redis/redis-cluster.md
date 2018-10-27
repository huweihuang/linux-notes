# 1. Redis部署

> 以下以Linux系统为例

## 1.1 下载和编译 

```bash
$ wget http://download.redis.io/releases/redis-4.0.7.tar.gz
$ tar xzf redis-4.0.7.tar.gz
$ cd redis-4.0.7
$ make
```

编译完成后会在`src`目录下生成Redis服务端程序`redis-server`和客户端程序`redis-cli`。

## 1.2 启动服务

**1、前台运行**

```bash
src/redis-server
```

该方式启动默认为`前台方式`运行，使用默认配置。

**2、后台运行**

可以修改`redis.conf`文件的`daemonize`参数为`yes`，指定配置文件启动，例如：

```bash
vi redis.conf

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes
```

指定配置文件启动。

```bash
src/redis-server redis.conf
```

例如：

```bash
#指定配置文件后台启动
[root@kube-node-1 redis-4.0.7]# src/redis-server redis.conf
95778:C 30 Jan 00:44:37.633 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
95778:C 30 Jan 00:44:37.634 # Redis version=4.0.7, bits=64, commit=00000000, modified=0, pid=95778, just started
95778:C 30 Jan 00:44:37.634 # Configuration loaded

#查看Redis进程
[root@kube-node-1 redis-4.0.7]# ps aux|grep redis
root      95779  0.0  0.0 145268   468 ?        Ssl  00:44   0:00 src/redis-server 127.0.0.1:6379
```

更多启动参数如下：

```bash
[root@kube-node-1 src]# ./redis-server --help
Usage: ./redis-server [/path/to/redis.conf] [options]
       ./redis-server - (read config from stdin)
       ./redis-server -v or --version
       ./redis-server -h or --help
       ./redis-server --test-memory <megabytes>

Examples:
       ./redis-server (run the server with default conf)
       ./redis-server /etc/redis/6379.conf
       ./redis-server --port 7777
       ./redis-server --port 7777 --slaveof 127.0.0.1 8888
       ./redis-server /etc/myredis.conf --loglevel verbose

Sentinel mode:
       ./redis-server /etc/sentinel.conf --sentinel
```

## 1.3 客户端测试

```bash
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

# 2. Redis集群部署

Redis的集群部署需要在每台集群部署的机器上安装Redis（可参考上述的[Redis安装] ），然后修改配置以集群的方式启动。

## 2.1 手动部署集群

### 2.1.1 设置配置文件及启动实例

修改配置文件redis.conf，集群模式的最小化配置文件如下：

```bash
#可选操作，该项设置后台方式运行，
daemonize yes

port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

> 更多集群配置参数可参考默认配置文件redis.conf中`Cluster`模块的说明

最小集群模式需要三个master实例，一般建议起六个实例，即三主三从。因此我们创建6个以端口号命名的目录存放实例的配置文件和其他信息。

```bash
mkdir cluster-test
cd cluster-test
mkdir 7000 7001 7002 7003 7004 7005
```

在对应端口号的目录中创建`redis.conf`的文件，配置文件的内容可参考上述的集群模式配置。每个配置文件中的端口号`port`参数改为对应目录的端口号。

复制`redis-server`的二进制文件到`cluster-test`目录中，通过指定配置文件的方式启动`redis`服务，例如：

```bash
cd 7000
../redis-server ./redis.conf
```

如果是以前台方式运行，则会在控制台输出以下信息：

```bash
[82462] 26 Nov 11:56:55.329 * No cluster configuration found, I'm 97a3a64667477371c4479320d683e4c8db5858b1
```

每个实例都会生成一个`Node ID`，类似`97a3a64667477371c4479320d683e4c8db5858b1`，用来作为Redis实例在集群中的唯一标识，而不是通过IP和Port，IP和Port可能会改变，该`Node ID`不会改变。



目录结构可参考：

```bash
cluster-test/
├── 7000
│   ├── appendonly.aof
│   ├── dump.rdb
│   ├── nodes.conf
│   └── redis.conf
├── 7001
│   ├── appendonly.aof
│   ├── dump.rdb
│   ├── nodes.conf
│   └── redis.conf
├── 7002
│   ├── appendonly.aof
│   ├── dump.rdb
│   ├── nodes.conf
│   └── redis.conf
├── 7003
│   ├── appendonly.aof
│   ├── dump.rdb
│   ├── nodes.conf
│   └── redis.conf
├── 7004
│   ├── appendonly.aof
│   ├── dump.rdb
│   ├── nodes.conf
│   └── redis.conf
├── 7005
│   ├── appendonly.aof
│   ├── dump.rdb
│   ├── nodes.conf
│   └── redis.conf
├── redis-cli
└── redis-server
```

### 2.1.2 redis-trib创建集群

Redis的实例全部运行之后，还需要`redis-trib.rb`工具来完成集群的创建，`redis-trib.rb`二进制文件在Redis包主目录下的`src`目录中，运行该工具依赖`Ruby`环境和`gem`，因此需要提前安装。

**1、安装Ruby**

```bash
yum -y install ruby rubygems
```

查看Ruby版本信息。

```bash
[root@kube-node-1 src]# ruby --version
ruby 2.0.0p648 (2015-12-16) [x86_64-linux]
```

由于`centos`系统默认支持Ruby版本为`2.0.0`，因此执行`gem install redis`命令时会报以下错误。

```bash
[root@kube-node-1 src]# gem install redis
Fetching: redis-4.0.1.gem (100%)
ERROR:  Error installing redis:
	redis requires Ruby version >= 2.2.2.
```

解决方法是先安装`rvm`，再升级`ruby`版本。

**2、安装rvm**

```bash
curl -L get.rvm.io | bash -s stable
```

如果遇到以下报错，则执行报错中的`gpg2 --recv-keys `的命令。

```bash
[root@kube-node-1 ~]# curl -L get.rvm.io | bash -s stable
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    335      0 --:--:-- --:--:-- --:--:--   335
100 24090  100 24090    0     0  17421      0  0:00:01  0:00:01 --:--:-- 44446
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
gpg: 于 2017年09月11日 星期一 04时59分21秒 CST 创建的签名，使用 RSA，钥匙号 BF04FF17
gpg: 无法检查签名：没有公钥
Warning, RVM 1.26.0 introduces signed releases and automated check of signatures when GPG software found. Assuming you trust Michal Papis import the mpapis public key (downloading the signatures).

GPG signature verification failed for '/usr/local/rvm/archives/rvm-1.29.3.tgz' - 'https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc'! Try to install GPG v2 and then fetch the public key:

    gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -

the key can be compared with:

    https://rvm.io/mpapis.asc
    https://keybase.io/mpapis

NOTE: GPG version 2.1.17 have a bug which cause failures during fetching keys from remote server. Please downgrade or upgrade to newer version (if available) or use the second method described above.
```

执行报错中的`gpg2 --recv-keys `的命令。

例如：

```bash
[root@kube-node-1 ~]# gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
gpg: 钥匙环‘/root/.gnupg/secring.gpg’已建立
gpg: 下载密钥‘D39DC0E3’，从 hkp 服务器 keys.gnupg.net
gpg: /root/.gnupg/trustdb.gpg：建立了信任度数据库
gpg: 密钥 D39DC0E3：公钥“Michal Papis (RVM signing) <mpapis@gmail.com>”已导入
gpg: 没有找到任何绝对信任的密钥
gpg: 合计被处理的数量：1
gpg:           已导入：1  (RSA: 1)
```

再次执行命令`curl -L get.rvm.io | bash -s stable`。例如：

```bash
[root@kube-node-1 ~]# curl -L get.rvm.io | bash -s stable
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    310      0 --:--:-- --:--:-- --:--:--   309
100 24090  100 24090    0     0  18230      0  0:00:01  0:00:01 --:--:--  103k
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
gpg: 于 2017年09月11日 星期一 04时59分21秒 CST 创建的签名，使用 RSA，钥匙号 BF04FF17
gpg: 完好的签名，来自于“Michal Papis (RVM signing) <mpapis@gmail.com>”
gpg:               亦即“Michal Papis <michal.papis@toptal.com>”
gpg:               亦即“[jpeg image of size 5015]”
gpg: 警告：这把密钥未经受信任的签名认证！
gpg:       没有证据表明这个签名属于它所声称的持有者。
主钥指纹： 409B 6B17 96C2 7546 2A17  0311 3804 BB82 D39D C0E3
子钥指纹： 62C9 E5F4 DA30 0D94 AC36  166B E206 C29F BF04 FF17
GPG verified '/usr/local/rvm/archives/rvm-1.29.3.tgz'
Creating group 'rvm'

Installing RVM to /usr/local/rvm/
Installation of RVM in /usr/local/rvm/ is almost complete:

  * First you need to add all users that will be using rvm to 'rvm' group,
    and logout - login again, anyone using rvm will be operating with `umask u=rwx,g=rwx,o=rx`.

  * To start using RVM you need to run `source /etc/profile.d/rvm.sh`
    in all your open bash windows, in rare cases you need to reopen all bash windows.
```

以上表示执行成功，

```bash
source /usr/local/rvm/scripts/rvm
```

查看rvm库中已知的ruby版本

```bash
rvm list known
```

例如：

```bash
[root@kube-node-1 ~]# rvm list known
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.7]
[ruby-]2.3[.4]
[ruby-]2.4[.1]
ruby-head
...
```

**3、升级Ruby**

```bash
#安装ruby
rvm install  2.4.0
#使用新版本
rvm use  2.4.0
#移除旧版本
rvm remove 2.0.0
#查看当前版本
ruby --version
```

例如：

```bash
[root@kube-node-1 ~]# rvm install  2.4.0
Searching for binary rubies, this might take some time.
Found remote file https://rvm_io.global.ssl.fastly.net/binaries/centos/7/x86_64/ruby-2.4.0.tar.bz2
Checking requirements for centos.
Installing requirements for centos.
Installing required packages: autoconf, automake, bison, bzip2, gcc-c++, libffi-devel, libtool, readline-devel, sqlite-devel, zlib-devel, libyaml-devel, openssl-devel................................
Requirements installation successful.
ruby-2.4.0 - #configure
ruby-2.4.0 - #download
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 14.0M  100 14.0M    0     0   852k      0  0:00:16  0:00:16 --:--:--  980k
No checksum for downloaded archive, recording checksum in user configuration.
ruby-2.4.0 - #validate archive
ruby-2.4.0 - #extract
ruby-2.4.0 - #validate binary
ruby-2.4.0 - #setup
ruby-2.4.0 - #gemset created /usr/local/rvm/gems/ruby-2.4.0@global
ruby-2.4.0 - #importing gemset /usr/local/rvm/gemsets/global.gems..............................
ruby-2.4.0 - #generating global wrappers........
ruby-2.4.0 - #gemset created /usr/local/rvm/gems/ruby-2.4.0
ruby-2.4.0 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems evaluated to empty gem list
ruby-2.4.0 - #generating default wrappers........

[root@kube-node-1 ~]# rvm use  2.4.0
Using /usr/local/rvm/gems/ruby-2.4.0

[root@kube-node-1 ~]# rvm remove 2.0.0
ruby-2.0.0-p648 - #already gone
Using /usr/local/rvm/gems/ruby-2.4.0

[root@kube-node-1 ~]# ruby --version
ruby 2.4.0p0 (2016-12-24 revision 57164) [x86_64-linux]
```

**4、安装gem**

```bash
gem install redis
```

例如：

```bash
[root@kube-node-1 ~]# gem install redis

Fetching: redis-4.0.1.gem (100%)
Successfully installed redis-4.0.1
Parsing documentation for redis-4.0.1
Installing ri documentation for redis-4.0.1
Done installing documentation for redis after 2 seconds
1 gem installed
```

**5、执行redis-trib.rb命令**

以上表示安装成功，可以执行`redis-trib.rb`命令。

```bash
cd src 
#执行redis-trib.rb命令
./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 \
> 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```

参数`create`表示创建一个新的集群，`--replicas 1`表示为每个master创建一个slave。

如果创建成功会显示以下信息

```bash
[OK] All 16384 slots covered
```

例如：

```bash
[root@kube-node-1 src]# ./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 \
> 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
Adding replica 127.0.0.1:7004 to 127.0.0.1:7000
Adding replica 127.0.0.1:7005 to 127.0.0.1:7001
Adding replica 127.0.0.1:7003 to 127.0.0.1:7002
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: d5a834d075fd93eefab877c6ebb86efff680650f 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
M: 13d0c397604a0b2644244c37b666fce83f29faa8 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
M: be2718476eba4e56f696e56b75e67df720b7fc24 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
S: 3d02f59b34047486faecc023685379de7b38076c 127.0.0.1:7003
   replicates 13d0c397604a0b2644244c37b666fce83f29faa8
S: dedf672f0a75faf37407ac4edd5da23bc4651e25 127.0.0.1:7004
   replicates be2718476eba4e56f696e56b75e67df720b7fc24
S: 99c07119a449a703583019f7699e15afa0e41952 127.0.0.1:7005
   replicates d5a834d075fd93eefab877c6ebb86efff680650f
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join....
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: d5a834d075fd93eefab877c6ebb86efff680650f 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: be2718476eba4e56f696e56b75e67df720b7fc24 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: 13d0c397604a0b2644244c37b666fce83f29faa8 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 3d02f59b34047486faecc023685379de7b38076c 127.0.0.1:7003
   slots: (0 slots) slave
   replicates 13d0c397604a0b2644244c37b666fce83f29faa8
S: 99c07119a449a703583019f7699e15afa0e41952 127.0.0.1:7005
   slots: (0 slots) slave
   replicates d5a834d075fd93eefab877c6ebb86efff680650f
S: dedf672f0a75faf37407ac4edd5da23bc4651e25 127.0.0.1:7004
   slots: (0 slots) slave
   replicates be2718476eba4e56f696e56b75e67df720b7fc24
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### 2.1.3 部署结果验证

**1、客户端访问**

使用客户端`redis-cli `二进制访问某个实例，执行`set`和`get`的测试。

```bash
$ redis-cli -c -p 7000
redis 127.0.0.1:7000> set foo bar
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK
redis 127.0.0.1:7002> set hello world
-> Redirected to slot [866] located at 127.0.0.1:7000
OK
redis 127.0.0.1:7000> get foo
-> Redirected to slot [12182] located at 127.0.0.1:7002
"bar"
redis 127.0.0.1:7000> get hello
-> Redirected to slot [866] located at 127.0.0.1:7000
"world"
```

**2、查看集群状态**

使用`cluster info`命令查看集群状态。

```bash
127.0.0.1:7000> cluster info
cluster_state:ok                       #集群状态
cluster_slots_assigned:16384           #被分配的槽位数
cluster_slots_ok:16384                 #正确分配的槽位
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6                  #当前节点
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:48273
cluster_stats_messages_pong_sent:49884
cluster_stats_messages_sent:98157
cluster_stats_messages_ping_received:49879
cluster_stats_messages_pong_received:48273
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:98157
```

**3、查看节点状态**

使用`cluster nodes`命令查看节点状态。

```bash
127.0.0.1:7000> cluster nodes
be2718476eba4e56f696e56b75e67df720b7fc24 127.0.0.1:7002@17002 master - 0 1517303607000 3 connected 10923-16383
13d0c397604a0b2644244c37b666fce83f29faa8 127.0.0.1:7001@17001 master - 0 1517303606000 2 connected 5461-10922
3d02f59b34047486faecc023685379de7b38076c 127.0.0.1:7003@17003 slave 13d0c397604a0b2644244c37b666fce83f29faa8 0 1517303606030 4 connected
d5a834d075fd93eefab877c6ebb86efff680650f 127.0.0.1:7000@17000 myself,master - 0 1517303604000 1 connected 0-5460
99c07119a449a703583019f7699e15afa0e41952 127.0.0.1:7005@17005 slave d5a834d075fd93eefab877c6ebb86efff680650f 0 1517303607060 6 connected
dedf672f0a75faf37407ac4edd5da23bc4651e25 127.0.0.1:7004@17004 slave be2718476eba4e56f696e56b75e67df720b7fc24 0 1517303608082 5 connected
```



 参考文章：

https://redis.io/download

https://redis.io/topics/cluster-tutorial



