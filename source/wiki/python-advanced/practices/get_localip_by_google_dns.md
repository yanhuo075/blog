---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.13 利用dns服务获取本地ip地址
order: 12
---

# python实战练手项目---利用谷歌dns服务器获取本地ip地址

## 1. 使用命令查看ip地址

不论是在公司还是在家里，我们的电脑都有一个局域网地址，在windows电脑上，你可以在cmd窗口里执行ipconfig 来查看你的本地ip地址，此外还可以查看子网掩码，在mac和linux系统下，可以使用ifconfig命令来查看，ifconfig命令可以显示网络设备。

既然，使用命令就可以查看本地ip地址，那么研究如何使用python来获取本地ip地址还有什么意义么？当然是有意义的，计算机，或者说编程的世界，是需要探索的，你知道的越多，就越能得心应手的使用一门编程语言来解决实际问题。而你努力的方向，却不可以是直接奔向问题本身，原因在于问题总是在变化，因此我们需要丰富的知识来应对变化，如果你能耐心的阅读完本文，我保证你会受益良多。

我并不想利用ifconfig命令来获取本地ip地址，因为这个命令会显示非常多的信息，从这么多信息中找到本地ip地址也不是一件容易的事情，篇幅有限，我只粘贴的部分。

```text
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
    options=1203<RXCSUM,TXCSUM,TXSTATUS,SW_TIMESTAMP>
    inet 127.0.0.1 netmask 0xff000000
    inet6 ::1 prefixlen 128
    inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1
    nd6 options=201<PERFORMNUD,DAD>
gif0: flags=8010<POINTOPOINT,MULTICAST> mtu 1280
stf0: flags=0<> mtu 1280
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
    ether a4:d1:8c:e3:eb:82
    inet6 fe80::8fd:6c23:5c32:c0da%en0 prefixlen 64 secured scopeid 0x4
    inet 192.168.1.108 netmask 0xffffff00 broadcast 192.168.1.255
    nd6 options=201<PERFORMNUD,DAD>
    media: autoselect
    status: active
```

我电脑显示的比较乱，大概是由于安装了docker

## 2. 利用谷歌dns服务器

这个思路概括起来很简单，在本地创建一个socket客户端，连接谷歌的dns服务器，建立连接后，获得客户端套接字自己的地址，这样的实现，既简单又可靠。

谷歌的dns服务器地址是8.8.8.8，开放端口是80。 说到dns服务器，有一个特别有意思的事情，你或者身边的朋友，或许曾经遇到过这样的问题，电脑不能上网浏览网页，但却可以登录QQ，这其实就是你电脑的dns设置出了问题，在dns设置里增加8.8.8.8，问题立马解决，关于dns的作用不是本文重点，不再赘述。

建立socket连接，客户端必须提供ip地址和端口号，这完全是对等的，服务端也提供了ip地址和端口号，一旦建立连接，就可以通过getsockname()方法来获得套接字自己的ip和端口

## 3. 示例代码

```python
import socket


def get_local_ip():
    client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    # 连接谷歌的dns服务器
    client.connect(("8.8.8.8", 80))
    ip, _ = client.getsockname()   # 获取套接字自己的地址,返回元组,ip地址和端口号
    client.close()
    return ip

if __name__ == '__main__':
    print(get_local_ip())
```
