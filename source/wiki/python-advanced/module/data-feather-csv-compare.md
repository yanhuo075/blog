---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 4.4 数据处理
order: 3
---

# 数据处理

## feather，一种比csv更快，体积更小的数据格式

还在用csv这种格式存储数据么？有一种比csv更快，生成文件体积更小的数据格式---feather。
feather 是一种用于存储数据帧的数据格式，它最初是为python和R之间更快速通信而设计的，它尽可能快的将数据帧从内存中读取出来或是写入内存。

生成csv文件，唯一的好处是可以打开文件查看其中的数据，但如果你没有这方面的需要而且数据量比较大，那么你应当抛弃csv转而使用feather，使用pip安装

```text
pip install feather-format
```

下面的例子将充分体现feather的优势

```python
import time
import feather
import numpy as np
import pandas as pd

np.random.seed = 42
df_size = 10_000_000

df = pd.DataFrame({
    'a': np.random.rand(df_size),
    'b': np.random.rand(df_size),
    'c': np.random.rand(df_size),
    'd': np.random.rand(df_size),
    'e': np.random.rand(df_size)
})

t1 = time.time()
df.to_feather('1M.feather')
t2 = time.time()
df.to_csv('to.csv')
t3 = time.time()

print(f'保存feather耗时{t2-t1}')
print(f'保存csv耗时{t3-t2}')
```

代码里使用DataFrame的to_feather方法保存数据，在方法里面用到了feather模块，你也可以直接使用feather保存数据

```text
feather.write_dataframe(df, '1M.feather')
```

现在让我们来比较一下保存文件时的耗时时间

```text
保存feather耗时3.7854344844818115
保存csv耗时119.42699456214905
```

这看起来有点夸张，足足有30倍的差距，再来看看生成的文件大小，1M.feather 的大小是381M，而to.csv文件的大小是993M，相差了足足2.6倍。

更小的体积，更快的写入速度，最后来比较一下读取文件的速度

```python
t1 = time.time()
pd.read_feather('1M.feather')
t2 = time.time()
pd.read_csv('to.csv')
t3 = time.time()
```

两种文件的读取速度如下

```text
读取feather耗时0.37110209465026855
读取csv耗时8.036199569702148
```

读取速度相差了20倍，feather完全碾压csv。

为什么feather会这么快呢，简单说，它是一种二进制数据格式，代码里生成的dataframe占用内存可以通过df.info()来查看，是381.5M，生成的feather文件是381M，feather将内存中的数据几乎是原封不动的写入到文件中，因而获得了超高的写入效率，且文件体积几乎无法变得更小。

看到这么牛叉的库，动心了吧，在处理较大数据量时，feather绝对是一把利器，节省时间，节省硬盘。
