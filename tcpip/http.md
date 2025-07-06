---
title: "HTTP协议"
weight: 5
catalog: true
date: 2018-09-20 10:50:57
subtitle:
header-img:
tags:
- TCPIP
catagories:
- TCPIP
---

# 1. web及网络基础

## 1.1. 通过HTTP访问web[C/S]

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579775/article/tcpip/http/basis/1.1.png)

## 1.2. TCP/IP四层模型

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579776/article/tcpip/http/basis/1.2.png)

### 1.2.1. 数据包的封装

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579776/article/tcpip/http/basis/1.2.1.png)

## 1.3. TCP/IP协议族

### 1.3.1. 负责传输的IP协议

使用ARP协议凭借MAC地址通信

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579776/article/tcpip/http/basis/1.3.1.png)

### 1.3.2. 确保可靠的TCP协议

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579775/article/tcpip/http/basis/1.3.2.png)

### 1.3.3. 负责域名解析的DNS服务

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579776/article/tcpip/http/basis/1.3.3.png)

### 1.3.4. 各协议与HTTP的关系

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579775/article/tcpip/http/basis/1.3.4.png)

## 1.4. URI与URL

- URI(Uniform Resource Identifier):统一资源标识符
- URL(Uniform Resource Locator):统一资源定位符；URL是URI的子集

### 1.4.1. URI的格式

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579775/article/tcpip/http/basis/1.4.1.png)

| 字段             | 说明                                         |
| ---------------- | -------------------------------------------- |
| 协议             | http/https                                   |
| 登录信息（认证） | user:pass@(一般没有)                         |
| 服务器地址       | 域名或IP                                     |
| 服务器端口号     | 服务端口号，省略则取默认端口号               |
| 带层次的文件路径 | 指定服务器上的文件路径来定位特指的资源       |
| 查询字符串       | 使用查询字符串传入参数                       |
| 片段标识符       | 标记以获取资源中的子资源（文档内的某个位置） |

### 1.4.2. URI的示例

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579775/article/tcpip/http/basis/1.4.2.png)

# 2. HTTP协议

## 2.1. 通过请求和响应的交换达成通信

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579776/article/tcpip/http/basis/2.1.png)

### 2.1.1. 请求报文

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579775/article/tcpip/http/basis/2.1.1.png)

### 2.1.2. 响应报文

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579776/article/tcpip/http/basis/2.1.2.png)

## 2.2. HTTP请求方法

### 2.2.1. GET:获取资源

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579776/article/tcpip/http/basis/2.2.1.png)

### 2.2.2. POST:传输实体主体

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579777/article/tcpip/http/basis/2.2.2.1.png)

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579776/article/tcpip/http/basis/2.2.2.2.png)

### 2.2.3. PUT:传输文件

PUT方法用来传输文件，像FTP协议一样，要求在请求报文的主体中包含文件内容，然后保存到请求URI指定的位置。

因为自身不带验证机制，有安全问题，因此一般不采用。若配合验证机制或者REST标准则可使用。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579777/article/tcpip/http/basis/2.2.3.png)

### 2.2.4. HEAD:获取报文头部

HEAD和GET一样但不返回报文主体部分，用于确认URI的有效性及资源的更新时间等。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579777/article/tcpip/http/basis/2.2.4.png)

### 2.2.5. DELETE:删除文件

DELETE与PUT作用相反，但不带安全验证机制一般不采用。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579777/article/tcpip/http/basis/2.2.5.1.png)

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579777/article/tcpip/http/basis/2.2.5.2.png)

### 2.2.6. OPTIONS:询问支持的方法

OPTIONS用来查询针对请求URI指定的资源支持的方法

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579777/article/tcpip/http/basis/2.2.6.png)

### 2.2.7. TRACE:追踪路径

TRACE用来查询发送出去的请求是怎样被加工修改/篡改的，因为易引发XST（跨站追踪）攻击，一般不使用。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579777/article/tcpip/http/basis/2.2.7.png)

### 2.2.8. CONNECT:要求用隧道协议连接代理

CONNECT要求在与代理服务器通信时建立隧道，实现用隧道协议进行TCP通信。主要使用SSL（Source Sockets Layer:安全套接字）和TLS（Transport Layer Security:传输层安全）协议把通信内容加密后经网络隧道传输。

