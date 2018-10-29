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



参考：

- 《图解HTTP》