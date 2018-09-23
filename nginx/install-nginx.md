# 1. 部署

## 1.1. 使用安装包的方式

rpm -ivh nginx-xxx.rpm

## 1.2. 使用源代码安装

### 1.2.1. 下载源码包

```shell
wget http://blob.wae.haplat.net/nginx/nginx-1.9.13.tar.gz
```

## 1.2.2. 创建临时目录并解压源码包

```shell
mkdir $HOME/build
cd $HOME/build && tar zxvf nginx-<version-number>.tar.gz
```

### 1.2.3. 编译并安装

```shell
 cd $HOME/build/nginx-<version-number>
 
./configure \
--prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
...
#<更多配置项见以下说明>
 
make && make install
```

### 1.2.4. 配置项

#### 1.2.4.1. 通用配置项

| 配置选项                    | 说明                                       |
| ----------------------- | ---------------------------------------- |
| --prefix=<path>         | nginx安装的根路径，所有其他的路径都要依赖与该选项              |
| --sbin-path=<path>      | nginx二进制文件的路径，如果没有指定则会依赖于--prefix        |
| --conf-path=<path>      | 如果在命令行中没有指定配置文件，则通过该配置项去查找配置文件           |
| --error-log-path=<path> | 指定错误文件的路径                                |
| --pid-path=<path>       | 指定的文件将会写入nginx master进程的pid，通常在/var/run下 |
| --lock-path=<path>      | 共享存储器互斥锁文件的路径                            |
| --user=<user>           | worker进程运行的用户                            |
| --group=<group>         | worker进程运行的组                             |
| --with-file-aio         | 启动异步I/O                                  |
| --with-debug            | 启用调试日志，生产环境不推荐配置                         |

#### 1.2.4.2. 优化配置项

| 配置选项                    | 说明                    |
| ----------------------- | --------------------- |
| --with-cc=<path>        | 如果想设置一个不在默认PATH下的C编译器 |
| --with-cpp=<path>       | 设置C预处理器的相应路径          |
| --with-cc-opt=<options> | 指定必要的include文件路径      |
| --with-ld-opt=<options> | 包含连接器库的路径和运行路径        |
| --with-cpu-opt=<cpu>    | 通过该选项为特定的CPU构建nginx   |

#### 1.2.4.3. http模块的配置项

| 配置选项                                | 说明                                       |
| ----------------------------------- | ---------------------------------------- |
| --without-http-cache                | 在使用upstream模块时，nginx能够配置本地缓存内容，该选项可以禁用缓存 |
| --with-http_perl_module             | nginx配置能够扩展使用perl代码。该项启用这个模块，但会降低性能      |
| --with-perl_modules_path=<path>     | 对于额外嵌入的perl模块，该选项指定该perl解析器的路径           |
| --with-perl=<path>                  | 如果在默认的路径中找不到perl则指定perl（5.6版本以上）的路径      |
| --http-log-path=<path>              | http访问日志的默认路径                            |
| --http-client-body-temp-path=<path> | 从客户端收到请求后，该项用于作为请求体临时存放的目录               |
| --http-proxy-temp-path=<path>       | 在使用代理后，通过该项设置存放临时文件路径                    |
| --http-fastcgi-temp-path=<path>     | 设置FastCGI临时文件的目录                         |
| --http-uwsgi-temp-path=<path>       | 设置uWSGI临时文件的目录                           |
| --http-scgi-temp-path=<path>        | 设置SCGI临时文件的目录                            |

#### 1.2.4.4. 其他模块额外配置项

默认没有安装这些模块，可以通过--with-<module-name>_module来启用相应的模块功能。

| 配置选项                            | 说明                                       |
| ------------------------------- | ---------------------------------------- |
| --with-http_ssl_module          | 如果需要对流量进行加密，可以使用该选项，再URLs中开始部分将会是https(需要OpenSSL库) |
| --with-http_realip_module       | 如果nginx在七层负载均衡器或者其他设备之后，它们将Http头中的客户端IP地址传递，则需要启用该模块，再多个客户处于一个IP地址的情况下使用 |
| --with-http_addition_module     | 该模块作为输出过滤器，使能够在请求经过一个location前或后时在该location本身添加内容 |
| --with-http_xslt_module         | 该模块用于处理XML响应转换，基于一个或多个XSLT格式             |
| --with-http_image_filter_module | 该模块被作为图像过滤器使用，在将图像投递到客户之前进行处理（需要libgd库）  |
| --with-http_geoip_module        | 使用该模块，能够设置各种变量以便在配置文件中的区段使用，基于地理位置查找客户端IP地址 |
| --with-http_sub_module          | 该模块实现替代过滤，在响应中用一个字符串替代另一个字符串             |
| --with-heep_dav_module          | 启用这个模块将激活使用WebDAV的配置指令。                  |
| --with-http_flv_module          | 如果需要提供Flash流媒体视频文件，那么该模块将会提供伪流媒体         |
| --with-http_mp4_module          | 这个模块支持H.264/AAC文件伪流媒体                    |
| --with-http_gzip_static_module  | 当被调用的资源没有.gz结尾格式的文件时，如果想支持发送预压缩版本的静态文件，那么使用该模块 |
| --with-http_gunzip_module       | 对于不支持gzip编码的客户，该模块用于为客户解压缩预压缩内容          |
| --with-http_random_index_module | 如果你想提供从一个目录中随机选择文件的索引文件，那么该模块需要激活        |
| --with-http_secure_link_module  | 该模块提供一种机制，它会将一个哈希值链接到一个URL中，因此只有那些使用正确密码能够计算链接 |
| --with-http_stub_status_module  | 启用这个模块后会收集Nginx自身的状态信息。输出的状态信息可以使用RRDtool或类似的东西绘制成图 |

