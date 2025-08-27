---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.3 找出大文件
order: 2
---

# python实战练手项目---找出电脑里遗忘的大文件

相信在你的电脑里有许多文件已经被遗忘在某个角落里很久了，那些特别大的文件，占用了磁盘空间，虽说硬盘越发便宜了，不影响使用，但编写一段程序将这些被遗忘的大文件找出来不是一件趣事么。

```python
def get_big_file(path, filesize):
    """
    找出path目录下文件大小大于filesize的文件
    :param path:
    :param filesize:
    :return:
    """
    pass
```

想要完成这个功能，有两个技术点要解决:

1. 遍历文件夹及其子文件夹，找出所有的文件目录
2. 获取文件的大小

遍历文件夹，可以使用os.walk方法，获取文件大小可以使用os.path.getsize方法，看来，技术点已经都解决了

```python
import os

def get_big_file(path, filesize):
    """
    找出path目录下文件大小大于filesize的文件
    :param path:
    :param filesize:
    :return:
    """
    # 遍历指定文件夹及其子文件夹
    for dirpath, dirnames, filenames in os.walk(path):
        for filename in filenames:
            target_file = os.path.join(dirpath, filename)
            # 要判断是否真的是文件,有可能是个链接哦
            if not os.path.isfile(target_file):
                continue
            size = os.path.getsize(target_file)
            if size > filesize:
                size = size//(1024*1024)    # 转换兆
                size = '{size}M'.format(size=size)
                print(target_file, size)

if __name__ == '__main__':
    get_big_file('/Users/kwsy', 500*1024*1024)
```

我在自己电脑上查找文件大小大于500M的文件，果真找出了一些

```text
/Users/kwsy/Downloads/[BD影视分享bd-film.cc]多哥.Togo.2019.HD720P.中英双字.人人.mp4 1609M
/Users/kwsy/Downloads/[niubaobao.cc]蝴蝶.Le.Papilion.2002.HDRip.国法双语.中字.mkv 547M
/Users/kwsy/Downloads/ml-20m/ratings.csv 508M
/Users/kwsy/Library/Caches/PyCharm50/caches/content.dat.storageData 509M
/Users/kwsy/Library/Containers/com.docker.docker/Data/vms/0/Docker.qcow2 20297M
```

好些我已经忘记了是什么时候下载的了，mac电脑空间很宝贵啊！
