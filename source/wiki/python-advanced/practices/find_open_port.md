---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.10 探测主机开放的端口
order: 9
---

# python实战练手项目---使用socket探测主机开放的端口

## 1. 探测思路

一台主机，可以开放的端口范围是从0~65535，这个范围，是由TCP/IP协议决定的，在该协议中，TCP的头结构如下
![tcp头结构](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827104827139.jpeg)

在基于IPv4的网络中，源端口号和目标端口号都是16位的，因此最大只能是65535。

端口0在网络编程中有着特殊作用，尤其在unix系统中，如果你申请打开0端口号，0更像是一个统配符，系统会寻找一个合适的端口供你使用，而不是按照你的要求打开端口0。

在TCP/IP 协议中，0端口号是保留的，在TCP和UDP中都不应该使用，因此探测范围是1到65535。

从端口1到65535， 逐个使用socket尝试建立连接，根据建立连接的情况，就可以判断一个端口号是否开放了。

## 2. 探测代码

```python
import socket


for port in range(1, 65535):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(1) 
    state = sock.connect_ex(("192.168.0.1", port))
    if 0 == state:
        print("port: {} is open".format(port))
    sock.close()
```

## socket函数

socket函数返回一个socket句柄，该函数有两个重要的参数，分别是family和type,family指定网络类型，type指定socket类型，下表是这两个参数的可选项说明

| 参数            | 可选值                     | 说明                               |
| --------------- | -------------------------- | ---------------------------------- |
| family          | socket.AF_UNIX             | UNIX 网络                          |
| socket.AF_INET  | 基于 IPv4 协议的网络       |                                    |
| socket.AF_INET6 | 基于 IPv6 协议的网络       |                                    |
| type            | SOCK_STREAM                | 默认值，创建基于 TCP 协议的 socket |
| SOCK_DGRAM      | 创建基于 UDP 协议的 socket |                                    |
| SOCK_RAW        | 创建原始 socket            |                                    |

## 设置超时

毕竟是6万多个端口号需要探测，因此使用sock.settimeout(1) 来设置超时时间，超过1秒钟还不能建立连接，就放弃这个端口号。

## connect_ex

connect_ex 函数在遇到C层面的异常时不会抛出异常，而是返回状态码，0状态码表示正常，你也可以使用connect方法进行连接，但这样，就需要使用异常捕获机制来捕获ConnectionRefusedError 异常。

针对我自己的阿里云服务器进行了探测，效果还是挺好的

```text
port: 22 is open
port: 80 is open
port: 3306 is open
port: 8888 is open
port: 27017 is open
```
