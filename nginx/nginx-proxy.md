# 1. 反向代理简介

Nginx可以作为反向代理，接收客户端的请求，并向上游服务器发起新的请求。该请求可以根据客户端请求的URI，客户机参数或其他逻辑进行拆分，原始URL中的任何部分可以以这种方式进行转换。

## 1.1. 代理模块指令

| 指令                             | 说明                                       |
| ------------------------------ | ---------------------------------------- |
| proxy_connect_timeout          | Nginx从接受到请求到连接至上游服务器的最长等待时间              |
| proxy_cookie_domain            | 替代从上游服务器来的Set-Cookie头的域domain            |
| proxy_cookie_path              | 替代从上游服务器来的Set-Cookie头的path属性             |
| proxy_headers_hash_bucket_size | 头名字的最大值                                  |
| proxy_headers_hash_max_size    | 从上游服务器接收到头的总大小                           |
| proxy_hide_header              | 不应该传递给客户端头的列表                            |
| proxy_http_version             | 用于通上游服务器通信的Http协议版本                      |
| proxy_ignore_client_abort      | 如果设置为ON，那么客户端放弃连接后，nginx将不会放弃同上游服务器的连接   |
| proxy_ignore_headers           | 当处理来自上游服务器的响应时，设置哪些头可以被忽略                |
| proxy_intercept_errors         | 如果启用该选项，Nginx将会显示配置的error_page错误，而不是来自于上游服务器的直接响应 |
| proxy_max_temp_file_size       | 在写入内存缓冲区时响应与内存不匹配时使用时，给出溢出文件的最大值         |
| proxy_pass                     | 指定请求被传递到的上游服务器，格式为URL                    |
| proxy_pass_header              | 覆盖掉在proxy_hide_header指令中设置的头，允许这些头传递到客户端 |
| proxy_pass_request_body        | 如果设置为off，将会阻止请求体传递到客户端                   |
| proxy_pass_request_headers     | 如果设置为on,则阻止请求头发送到上游服务器                   |
| proxy_read_timeout             | 给出连接关闭前从上游服务器两次成功的读操作耗时，如果上游服务器处理请求比较慢，那么该值需设置较高些 |
| proxy_redirect                 | 重写来自于上游服务器的Location和Refresh头             |
| proxy_send_timeout             | 给出连接关闭前从上游服务器两次成功的写操作耗时，如果上游服务器处理请求比较慢，那么该值需设置较高些 |
| proxy_set_body                 | 发送到上游服务器的请求体可能会被该指令的设置值修改                |
| proxy_set_header               | 重写发送到上游服务器头的内容，也可以通过将某种头的值设置为空字符，而不是发送某种头的方法实现 |
| proxy_temp_file_write_size     | 在同一时间内限制缓冲到一个临时文件的数据量，以使得Nginx不会过长地阻止单个请求 |
| proxy_temp_path                | 设定临时文件的缓冲，用于缓冲从上游服务器来的文件，可以设定目录的层次       |

## 1.2. upstream模块

upstream指令将会启用一个新的配置区域，在该区域定义了一组上游服务器，这些服务器可以被设置为不同的权重（权重高的服务器将会被Nginx传递越多的连接）。

| 指令         | 说明                                       |
| ---------- | ---------------------------------------- |
| ip_hash    | 通过IP地址的哈希值确保客户端均匀地连接所有的服务器，键值基于C类地址      |
| keepalive  | 每一个worker进程缓存的到上游服务器的连接数。再使用Http连接时，proxy_http_verison设置1.1，并将proxy_set_header设置为Connection "" |
| least_conn | 激活负载均衡算法，将请求发送到活跃连接数最少的那台服务器             |
| server     | 为upstream定义一个服务器地址（带有端口号的域名、IP地址，或者是UNIX套接字）和一个可选的参数，参数如下：weight：设置一个服务器的优先级优于其他服务器。max_fails：设置在fail_timeout时间之内尝试对一个服务器连接的最大次数，如果超过这个次数，那么就会被标记为down。fail_timeout：在这个指定的时间内服务器必须提供响应，如果在这个时间内没有收到响应，那么服务器就会被标记为down状态。backup：一旦其他服务器宕机，那么有该标记的机器就会接收请求。down：标记为一个服务器不再接受任何请求。 |

### 1.2.1. 负载均衡算法

upstream模块能够使用轮询、IP hash和最少连接数三种负载均衡算法之一来选择哪个上游服务器将会被在下一步中连接。

#### 1.2.1.1. 轮询

默认情况使用轮询，不需要配置指令来设置，该算法选择下一个服务器，基于先前选择，再配置文件中哪一个是下一个服务器，以及每个服务器的负载。轮询算法是基于在队列中谁是下一个的原理确保将访问量均匀的分配给每一个上游服务器。

#### 1.2.1.2. IP 哈希

通过ip_hash指令激活使用，从而将某些IP地址映射到同一个上游服务器。

#### 1.2.1.3. 最少连接数

通过least_conn指令启用，该算法通过选择一个活跃的最少连接数服务器，然后将负载均匀分配给上游服务器。如果上游服务器的处理器能力不同，那么可以为server指令使用weight来指示说明。该算法将考虑到不同服务器的加权最小连接数。

# 2. Upstream服务器类型

上游服务器是Ngixn代理连接的一个服务器，可以是物理机或虚拟机。

## 2.1. 单个upstream服务器

指令try_files(包括http core模块内)意味着按顺序尝试，直到找到一个匹配为止。Nginx将会投递与客户端给定URI匹配的任何文件，如果没有找到任何配置文件，将会把请求代理到Apache作进一步处理。

## 2.2. 多个upstream服务器

Nginx将会通过轮询的方式将连续请求传递给3个上游服务器。这样应用程序不会过载。

![这里写图片描述](https://res.cloudinary.com/dqxtn0ick/image/upload/v1537696859/article/nginx/nginx.png)

# 3. 负载均衡特别说明

1. 如果客户端希望总是访问同一个上游服务器，可以使用ip_hash指令；
2. 如果请求响应时间长短不一，可以使用least_conn指令；
3. 默认为轮询。
