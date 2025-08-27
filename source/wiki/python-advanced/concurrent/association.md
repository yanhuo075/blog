---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 11.4 协程
order: 3
---

# 理解python的协程

尽管文章标题是《理解python的协程》，但协程并非python专属的概念，而是一个计算机概念，不同编程语言有不同的实现。本文将带你深入理解协程的概念，并使用python的生成器实现简单的协程。

## 1. 子例程

子例程是主程序的一部分代码，它执行特定的功能并与主程序的其他代码保持独立。对于这个概念，你肯定感到困惑，别急，你最熟悉的函数就是子例程。函数时主程序的一部分，函数执行特定的功能，而且与其他代码保持独立。

我下介绍子例程，为的是介绍协程时将他们两个进行比较，以便可以更好的理解协程。

## 2. yield

如果一个python函数里使用yield， 那么这个函数就是一个生成器函数，调用生成器函数可以获得一个生成器，从概念上讲，生成器就是一个协程。

yield 应当取其 ‘“让步” 而非“产出” 或 “返回” 之意，这是理解协程的关键。下面是一段演示生成器使用的代码

```python
def my_generator(n):
    index = 0
    while index < n:
        yield index
        index += 1


gen_one = my_generator(5)
gen_two = my_generator(5)

# 执行协程 gen_one
value = next(gen_one)
print(value, 'gen_one')

# 执行协程 gen_two
value = next(gen_two)
print(value, 'gen_two')

# 执行协程 gen_one
value = next(gen_one)
print(value, 'gen_one')

# 执行协程 gen_two
value = next(gen_two)
print(value, 'gen_two')
```

程序运行结果为

```text
0 gen_one
0 gen_two
1 gen_one
1 gen_two
```

yield 有让步之意，因为它交出了程序的控制权，但这个协程并没有结束，下一次执行时，将恢复到之前让出程序控制权的地方，也就是yield语句执行的地方继续执行。在上面的代码里，我创建了两个生成器，也就是创建了两个协程，我编写了8行代码，让这两个协程交替执行，想象一下，my_generator函数中，yield 语句执行的地方不是紧跟着一个整数，而是一个IO操作，那么这就会大大提高程序的并发。IO操作是阻塞的，耗时的，但协程可以在遇到IO操作的时候将程序的控制权让出，这个时候别的协程获得程序控制权继续执行。

现在请思考，函数具备这样的功能么？函数或者说子例程，将程序控制权交出去，其本质不就是函数运行结束么？不论如何也不会在中间某个位置交出控制权，过一段时间又回到这个位置继续执行，而协程就可以。

上面的代码，始终是在同一个线程内执行，这是很多人都会忽视的一个关键事实。多个协程在一个线程内协作，在协程间的切换不涉及到任何系统调用，也不会产生阻塞。

## 3. python 实现协程

尽管我在前面说从概念上讲，生成器就是一个协程，但严格的讲，生成器是协程的子集。生成器在交出控制权后，并不能决定由哪个协程接替自己继续运行，生成器只能将控制权交给自己的调用者。我们可以在生成器的基础上，实现简单的协程，这需要我们实现一个顶层的负责在协程之间调度的派遣器，当一个协程让出控制权后，这个派遣器可以协调另一个协程来接管控制权。

我想定义两个函数，在函数执行过程中，分别sleep 3秒钟，如果不使用协程技术，那么这两个函数执行所需要的时间是6秒钟，但如果使用协程技术，只需要3秒钟就可以。你可能想到了用yield time.sleep(3) 的方式，但这样是行不通的，因为yield 会等待sleep函数执行结束后才能让出控制权。我需要实现一个既能实现sleep 效果同时又能立即返回的对象。

