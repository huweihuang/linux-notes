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