方法格式如下：

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579777/article/tcpip/http/basis/2.2.8.1.png)

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579778/article/tcpip/http/basis/2.2.8.2.png)

## 2.3. 持久连接

### 2.3.1. keep-alive

为解决每进行一次HTTP通信就要断开一次TCP连接，增加了通信量的开销，HTTP/1.1通过keep-alive持久连接，只要任意一端没有明确提出断开连接，则保持TCP连接状态。

持久连接减少了TCP连接的重复建立和断开所造成的额外开销，减轻了服务器端的负载。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579778/article/tcpip/http/basis/2.3.1.png)

### 2.3.2. 管线化

持续连接使得多数请求以管线化（pipelining）方式发送成为可能。管线化即同时并行发送多个请求，而不需要一个接一个等待响应。管线化技术比持续连接速度快，请求数越多越明显。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579778/article/tcpip/http/basis/2.3.2.png)

### 2.3.3. 使用cookie的状态管理

HTTP是无状态协议，不对之前发生过的请求和响应的状态进行管理，即无法根据之前的状态进行本次的请求处理。无状态协议的优点在于不必保存状态，减少服务器CPU及内存资源的消耗。

cookie技术通过在请求和响应报文中写入cookie信息来控制客户端的状态。cookie会根据从服务端发送的响应报文内的一个叫做Set-Cookie的首部字段通知客户端保存Cookie；当客户端再往服务端发送请求时，客户端自动在请求报文中加入Cookie值后发送出去。服务器发现Cookie后会检查从哪个客户端发送来的连接请求，对比服务器上的记录，最后得到之前的状态信息。

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579778/article/tcpip/http/basis/2.3.3.png)

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579778/article/tcpip/http/basis/2.3.3.2.png)

![](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579778/article/tcpip/http/basis/2.3.3.3.png)


# 3. HTTP报文

## 3.1. HTTP报文

用于HTTP协议交互的信息被称为HTTP报文，客户端的HTTP报文叫做请求报文，服务端的叫做响应报文。报文大致分为报文首部和报文主体，但并不一定要有报文主体。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580019/article/tcpip/http/codestatus/3.1.png)

## 3.2. 报文结构

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580019/article/tcpip/http/codestatus/3.2.png)

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580020/article/tcpip/http/codestatus/3.2.2.png)

| 字段     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 请求行   | 请求方法，请求URI和HTTP版本                                  |
| 状态行   | 响应结果的状态码，原因短语和HTTP版本                         |
| 首部字段 | 请求和响应的各种条件和属性的各类首部：通用首部、请求首部、响应首部、实体首部 |
| 其他     | HTTP的RFC里未定义的首部（Cookie等）                          |

## 3.3. 编码提升传输速率

HTTP在传输数据时可以按照数据原貌直接传输也可以在传输过程中编码提升传输速率；通过编码可以处理大量请求但会消耗更多的CPU等资源。

### 3.3.1. 报文主体和实体主体的差异

- 报文：是HTTP通信中的基本单位，由8位组字节流组成，通过HTTP通信传输。
- 实体：作为请求或响应的有效载荷数据被传输，其内容由实体首部和实体主体组成。

通常报文主体等于实体主体，但当传输中进行编码时，实体主体的内容发生变化才会与报文主体产生差异。

### 3.3.2. 压缩传输的内容编码

HTTP中的内容编码指明应用在实体内容上的编码格式，并保持实体信息原样压缩，内容编码后的实体由客户端接收并负责解码。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580019/article/tcpip/http/codestatus/3.3.2.png)

常用的内容编码：

- gzip(GNU ZIP)
- compress(UNIX系统的标准压缩)
- deflate(zlib)
- identity(不进行编码)

### 3.3.3. 分块传输编码

分块传输编码会将实体主体分成多个块，每一块都会用十六进制来标记快的大小，而实体的最后一块会使用“0（CR+LF）”来标记。

由接收的客户端负责解码，回复到编码前的实体主体。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580020/article/tcpip/http/codestatus/3.3.3.png)

## 3.4. 发送多种数据的多部分对象集合

HTTP中的多部分对象集合即发送一份报文主体内可含有多类型实体，通常是图片或文本文件上传等。

多部分对象集合包含的对象：