# 2. 配置

配置文件一般为/etc/nginx/nginx.conf或/usr/local/nginx/conf/nginx.conf。

## 2.1. 基本配置格式

```shell
<section>{
    <directive> <parameters>;
}
```

每一个指令行由分号结束，大括号{}表示一个新的上下文。

## 2.2. Nginx全局配置参数

全局配置指令

| 模块                 | 配置项                                      | 说明                                       |
| ------------------ | ---------------------------------------- | ---------------------------------------- |
| main模块             | user                                     | 配置worker进程的用户和组，如果忽略group，则group等于指定的用户的所属组 |
| worker_processes   | 指定worker进程的启动数量，可将其设置为可用的CPU内核数，若为auto为自动检测 |                                          |
| error_log          | 所有错误的写入文件，第二个参数指定错误的级别（debug，info，notice，warn，error，crit，alert，emerg） |                                          |
| pid                | 设置主进程IP的文件                               |                                          |
| events模块           | use                                      | 用于设置使用什么样的连接方法                           |
| worker_connections | 用于配置一个工作进程能够接受的并发连接最大数。包括客户连接和向上游服务器的连接。 |                                          |

## 2.3. 使用include文件

include文件可以在任何地方以增强配置文件的可读性，使用include文件要确保被包含文件自身正确的nginx语法，即配置指令和块，然后指定这些文件的路径。

include /etc/nginx/mime.types;

若使用通配符则表示通配的多个文件，若没有给定全路径则依据主配置文件路径进行搜索。

include /etc/nginx/conf.d/*.conf

测试配置文件(包括include的配置文件)语法：

nginx -t -c {path-to-nginx.conf}

## 2.4. 配置说明

### 2.4.1. main模块

```shell
#main模块类似main函数包含其他子模块，非模块配置项(包括模块内)分号结尾，子模块配置花括号结尾
user nobady;   #一般按默认设置
pid /var/run/nginx.pid;   #进程标识符存放路径，一般按默认设置
worker_processes auto;   #nginx对外提供web服务时的worder进程数，可将其设置为可用的CPU内核数，auto为自动检测
worker_rlimit_nofile 100000;  # 更改worker进程的最大打开文件数限制
error_log logs/error.log  info;   #错误日志存放路径
keepalive_timeout 60;  #keepalive_timeout 60;
events{
  #见events模块
}
http{  #见http模块
  server{ 
    ...
    location /{
     
    }
  }
}
mail{
  #见mail模块
}
```

### 2.4.2. events模块

```shell
events {
  worker_connections 2048;    #设置可由一个worker进程同时打开的最大连接数
  multi_accept on;   #告诉nginx收到一个新连接通知后接受尽可能多的连接
  use epoll; #设置用于复用客户端线程的轮询方法。Linux 2.6+：使用epoll；*BSD：使用kqueue。
}
```

### 2.4.3. http模块

```shell
http {  #http模块
    server {  #server模块，http服务上的虚拟主机， server 当做对应一个域名进行的配置
        listen          80;  #配置监听端口
        server_name     www.linuxidc.com; #配置访问域名
        access_log      logs/linuxidc.access.log main;  #指定日志文件的存放路径
        index index.html;    #默认访问页面
        root  /var/www/androidj.com/htdocs;  # root 是指将本地的一个文件夹作为所有 url 请求的根路径
        upstream backend {   #反向代理的后端机器，实现负载均衡
            ip_hash;    #指明了我们均衡的方式是按照用户的 ip 地址进行分配
            server backend1.example.com;
            server backend2.example.com;
            server backend3.example.com;
            server backend4.example.com;
        }
        location / {  #location 是在一个域名下对更精细的路径进行配置
            proxy_pass http://backend;  #反向代理到后端机器
        }
    }
 
    server {
        listen          80;
        server_name     www.Androidj.com;
        access_log      logs/androidj.access.log main;
        location / {
            index index.html;
            root  /var/www/androidj.com/htdocs;
        }
    }
}
```

### 2.4.4. mail模块

```shell
mail {
    auth_http  127.0.0.1:80/auth.php;
    pop3_capabilities  "TOP"  "USER";
    imap_capabilities  "IMAP4rev1"  "UIDPLUS";
 
    server {
        listen     110;
        protocol   pop3;
        proxy      on;
    }
    server {
        listen      25;
        protocol    smtp;
        proxy       on;
        smtp_auth   login plain;
        xclient     off;
    }
}
```

