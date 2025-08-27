---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 3.8 垃圾回收机制
order: 7
---

# 深度讲解python垃圾回收机制

python的垃圾回收机制，以引用计数为主，标记清除和分代回收为辅。
![python垃圾回收机制](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091928466.jpeg)

## 1. 引用计数

关于引用计数，推荐我的一个视频教程, 地址为 https://www.bilibili.com/video/av60473350

本文先做一个简单的回顾

python对象的核心则是一个结构体

```c
typedef struct_object {
 int ob_refcnt;
 struct_typeobject *ob_type;
} PyObject;
```

不懂结构体没有关系，你只需要知道ob_refcnt记录了对象的引用次数就好了。当一个对象有新的引用时，ob_refcnt加1，引用它的对象被删除时，ob_refcnt减1，当ob_refcnt=0时，这个对象的生命就结束了,这时，垃圾回收机制就会启动将这个对象回收。

```python
class Pyobj:
    def __del__(self):
        print("对象被销毁")

print("1")
obj = Pyobj()
obj = 6     # 让变量obj指向其他对象
print("2")
```

当obj = 6 这行被执行时，__del__方法会被执行，print("对象被销毁") 会先于print("2") 执行。

程序输出结果

```text
1
对象被销毁
2
```

理论与实践完美结合，下图从内存层面上展示了这一过程
![python对象被垃圾回收](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091930220.jpeg)

### 引用计数的优点

- 简单
- 实时性高，只要引用计数为0，对象就会被销毁，内存被释放，回收内存的的时间平摊到了平时

### 引用计数的缺点

- 为了维护引用计数消耗了很多资源
- 循环引用，循环引用导致内存泄漏，例如下面的代码

下面是一个循环引用的例子

```python
list1 = []
list2 = []
list1.append(list2)
list2.append(list1)
```

list1 和 list2的引用计数永远大于0，除非手动操作，他们不可能被GC回收，但如果你手动将其释放回收，那么GC机制岂不是形同虚设？针对这种情况，python引入了标记清除和分代回收机制作为补充。

## 2. 标记清除

引用计数，并不能解决所有的问题，一旦出现了循环引用，那么，这些对象的引用次数永远都是大于0的，但是这些对象都是不可用的垃圾数据。下面的代码展示了一种循环引用的情况。

```python
import gc


class DictA(dict):
    def __del__(self):
        print('DictA对象被销毁')


class DictB(dict):
    def __del__(self):
        print('DictB对象被销毁')
```

为了便于观察循环引用导致的内存泄漏问题，我定义了两个类，DictA,和DictB两个类，他们均继承了字典类。

下面的代码，先为你演示没有循环引用的情况

```python
a = DictA()
b = DictB()

a = 1
b = 1

print('ok')
```

下图是内存层面的对象示意图。
![python循环引用](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091941195.jpeg)

a = 1 这样代码被执行时，变量a指向了内存中的1， 那么原来所指向的那字典对象的引用计数就变成了0，因此对象被销毁，程序运行的结果是是

```text
DictA对象被销毁
DictB对象被销毁
ok
```

注意，ok是最后被输出的

下面的代码，演示有循环引用的情况

```python
a = DictA()
b = DictB()

a['b'] = b      # 循环引用
b['a'] = a

a = 1
b = 1

print('ok')
```

由于存在循环引用，因此，内存中DictA对象的引用计数是2，当a = 1被执行时，引用计数减少为1，但仍然大于0，不会被回收，DictB的对象同样如此，下图是存在循环引用时的内存对象示意图
![python循环引用](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091948960.jpeg)

程序的执行结果为

```text
ok
DictA对象被销毁
DictB对象被销毁
```

对象销毁的信息是在print('ok')以后才被输出的，这说明，当a = 1被执行时，原来a所指向的那个字典对象并没有被销毁。

### 标记清除的原理

标记清除可以处理这种循环引用的情况，它分为两个阶段

#### 第1阶段，标记阶段

GC会把所有活动对象打上标记，这些活动的对象就如同一个点，他们之间的引用关系构成边，最终点个边构成了一个有向图，如下图所示
![python标记清除垃圾](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827091929612.jpeg)

#### 第2阶段，搜索清除阶段

从根对象（root）出发，沿着有向边遍历整个图，不可达的对象就是需要清理的垃圾对象。这个根对象就是全局对象，调用栈，寄存器。

在上图中，从root出发后，可以到达 1 2 3 4，而5， 6， 7均不能到达，其中6和7互相引用，这3个对象都会被回收。

## 3. 分代回收

分代回收建立标记清除的基础之上，是一种以空间换时间的操作方式。标记清除可以回收循环引用的垃圾，但是，回收的频次是需要控制的，如果时时刻刻做标记清除，可以想象，python的程序会慢成什么样子。

分代回收，根据内存中对象的存活时间将他们分为3代，新生的对象放入到0代，如果一个对象能在第0代的垃圾回收过程中存活下来，GC就会将其放入到1代中，如果1代里的对象在第1代的垃圾回收过程中存活下来，则会进入到2代。

### 分代回收的触发机制

```python
import gc

print(gc.get_threshold())
```

上面的代码执行结果是(700, 10, 10)

- 当分配对象的个数减去释放对象的个数的差值大于700时，就会产生一次0代回收
- 10次0代回收会导致一次1代回收
- 10次1代回收会导致一次2代回收

对于第0代的对象来说，他们很可能就被使用一次，因此需要经常被回收。

经过一轮一轮的回收后，能够活着成为第2代的对象，必然是那些使用频繁的对象，而且他们已经存活很久的时间了，大概率的，还会存活很久，因此，2代回收的就不那么频繁，

你可以通过设置这三个阈值，来改变分代回收的触发条件

```python
import gc

gc.set_threshold(600, 10, 5)
print(gc.get_threshold())
```

经过了上面的设置，0代和2代的回收会更加频繁