- multipart/form-data:在web表单文件上传时使用
- multipart/byteranges：状态码206响应报文包含了多个范围的内容时使用

## 3.5. 获取部分内容的范围请求

指定范围发送的请求叫做范围请求，对于一份10000字节大小的资源，如果使用范围请求，可以只请求5001-10000字节内的资源。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580020/article/tcpip/http/codestatus/3.5.png)

执行范围请求时，会用到首部字段Range来指定资源的byte范围

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580020/article/tcpip/http/codestatus/3.5.2.png)

## 3.6. 内容协商返回最合适的内容

内容协商机制是指客户端和服务端就响应的资源内容进行交涉，然后提供给客户端最合适的资源。内容协商会以响应资源的语言、编码方式等作为判断的基准。

内容协商类型：

- 服务器驱动协商
- 客户端驱动协商
- 透明协商

# 4. HTTP状态码

状态码即服务器返回的请求结果。

| 状态码 | 类型                        | 说明                       |
| ------ | --------------------------- | -------------------------- |
| 1xx    | Informational(信息性状态码) | 接收的请求正在处理         |
| 2xx    | Success(成功)               | 请求正常处理完毕           |
| 3xx    | Redirection(重定向)         | 需要进行附加操作以完成请求 |
| 4xx    | Client Error(客户端错误)    | 服务器无法处理请求         |
| 5xx    | Server Error(服务端错误)    | 服务器处理请求出错         |

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580022/article/tcpip/http/codestatus/4.png)

## 4.1. 2XX成功

### 4.1.1. 200 OK

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580020/article/tcpip/http/codestatus/4.1.1.png)

### 4.1.2. 204 No Content

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580020/article/tcpip/http/codestatus/4.1.2.png)

表示请求已成功处理，但在返回的响应报文中不含实体的主体部分。

### 4.1.3. 206 Partial Content

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580020/article/tcpip/http/codestatus/4.1.3.png)

该状态码表示客户端进行了范围请求，服务器成功执行了这部分的GET请求。响应报文中包含由Content-Range指定范围的实体内容。

## 4.2. 3XX 重定向

### 4.2.1. 301 Moved Permanently

永久性重定向，表示资源已被分配了新的URI，以后应使用新的URI。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580020/article/tcpip/http/codestatus/4.2.1.png)

### 4.2.2. 302 Found

临时性重定向，表示请求的资源已被分配了新的URI，但是临时性的。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580021/article/tcpip/http/codestatus/4.2.2.png)

### 4.2.3. 303 See Other

表示由于请求的资源存在另一个URI，应使用GET方法重定向获取请求的资源。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580020/article/tcpip/http/codestatus/4.2.3.png)

### 4.2.4. 304 Not Modified

表示客户端发送附带条件的请求时（GET中的If-Modified-Since等首部），服务器允许访问资源，但未满足附带条件因此直接返回304（服务器的资源未改变，可直接使用客户端未过期的缓存），不包含任何响应的主体部分。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580021/article/tcpip/http/codestatus/4.2.4.png)

### 4.2.5. 307 Temporary Redirect

临时重定向，该状态与302有相同的含义。

## 4.3. 4XX 客户端错误

### 4.3.1. 400 Bad Request

表示请求报文中存在语法错误，需修改内容重新发送请求。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580021/article/tcpip/http/codestatus/4.3.1.png)

### 4.3.2. 401 Unauthorized

表示需要通过HTTP认证。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580021/article/tcpip/http/codestatus/4.3.2.png)

### 4.3.3. 403 Forbidden

表示请求被服务器拒绝，未获得访问授权。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580021/article/tcpip/http/codestatus/4.3.3.png)

### 4.3.4. 404 No Found

表明服务器上找不到请求的资源，也可以在服务器拒绝请求且不想说明理由时使用。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580021/article/tcpip/http/codestatus/4.3.4.png)

## 4.4. 5XX 服务器错误

### 4.4.1. 500 Internal Server Error

表明服务器在执行请求时发生了错误，也可能是Web应用存在bug或临时故障等。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580021/article/tcpip/http/codestatus/4.4.1.png)

### 4.4.2. 503 Service Unavailable

表明服务器暂时处于超负荷或正在进行停机维护，现在不能处理请求。

![img](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510580022/article/tcpip/http/codestatus/4.4.2.png)



参考：

- 《图解HTTP》