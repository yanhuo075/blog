---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.6 下载文件并添加进度条
order: 5
---

# python实战练手项目---下载文件并添加进度条

本文研究讨论如何使用python进行文件下载

## 1. 选择下载工具

下载文件，本质上是一个发送http请求然后接受服务器响应数据的过程。python标准库里提供了urllib3这个可以进行网络编程的标准库，但其提供的接口并不友好，至少不如requests库，因此，我首先推荐你使用requests进行网络编程

### 1.1 使用requests进行文件下载

```python
import requests
url = 'https://www.python.org/static/img/python-logo@2x.png'
res = requests.get(url)

with open('pythonimage.png', 'wb') as f:
    f.write(res.content)
```

发送get请求，图片数据保存在res.content中，打开文件并写入文件就完成了下载

### 1.2 使用wget模块下载

wget需要先安装

```text
pip3 install wget
```

它的使用也十分方便

```python
import wget
import ssl

# 取消ssl全局验证
ssl._create_default_https_context = ssl._create_unverified_context

url = 'https://www.python.org/static/img/python-logo@2x.png'
wget.download(url, 'pythonlogo.png')
```

## 2. 下载大文件

下载大文件时，需要考虑内存问题，使用requests.get方法，默认会立即下载文件内容并保存到内存中，如果文件很大，会给内存造成压力，因此我们需要设置stream参数为True，这样，只有当我们遍历iter_content时才会进行数据下载

```python
import requests

url = 'https://www.python.org/ftp/python/3.8.1/python-3.8.1-macosx10.9.pkg'
res = requests.get(url, stream=True)

print(res.status_code, res.headers)

with open("py.pkg", "wb") as pypkg:
    for chunk in res.iter_content(chunk_size=1024):
        if chunk:
            pypkg.write(chunk)
```

使用这种方式下载还有一个好处，由于文件内容是分块下载的，因此，可以使用进度条来观察下载的进度，本文使用clint模块来显示下载进度，你需要先安装

```python
import requests
from clint.textui import progress

url = 'https://www.python.org/ftp/python/3.8.1/python-3.8.1-macosx10.9.pkg'
res = requests.get(url, stream=True)
total_length = int(res.headers.get('content-length'))

with open("py.pkg", "wb") as pypkg:
    for chunk in progress.bar(res.iter_content(chunk_size=1024), expected_size=(total_length/1024) + 1, width=100):
        if chunk:
            pypkg.write(chunk)
```

一次读取1024个字节，总文件大小是29051411，共需要读取28371次，随着读取的进行，进度条也会发生变化
![python进度条](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827104449485.jpeg)
