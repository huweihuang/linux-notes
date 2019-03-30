# 1. Memcached简介

- Memcached是一个开源的，高性能，分布式内存对象缓存系统。

- Memcached是一种基于内存的key-value存储，用来存储小块的任意数据（字符串、对象）。这些数据可以是数据库调用、API调用或者是页面渲染的结果。

- 一般的使用目的是，通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。

## 1.1. 特征

memcached作为高速运行的分布式缓存服务器，具有以下的特点。

- 协议简单
- 基于libevent的事件处理
- 内置内存存储方式
- memcached不互相通信的分布式

# 2. 安装与运行

## 2.1. 自动安装

```bash
# For Redhat/Fedora
yum install -y memcached

# For Debian or Ubuntu
apt-get install memcached
```

## 2.2. 源码安装

安装指定版本的Memcached可以从 https://github.com/memcached/memcached/wiki/ReleaseNotes 地址下载。

```bash
# Memcached depends on libevent
yum install libevent-devel

# install 
wget https://memcached.org/latest
[you might need to rename the file]
tar -zxf memcached-1.x.x.tar.gz
cd memcached-1.x.x
./configure --prefix=/usr/local/memcached
make && make test && sudo make install
```

**问题**

如遇以下报错，可再执行`make install`。

```bash
Signal handled: Interrupt.
ok 51 - shutdown
ok 52 - stop_server
/bin/sh:行3: prove: 未找到命令
make: *** [test] Error 127
```

## 2.3. 验证

确认是否安装成功，可执行以下命令

```bash
/usr/local/memcached/bin/memcached -h      
```

## 2.4. 运行

### 2.4.1. 前台运行

```bash
/usr/local/memcached/bin/memcached -p 11211 -m 64m -vv
```

### 2.4.2. 后台运行

```bash
/usr/local/memcached/bin/memcached -p 11211 -m 64m -d -c 102400 -t 8 -P /tmp/memcached.pid 
```

## 2.5. 连接

```bash
$ telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
set foo 0 0 3                                                   保存命令
bar                                                             数据
STORED                                                          结果
get foo                                                         取得命令
VALUE foo 0 3                                                   数据
bar                                                             数据
END                                                             结束行
quit                                                            退出
```

# 3. Memcached运行参数

```bash
# /usr/local/memcached/bin/memcached -h
memcached 1.5.12
-p, --port=<num>          TCP port to listen on (default: 11211)
-U, --udp-port=<num>      UDP port to listen on (default: 0, off)
-s, --unix-socket=<file>  UNIX socket to listen on (disables network support)
-A, --enable-shutdown     enable ascii "shutdown" command
-a, --unix-mask=<mask>    access mask for UNIX socket, in octal (default: 0700)
-l, --listen=<addr>       interface to listen on (default: INADDR_ANY)
-d, --daemon              run as a daemon
-r, --enable-coredumps    maximize core file limit
-u, --user=<user>         assume identity of <username> (only when run as root)
-m, --memory-limit=<num>  item memory in megabytes (default: 64 MB)
-M, --disable-evictions   return error on memory exhausted instead of evicting
-c, --conn-limit=<num>    max simultaneous connections (default: 1024)
-k, --lock-memory         lock down all paged memory
-v, --verbose             verbose (print errors/warnings while in event loop)
-vv                       very verbose (also print client commands/responses)
-vvv                      extremely verbose (internal state transitions)
-h, --help                print this help and exit
-i, --license             print memcached and libevent license
-V, --version             print version and exit
-P, --pidfile=<file>      save PID in <file>, only used with -d option
-f, --slab-growth-factor=<num> chunk size growth factor (default: 1.25)
-n, --slab-min-size=<bytes> min space used for key+value+flags (default: 48)
-L, --enable-largepages  try to use large memory pages (if available)
-D <char>     Use <char> as the delimiter between key prefixes and IDs.
              This is used for per-prefix stats reporting. The default is
              ":" (colon). If this option is specified, stats collection
              is turned on automatically; if not, then it may be turned on
              by sending the "stats detail on" command to the server.
-t, --threads=<num>       number of threads to use (default: 4)
-R, --max-reqs-per-event  maximum number of requests per event, limits the
                          requests processed per connection to prevent
                          starvation (default: 20)
-C, --disable-cas         disable use of CAS
-b, --listen-backlog=<num> set the backlog queue limit (default: 1024)
-B, --protocol=<name>     protocol - one of ascii, binary, or auto (default)
-I, --max-item-size=<num> adjusts max item size
                          (default: 1mb, min: 1k, max: 128m)
-F, --disable-flush-all   disable flush_all command
-X, --disable-dumping     disable stats cachedump and lru_crawler metadump
-o, --extended            comma separated list of extended options
                          most options have a 'no_' prefix to disable
   - maxconns_fast:       immediately close new connections after limit
   - hashpower:           an integer multiplier for how large the hash
                          table should be. normally grows at runtime.
                          set based on "STAT hash_power_level"
   - tail_repair_time:    time in seconds for how long to wait before
                          forcefully killing LRU tail item.
                          disabled by default; very dangerous option.
   - hash_algorithm:      the hash table algorithm
                          default is murmur3 hash. options: jenkins, murmur3
   - lru_crawler:         enable LRU Crawler background thread
   - lru_crawler_sleep:   microseconds to sleep between items
                          default is 100.
   - lru_crawler_tocrawl: max items to crawl per slab per run
                          default is 0 (unlimited)
   - lru_maintainer:      enable new LRU system + background thread
   - hot_lru_pct:         pct of slab memory to reserve for hot lru.
                          (requires lru_maintainer)
   - warm_lru_pct:        pct of slab memory to reserve for warm lru.
                          (requires lru_maintainer)
   - hot_max_factor:      items idle > cold lru age * drop from hot lru.
   - warm_max_factor:     items idle > cold lru age * this drop from warm.
   - temporary_ttl:       TTL's below get separate LRU, can't be evicted.
                          (requires lru_maintainer)
   - idle_timeout:        timeout for idle connections
   - slab_chunk_max:      (EXPERIMENTAL) maximum slab size. use extreme care.
   - watcher_logbuf_size: size in kilobytes of per-watcher write buffer.
   - worker_logbuf_size:  size in kilobytes of per-worker-thread buffer
                          read by background thread, then written to watchers.
   - track_sizes:         enable dynamic reports for 'stats sizes' command.
   - no_inline_ascii_resp: save up to 24 bytes per item.
                           small perf hit in ASCII, no perf difference in
                           binary protocol. speeds up all sets.
   - no_hashexpand:       disables hash table expansion (dangerous)
   - modern:              enables options which will be default in future.
             currently: nothing
   - no_modern:           uses defaults of previous major version (1.4.x)
```

常用参数：

- -d是启动一个守护进程；
- -m是分配给Memcache使用的内存数量，单位是MB；
- -u是运行Memcache的用户；
- -l是监听的服务器IP地址，可以有多个地址；
- -p是设置Memcache监听的端口，，最好是1024以上的端口；
- -c是最大运行的并发连接数，默认是1024；
- -t是线程数，默认为4；
- -P是设置保存Memcache的pid文件。



参考文章：

- https://github.com/memcached/memcached/wiki/Overview
- https://github.com/memcached/memcached/wiki/Install
- http://www.runoob.com/memcached/memcached-tutorial.html
