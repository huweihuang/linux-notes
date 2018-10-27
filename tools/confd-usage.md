> confd的源码参考：https://github.com/kelseyhightower/confd

# 1. confd的部署

以下Linux系统为例。

下载`confd`的二进制文件，下载地址为：https://github.com/kelseyhightower/confd/releases。例如：

```bash
# Download the binary
wget https://github.com/kelseyhightower/confd/releases/download/v0.16.0/confd-0.16.0-linux-amd64

# 重命名二进制文件，并移动到PATH的目录下
mv confd-0.16.0-linux-amd64 /usr/local/bin/confd
chmod +x /usr/local/bin/confd

# 验证是否安装成功
confd --help
```

# 2. confd的配置

`Confd`通过读取后端存储的配置信息来动态更新对应的配置文件，对应的后端存储可以是`etcd`，`redis`等，其中etcd的v3版本对应的存储后端为`etcdv3`。

## 2.1. confd.toml

`confd.toml`为confd服务本身的配置文件，主要记录了使用的存储后端、协议、confdir等参数。

**示例：**

- 存储后端`etcdv3`：

```
backend = "etcdv3"
confdir = "/etc/confd"
log-level = "debug"
interval = 5
nodes = [
  "http://192.168.10.4:12379",
]
scheme = "http"
watch = true
```

其中`watch`参数表示实时监听后端存储的变化，如有变化则更新confd管理的配置。

- 存储后端为`redis`

```
backend = "redis"
confdir = "/etc/confd"
log-level = "debug"
interval = 1  # 间隔 1 秒同步一次配置文件
nodes = [
  "127.0.0.1:6379",
]
scheme = "http"
client_key = "123456"  # redis的密码，不是 password 参数
#watch = true
```

如果没有启动`watch`参数，则会依据`interval`参数定期去redis存储后端拿取数据，并比较与当前配置数据是否有变化（主要比较`md5`值），如果有变化则更新配置，没有变化则定期再去拿取数据，以此循环。

如果启动了`watch`参数，则修改redis存储数据的同时，还要执行`publish`的操作，促使`confd`去触发比较配置并更新配置的操作。

publish的命令格式如下:
```bash
publish __keyspace@0__:{prefix}/{key} set(or del)
```

## 2.2. 创建confdir

confdir底下包含两个目录:

