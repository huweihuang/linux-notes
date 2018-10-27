# 1. 基础知识

## 1.1. 协议

计算机与网络设备要相互通信，必须基于相同的方法。比如，如何探测到通信目标，使用哪种语言通信，如何结束通信等规则要事先确定。

不同硬件，操作系统之间的通信都需要一种规则，我们将这种`事先约定好的规则称之为协议`。

## 1.2. 地址

地址：在某一范围内确认的唯一标识符，即数据包传到某一个范围，需要有一个明确唯一的目标地址。

| 类型    | 层         | 地址     | 说明                               |
| ------- | ---------- | -------- | ---------------------------------- |
| 端口号  | 传输层     | 程序地址 | 同一个计算机中不同的应用程序       |
| IP地址  | 网络层     | 主机地址 | 识别TCP/IP网络中不同的主机或路由器 |
| MAC地址 | 数据链路层 | 物理地址 | 在同一个数据链路中识别不同的计算机 |

## 1.3. 网络构成

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579692/article/tcpip/basics/network.png" width="80%">

| 构成要素         | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| 网卡             | 连入网络必须使用网卡，又称网络接口卡。                       |
| 中继器           | OSI第1层，物理层上延长网络的设备，将电缆的信号放大传给另一个电缆。 |
| 网桥/2层交换机   | OSI第2层，数据链路层面上连接两个网络的设备，识别数据帧的内容并转发给相邻的网段，根据MAC地址进行处理。 |
| 路由器/3层交换机 | OSI第3层，网络层面连接两个网络并对分组报文进行转发，根据IP进行处理。 |
| 4-7层交换机      | 传输层到应用层，以TCP等协议分析收发数据，负载均衡器就是其中一种。 |
| 网关             | 对传输层到应用层的数据进行转换和转发的设备，通常会使用表示层或应用层的网关来处理不同协议之间的翻译和通信，代理服务器（proxy）就是应用网关的一种。 |

# 2. OSI与TCP/IP参考模型

## 2.1. OSI与TCP/IP参考模型图

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579693/article/tcpip/basics/TCPIP-OSI.png" width=80%>

## 2.2. OSI参考模型分层说明

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579692/article/tcpip/basics/osi-function.png" width=80%>

## 2.3. OSI参考模型通信过程

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579692/article/tcpip/basics/OSI.png" width=80%>


1、打包数据时，每一层在处理上一层传过来的数据时，会在数据上附上当前层的首部信息后传给下一层；

2、解包数据时，每一层在处理下一层传过来的数据时，会将当前层的首部信息与数据分开，将数据传给上一层。

3、数据通信过程

| 分层       | 每层的操作                                                   |
| ---------- | ------------------------------------------------------------ |
| 应用层     | 在数据前面加首部，首部包括数据内容、源地址和目标地址，同时也会处理异常的反馈信息。 |
| 表示层     | 将特有的数据格式转换为通用的数据格式，同时也会加上表示层的首部信息以供解析。 |
| 会话层     | 对何时连接，以何种方式连接，连接多久，何时断开等做记录。同时也会加会话层的首部信息。 |
| 传输层     | 建立连接，断开连接，确认数据是否发送成功和执行失败重发任务。 |
| 网络层     | 负责将数据发到目标地址，也包含首部信息。                     |
| 数据链路层 | 通过物理的传输介质实现数据的传输。                           |
| 物理层     | 将0/1转换成物理的传输介质，通过MAC地址进行传输。             |

## 2.4. TCP/IP应用层协议

### 2.4.1. 通信模型

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579692/article/tcpip/basics/TrafficModel.png" width=100%>

### 2.4.2. 应用层协议说明

| 应用类型 | 协议       | 协议说明                                                     |
| -------- | ---------- | ------------------------------------------------------------ |
| WWW      | HTTP,HTML  | <img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579693/article/tcpip/basics/www.png" width=100%> |
| 电子邮件 | SMTP，MIME | <img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579691/article/tcpip/basics/email.png" width=100%> |
| 文件传输 | FTP        | <img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579691/article/tcpip/basics/ftp.png" width=100%> |
| 远程登录 | TELNET,SSH | <img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579693/article/tcpip/basics/telnet.png" width=100%> |
| 网络管理 | SNMP,MIB   | <img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579692/article/tcpip/basics/snmp.png" width=100%> |

# 3. TCP/IP通信过程

## 3.1. 数据包结构

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579691/article/tcpip/basics/data-package.png" width=100%>

## 3.2. 数据打包和解包过程

### 3.2.1. 包的封装

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579692/article/tcpip/basics/package.png" width=80%> 

### 3.2.2. 发送与接收

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579693/article/tcpip/basics/send-receive.png" width=70%>

## 3.3. 数据包传输过程

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1510579693/article/tcpip/basics/transmission.png" width=80%>



文章：

- 《图解TCP/IP》