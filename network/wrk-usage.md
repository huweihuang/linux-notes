---
title: "wrk的使用"
weight: 4
catalog: true
date: 2023-04-13 10:50:57
---

# 1. Installation

```bash
# mac
brew install wrk
```

# 2. Usage

```bash
$ wrk --help
Usage: wrk <options> <url>
  Options:
    -c, --connections <N>  Connections to keep open # 跟服务器建立并保持的TCP连接数量  
    -d, --duration    <T>  Duration of test # 压测时间    
    -t, --threads     <N>  Number of threads to use # 使用多少个线程进行压测 

    -s, --script      <S>  Load Lua script file # 指定Lua脚本路径 
    -H, --header      <H>  Add header to request  # 为每一个HTTP请求添加HTTP头  
        --latency          Print latency statistics # 在压测结束后，打印延迟统计信息 
        --timeout     <T>  Socket/request timeout  # 超时时间  
    -v, --version          Print version details # 打印正在使用的wrk的详细版本信息

  Numeric arguments may include a SI unit (1k, 1M, 1G) # 代表数字参数，支持国际单位 (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h) # 代表时间参数，支持时间单位 (2s, 2m, 2h)
```

参数设置说明：

一般设置线程数t，并发数c，压测时间d，--latency四个通用的参数。

- 线程数：一般设置为压测机器CPU核数的2-4倍，过大会导致线程切换频繁，效果下降。

- 并发数：根据压测结果，动态调整并发数使得压测达到瓶颈。

示例：

```bash
wrk -t12 -c400 -d30s --latency http://www.baidu.com
```

# 3. 压测结果

```bash
# wrk -t12 -c400 -d30s --latency http://www.baidu.com
Running 30s test @ http://www.baidu.com
  12 threads and 400 connections
               （平均值） （标准差）  （最大值）（正负一个标准差所占比例）
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    （延迟）
    Latency   568.16ms  250.70ms   2.00s    83.49%
   (每秒请求数)
    Req/Sec    28.26     14.99    90.00     65.71%
  Latency Distribution （延迟分布）
     50%  530.91ms
     75%  585.73ms
     90%  691.64ms
     99%    1.78s
  9842 requests in 30.10s, 99.03MB read  (30.10s内处理了9842个请求，耗费流量99.03MB)
  Socket errors: connect 158, read 0, write 0, timeout 580  (发生错误数)
Requests/sec:    327.00  (QPS ,即平均每秒处理请求数)
Transfer/sec:      3.29MB  (平均每秒流量)
```

# 4. Lua脚本定制压测参数

例如：压测post请求，需要设置指定参数。

写入以下lua脚本，login.lua

```lua
wrk.method = "POST"
wrk.body = '{"username":"xxx","password":"xxx"}'
wrk.headers["Content-Type"] = "application/x-www-form-urlencoded"
```

发起压测请求：

```bash
wrk -t12 -c1000 -d30s --latency  http://127.0.0.1:8081/login -s login.lua
```





参考：

- [GitHub - wg/wrk: Modern HTTP benchmarking tool](https://github.com/wg/wrk)

- https://www.cnblogs.com/quanxiaoha/p/10661650.html


