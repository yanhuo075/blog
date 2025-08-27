---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.12 获取可用wifi信息
order: 11
---

# python实战练手项目---获取可用wifi信息

点击电脑的网络设置，总能看到附近可用的wifi，那么能否用python写程序获取这些可用wifi网络信息呢？

某些人看到这里已经开始质疑做这件事情的意义了，觉得这样的技术在工作中永远不会使用，不值得学习。对于这样的观点，我不认同。对于技术，我们应始终保持好奇心，始终保持很强的探索欲望，除了通过点击查看网络设置，你不想知道还有别的方式来获取可用wifi网络么？

在windows系统上，打开cmd命令窗口，执行命令

```text
netsh wlan show network
```

这个命令就可以查看计算机可以连接的wifi网络
![netsh wlan show network](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827104938978.png)

有了查看wifi的命令，就可以使用python的subprocess模块来执行这个命令

示例代码

```python
import subprocess

result = subprocess.check_output(['netsh', 'wlan', 'show', 'network'])
result = result.decode('gbk')
lst = result.split('\r\n')
lst = lst[4:]

for index in range(len(lst)):
    if index % 5 == 0:
        print(lst[index])
```

check_output 方法会在子进程中执行netsh wlan show network 命令，其结果将以字符串的形式返回，类型为bytes，使用decode方法将数据转成str，这里要注意编码，我电脑里执行代码时用的是gbk，如果你实验时在这里报错，可以改成ascii或者utf-8试试。

为什么要用\r\n做分割呢，因为返回的字符串，每一个段落末尾都是\r\n，这样才能保证一个字符串在屏幕上显示时可以分行，不然就只能显示在一行上了。

至于for循环里对5取模，是因为lst列表里，索引是5的倍数的元素恰好就是wifi的名称，这是前面使用split分割导致的，具体的，你可以输出lst的内容查看，便能够明白了。

虽然只是一个小小的功能，但你通过这个不起眼的功能学会了使用subprocess.check_output，那么今后，需要在python当中使用命令来获取信息或进行操作时，你都可以使用这个方法，除了check_output，subprocess还有很多其他利害的方法等待你去探索