- `conf.d`:confd的配置文件，主要包含配置的生成逻辑，例如模板源，后端存储对应的keys，命令执行等。
- `templates`:配置模板Template，即基于不同组件的配置，修改为符合 [Golang text templates](http://golang.org/pkg/text/template/#pkg-overview)的模板文件。

```bash
sudo mkdir -p /etc/confd/{conf.d,templates}
```

### 2.2.1. Template Resources

模板源配置文件是`TOML`格式的文件，主要包含配置的生成逻辑，例如模板源，后端存储对应的keys，命令执行等。默认目录在`/etc/confd/conf.d`。

参数说明：

**必要参数**

- `dest` (string) - The target file.
- `keys` (array of strings) - An array of keys.
- `src` (string) - The relative path of a [configuration template](https://github.com/kelseyhightower/confd/blob/master/docs/templates.md).

**可选参数**

- `gid` (int) - The gid that should own the file. Defaults to the effective gid.
- `mode` (string) - The permission mode of the file.
- `uid` (int) - The uid that should own the file. Defaults to the effective uid.
- `reload_cmd` (string) - The command to reload config.
- `check_cmd` (string) - The command to check config. Use `{{src}}` to reference the rendered source template.
- `prefix` (string) - The string to prefix to keys.

**例子**

例如：`/etc/confd/conf.d/myapp-nginx.toml`

```
[template]
prefix = "/myapp"
src = "nginx.tmpl"
dest = "/tmp/myapp.conf"
owner = "nginx"
mode = "0644"
keys = [
  "/services/web"
]
check_cmd = "/usr/sbin/nginx -t -c {{.src}}"
reload_cmd = "/usr/sbin/service nginx reload"
```

### 2.2.2. Template

`Template`定义了单一应用配置的模板，默认存储在`/etc/confd/templates`目录下，模板文件符合Go的[`text/template`](http://golang.org/pkg/text/template/)格式。

模板文件常用函数有`base`，`get`，`gets`，`lsdir`，`json`等。具体可参考https://github.com/kelseyhightower/confd/blob/master/docs/templates.md。

例子：

`/etc/confd/templates/nginx.tmpl`

```
{{range $dir := lsdir "/services/web"}}
upstream {{base $dir}} {
    {{$custdir := printf "/services/web/%s/*" $dir}}{{range gets $custdir}}
    server {{$data := json .Value}}{{$data.IP}}:80;
    {{end}}
}

server {
    server_name {{base $dir}}.example.com;
    location / {
        proxy_pass {{base $dir}};
    }
}
{{end}}
```

# 3. 创建后端存储的配置数据

以`etcdv3`存储为例，在etcd中创建以下数据。

```bash
etcdctl --endpoints=$endpoints put /services/web/cust1/2 '{"IP": "10.0.0.2"}'
etcdctl --endpoints=$endpoints put /services/web/cust2/2 '{"IP": "10.0.0.4"}'
etcdctl --endpoints=$endpoints put /services/web/cust2/1 '{"IP": "10.0.0.3"}'
etcdctl --endpoints=$endpoints put /services/web/cust1/1 '{"IP": "10.0.0.1"}'
```

# 4. 启动confd的服务

confd支持以`daemon`或者`onetime`两种模式运行，当以`daemon`模式运行时，confd会监听后端存储的配置变化，并根据配置模板动态生成目标配置文件。

confd可以使用`-config-file`参数来指定confd的配置文件，而将其他参数写在配置文件中。

```bash
/usr/local/bin/confd -config-file /etc/confd/conf/confd.toml
```

如果以`daemon`模式运行，在命令后面添加`&`符号，例如：

```bash
confd -watch -backend etcdv3 -node http://172.16.5.4:12379 &
```

以下以`onetime`模式运行为例。其中对应的后端存储类型是`etcdv3`。

```bash
# 执行命令
confd -onetime -backend etcdv3 -node http://172.16.5.4:12379

# output
2018-05-11T18:04:59+08:00 k8s-dbg-master-1 confd[35808]: INFO Backend set to etcdv3
2018-05-11T18:04:59+08:00 k8s-dbg-master-1 confd[35808]: INFO Starting confd
2018-05-11T18:04:59+08:00 k8s-dbg-master-1 confd[35808]: INFO Backend source(s) set to http://172.16.5.4:12379
2018-05-11T18:04:59+08:00 k8s-dbg-master-1 confd[35808]: INFO /root/myapp/twemproxy/conf/twemproxy.conf has md5sum 6f0f43abede612c75cb840a4840fbea3 should be 32f48664266e3fd6b56ee73a314ee272
2018-05-11T18:04:59+08:00 k8s-dbg-master-1 confd[35808]: INFO Target config /root/myapp/twemproxy/conf/twemproxy.conf out of sync
2018-05-11T18:04:59+08:00 k8s-dbg-master-1 confd[35808]: INFO Target config /root/myapp/twemproxy/conf/twemproxy.conf has been updated
```

# 5. 查看生成的配置文件

在`/etc/confd/conf.d/myapp-nginx.toml`中定义的配置文件的生成路径为`/tmp/myapp.conf`。

```bash
[root@k8s-dbg-master-1 dest]# cat myapp.conf
upstream cust1 {
    server 10.0.0.1:80;
    server 10.0.0.2:80;
}

server {
    server_name cust1.example.com;
    location / {
        proxy_pass cust1;
    }
}

upstream cust2 {
    server 10.0.0.3:80;
    server 10.0.0.4:80;
}

server {
    server_name cust2.example.com;
    location / {
        proxy_pass cust2;
    }
}
```

# 6. confd动态更新twemproxy

## 6.1. twemproxy.toml

confd的模板源文件配置：/etc/confd/conf.d/twemproxy.toml

```
[template]
src = "twemproxy.tmpl"
dest = "/root/myapp/twemproxy/conf/twemproxy.conf"
keys = [
  "/twemproxy/pool"
]
check_cmd = "/usr/local/bin/nutcracker -t -c /root/myapp/twemproxy/conf/twemproxy.conf"
reload_cmd = "bash /root/myapp/twemproxy/reload.sh"
```

## 6.2. twemproxy.tmpl

模板文件：/etc/confd/templates/twemproxy.tmpl

```yaml
global:
  worker_processes: 4         # 并发进程数, 如果为0, 这 fallback 回原来的单进程模型(不支持 config reload!)
  user: nobody                # worker 进程的用户, 默认 nobody. 只要主进程是 root 用户启动才生效.
  group: nobody               # worker 进程的用户组
  worker_shutdown_timeout: 30 # 单位为秒. 用于 reload 过程中在改时间段之后强制退出旧的 worker 进程.

pools: {{range gets "/twemproxy/pool/*"}}
  {{base .Key}}: {{$pool := json .Value}}
    listen: {{$pool.ListenAddr.IP}}:{{$pool.ListenAddr.Port}}
    hash: fnv1a_64 # 选择实例的 hash 规则
    distribution: ketama
    auto_eject_hosts: true # server 有问题是否剔除
    redis: true # 是否为 Redis 协议
    {{if $pool.Password}}redis_auth: {{$pool.Password}}{{end}}
    server_retry_timeout: 5000 # 被剔除多长时间后会重试
    server_connections: 25 # NOTE: server 连接池的大小, 默认为 1, 建议调整
    server_failure_limit: 5 # 失败多少次后暂时剔除
    timeout: 1000 # Server 超时时间, 1 sec
    backlog: 1024 # 连接队列大小
    preconnect: true # 预连接大小
    servers:{{range $server := $pool.Servers}}
     - {{$server.IP}}:{{$server.Port}}:1 {{if $server.Master}}master{{end}}
    {{end}}
{{end}}
```

## 6.3. etcd中的配置格式

`etcd`中的配置通过一个map来定义为完整的配置内容。其中`key`是`twemproxy`中`pool`的名称，`value`是`pool`的所有内容。

配置对应go结构体如下：

```go
type Pool struct{
    ListenAddr  ListenAddr `json:"ListenAddr,omitempty"`
    Servers []Server `json:"Servers,omitempty"`
    Password string `json:"Password,omitempty"`
}

type ListenAddr struct {
    IP string `json:"IP,omitempty"`
    Port string `json:"Port,omitempty"`
}

type Server struct {
    IP string `json:"IP,omitempty"`
    Port string `json:"Port,omitempty"`
    Master bool `json:"Master,omitempty"`
}
```

配置对应`JSON`格式如下：

```Json
{
    "ListenAddr": {
        "IP": "192.168.5.7",
        "Port": "22225"
    },
    "Servers": [
        {
            "IP": "10.233.116.168",
            "Port": "6379",
            "Master": true
        },
        {
            "IP": "10.233.110.207",
            "Port": "6379",
            "Master": false
        }
    ],
    "Password": "987654"
}
```

## 6.4. 生成`twemproxy`配置文件

```bash
global:
  worker_processes: 4         # 并发进程数, 如果为0, 这 fallback 回原来的单进程模型(不支持 config reload!)
  user: nobody                # worker 进程的用户, 默认 nobody. 只要主进程是 root 用户启动才生效.
  group: nobody               # worker 进程的用户组
  worker_shutdown_timeout: 30 # 单位为秒. 用于 reload 过程中在改时间段之后强制退出旧的 worker 进程.

pools:
  redis1:
    listen: 192.168.5.7:22223
    hash: fnv1a_64 # 选择实例的 hash 规则
    distribution: ketama
    auto_eject_hosts: true # server 有问题是否剔除
    redis: true # 是否为 Redis 协议
    redis_auth: 987654
    server_retry_timeout: 5000 # 被剔除多长时间后会重试
    server_connections: 25 # NOTE: server 连接池的大小, 默认为 1, 建议调整
    server_failure_limit: 5 # 失败多少次后暂时剔除
    timeout: 1000 # Server 超时时间, 1 sec
    backlog: 1024 # 连接队列大小
    preconnect: true # 预连接大小
    servers:
     - 10.233.116.169:6379:1


  redis2:
    listen: 192.168.5.7:22224
    hash: fnv1a_64 # 选择实例的 hash 规则
    distribution: ketama
    auto_eject_hosts: true # server 有问题是否剔除
    redis: true # 是否为 Redis 协议
    redis_auth: 987654
    server_retry_timeout: 5000 # 被剔除多长时间后会重试
    server_connections: 25 # NOTE: server 连接池的大小, 默认为 1, 建议调整
    server_failure_limit: 5 # 失败多少次后暂时剔除
    timeout: 1000 # Server 超时时间, 1 sec
    backlog: 1024 # 连接队列大小
    preconnect: true # 预连接大小
    servers:
     - 10.233.110.223:6379:1 master

     - 10.233.111.21:6379:1
```

# 7. confd的命令

```bash
$ confd --help
Usage of confd:
  -app-id string
    	Vault app-id to use with the app-id backend (only used with -backend=vault and auth-type=app-id)
  -auth-token string
    	Auth bearer token to use
  -auth-type string
    	Vault auth backend type to use (only used with -backend=vault)
  -backend string
    	backend to use (default "etcd")
  -basic-auth
    	Use Basic Auth to authenticate (only used with -backend=consul and -backend=etcd)
  -client-ca-keys string
    	client ca keys
  -client-cert string
    	the client cert
  -client-key string
    	the client key
  -confdir string
    	confd conf directory (default "/etc/confd")
  -config-file string
    	the confd config file (default "/etc/confd/confd.toml")
  -file value
    	the YAML file to watch for changes (only used with -backend=file)
  -filter string
    	files filter (only used with -backend=file) (default "*")
  -interval int
    	backend polling interval (default 600)
  -keep-stage-file
    	keep staged files
  -log-level string
    	level which confd should log messages
  -node value
    	list of backend nodes
  -noop
    	only show pending changes
  -onetime
    	run once and exit
  -password string
    	the password to authenticate with (only used with vault and etcd backends)
  -path string
    	Vault mount path of the auth method (only used with -backend=vault)
  -prefix string
    	key path prefix
  -role-id string
    	Vault role-id to use with the AppRole, Kubernetes backends (only used with -backend=vault and either auth-type=app-role or auth-type=kubernetes)
  -scheme string
    	the backend URI scheme for nodes retrieved from DNS SRV records (http or https) (default "http")
  -secret-id string
    	Vault secret-id to use with the AppRole backend (only used with -backend=vault and auth-type=app-role)
  -secret-keyring string
    	path to armored PGP secret keyring (for use with crypt functions)
  -separator string
    	the separator to replace '/' with when looking up keys in the backend, prefixed '/' will also be removed (only used with -backend=redis)
  -srv-domain string
    	the name of the resource record
  -srv-record string
    	the SRV record to search for backends nodes. Example: _etcd-client._tcp.example.com
  -sync-only
    	sync without check_cmd and reload_cmd
  -table string
    	the name of the DynamoDB table (only used with -backend=dynamodb)
  -user-id string
    	Vault user-id to use with the app-id backend (only used with -backend=value and auth-type=app-id)
  -username string
    	the username to authenticate as (only used with vault and etcd backends)
  -version
    	print version and exit
  -watch
    	enable watch support
```





参考文章：

https://github.com/kelseyhightower/confd/blob/master/docs/installation.md

https://github.com/kelseyhightower/confd/blob/master/docs/quick-start-guide.md

https://github.com/kelseyhightower/confd/blob/master/docs/template-resources.md

https://github.com/kelseyhightower/confd/blob/master/docs/templates.md