```python
import time

event_list = list()

class CoroutinueSleep():
    def __init__(self, sleep_time):
        event_list.append(self)
        self.sleep_time = sleep_time
        self.call_back = None
        self.start_time = time.time()

    def is_ready(self):
        ready = self._is_ready()
        if ready:
            self.call_back()    # 执行回调函数

        return ready

    def _is_ready(self):
        return (time.time() - self.start_time) > self.sleep_time

    def set_callback(self, callback):
        self.call_back = callback
```

如果我创建了一个CoroutinueSleep 对象，那么这个对象根本没有sleep的效果，但是这个类实现了一个is_ready方法，如果有另一段程序（派遣器）可以不停的调用is_ready方法，就可以实现了sleep的效果了，下面是两个函数，函数内使用CoroutinueSleep类希望获得time.sleep()一样的效果

```python
def func_a():
    print("a函数开始执行: ", time.time())
    yield CoroutinueSleep(1)
    yield CoroutinueSleep(1)
    yield CoroutinueSleep(1)
    print('a函数执行结束: ', time.time())

def func_b():
    print("b函数开始执行: ", time.time())
    yield CoroutinueSleep(3)
    print('b函数执行结束: ', time.time())
```

每次执行到yield 时，就会交出控制权，那么派遣器就可以通过调用is_ready方法来判断是否已经休眠了足够的时间，当满足休眠时间后，重新将控制权交还给func_a 和 func_b， 做到这一点，只需要再次调用next函数即可，生成器将恢复到上一次执行yeild的地方，也就是交出控制权的地方，下面是派遣器的实现

```python
class CoroutinueManager():
    def __init__(self, task_lst):
        self.task_lst = task_lst
        self._start()

    def _start(self):
        for task in self.task_lst:
            self._next(task)

    def _next(self, task):
        try:
            gen = next(task)
            gen.set_callback(lambda : self._next(task))   # 设置CoroutinueSleep对象的回调函数
        except StopIteration:
            pass

    def polling(self):
        while len(event_list):
            for event in event_list:
                if event.is_ready():
                    event_list.remove(event)
                    break   # 使用了remove ，这一轮循环就不能再继续了

            time.sleep(0.01)
```

这个类的核心在于polling 方法，每次创建CoroutinueSleep对象时，都会将CoroutinueSleep对象追加到event_list，而polling方法遍历event_list， 如果is_ready方法返回True， 那么就将CoroutinueSleep对象移除，因为它已经执行结束，注意，不是函数func_a或者func_b执行结束，而是一次CoroutinueSleep 休眠结束。

CoroutinueSleep对象 在结束时，会调用call_back， 这个这个call_back 是在CoroutinueManager 的_next方法里设置的，再次对对task 执行next函数。这里的task 是 func_a() 和 func_b() 返回的生成器， 如果生成器里没有可以继续执行的yield ，则抛出StopIteration异常，与普通函数不同，虽然生成器函数抛出了StopIteration 异常，但由于程序做了捕获，yield剩余的部分代码仍将执行。如果生成器函数里还有yield，就像func_a 中那样，则会返回所创建的CoroutinueSleep对象，并为它设置callback， 这个新返回的对象已经加入到event_list中。

最后来测试上述代码

```python
t1 = time.time()

cm = CoroutinueManager([func_a(), func_b()])
cm.polling()

t2 = time.time()
print('总耗时: ', t2-t1)
```

程序输出结果为

```text
a函数开始执行:  1592991246.2696626
b函数开始执行:  1592991246.2696626
b函数执行结束:  1592991249.2790859
a函数执行结束:  1592991249.2897615
总耗时:  3.0310726165771484
```

从输出结果上看，每个函数都执行了3秒钟的时间，但总耗时也是3秒，两个函数是并发执行的。python内置的asyncio，tornado 的协程，在实现上更加复杂，但本质上原理是相同的，遇到有IO操作时，都必须挂起并交出控制权，让其他的协程来执行，存在一个派遣器来负责调度，当挂起的协程的IO操作结束后，这个协程可以被调度，恢复到执行状态。
