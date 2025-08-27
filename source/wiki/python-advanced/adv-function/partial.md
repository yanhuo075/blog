---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 5.3 partial
order: 2
---

# partial --- 偏函数

## 1. 偏函数

偏函数partial是functools 模块里提供的一个函数

和装饰器对比来理解，装饰器改变了一个函数的行为，而偏函数不能改变一个函数的行为。偏函数只能根据已有的函数生成一个新的函数，这个新的函数完成已有函数相同的功能，但是，这个新的函数的部分参数已被偏函数确定下来

## 2. 场景示例

### 2.1 常规实现

为了便于理解，我们构造一个使用场景，假设我们的程序要在dest目录下新建一些文件夹，那么常见的实现功能代码如下

```python
import os
from os import mkdir


mkdir(os.path.join('./dest', 'dir1'))
mkdir(os.path.join('./dest', 'dir2'))
mkdir(os.path.join('./dest', 'dir3'))
```

功能很简单，代码很简洁，但是有个小小的不如意之处，每次都是在dest目录下新建文件夹，既然它这么固定，是不是可以不用传递dest参数呢？

## 2.2 偏函数实现

```python
import os
from os import mkdir
from functools import partial


dest_join = partial(os.path.join, './dest')

mkdir(dest_join('dir1'))
mkdir(dest_join('dir2'))
mkdir(dest_join('dir3'))
```

dest_join是partial创建出来的一个新的函数，这个函数在执行时会调用os.path.join并将'./dest'作为参数传给join

偏函数将预先确定的参数冻结，起到了缓存的作用，在获得了剩余的参数后，连同之前冻结的确定参数一同传给最终的处理函数。

虽然看起来代码没有减少，但如果你实际使用就会发现，用了偏函数，免去了反复输入相同参数的麻烦，尤其当这些参数很多很难记忆时
