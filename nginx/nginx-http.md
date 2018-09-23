# 1. Nginx的系统架构

1. Nginx包含一个单一的master进程和多个worker进程，每个进程都是单进程，并且设计为同时处理成千上万个连接。
2. worker进程是处理连接的地方，Nginx使用了操作系统事件机制来快速响应这些请求。
3. master进程负责读取配置文件、处理套接字、派生worker进程、打开日志文件和编译嵌入式的perl脚本。master进程是一个可以通过处理信号量来管理请求的进程。
4. worker进程运行在一个忙碌的事件循环处理中，用于处理进入的连接。每一个nginx模块被构筑在worker中。任何请求处理、过滤、处理代理的连接和更多操作都在worker中完成。
5. 如果没有阻塞worker进程的进程（例如磁盘I/O），那么需要配置的worker进程要多于CPU内核数，以便处理负载。

# 2. Http核心模块

## 2.1.1. server

指令server开始一个新的上下文（context）。

http server指令

| 指令                      | 说明                                       |
| ----------------------- | ---------------------------------------- |
| port_in_redirect        | 确认nginx是否对端口指定重定向                        |
| server                  | 创建一个新的配置区域，定义一个虚拟主机。listen指令指定IP和端口；server_name列举用于匹配的Host头值 |
| server_name             | 配置用于响应请求的虚拟主机名称                          |
| server_name_in_redirect |                                          |
| server_tokens           | 在错误信息中禁止发送nginx的版本号和server响应头            |

## 2.1.2. 日志格式

| **参数**                  | **说明**                     | **示例**                                   |
| ----------------------- | -------------------------- | ---------------------------------------- |
| $remote_addr            | 客户端地址                      | 211.28.65.253                            |
| $remote_user            | 客户端用户名称                    | --                                       |
| $time_local             | 访问时间和时区                    | 18/Jul/2012:17:00:01 +0800               |
| $request                | 请求的URI和HTTP协议              | "GET /article-10000.html HTTP/1.1"       |
| $http_host              | 请求地址，即浏览器中你输入的地址（IP或域名）    | www.it300.com192.168.100.100             |
| $status                 | HTTP请求状态                   | 200                                      |
| $upstream_status        | upstream状态                 | 200                                      |
| $body_bytes_sent        | 发送给客户端文件内容大小               | 1547                                     |
| $http_referer           | url跳转来源                    | https://www.baidu.com/                   |
| $http_user_agent        | 用户终端浏览器等信息                 | "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; SV1; GTB7.0; .NET4.0C; |
| $ssl_protocol           | SSL协议版本                    | TLSv1                                    |
| $ssl_cipher             | 交换数据中的算法                   | RC4-SHA                                  |
| $upstream_addr          | 后台upstream的地址，即真正提供服务的主机地址 | 10.10.10.100:80                          |
| $request_time           | 整个请求的总时间                   | 0.205                                    |
| $upstream_response_time | 请求过程中，upstream响应时间         | 0.002                                    |

**日志切割**

```shell
# vim /etc/logrotate.d/nginx
/usr/local/nginx/logs/*.log{
        #指定转储周期为每天
        daily
        #保留30个备份
        rotate 30
        #需要压缩
        delaycompress
        #YYYYMMDD日期格式
        dateext
        #忽略错误
        missingok
        #如果日志为空则不做轮询
        notifempty
        #只为整个日志组运行一次的脚本
        sharedscripts
        #日志轮询后执行的脚本
        postrotate
                service nginx reload
        endscript
}
```
