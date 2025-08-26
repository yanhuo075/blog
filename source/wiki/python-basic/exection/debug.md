---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 10.7 使用pycharm调试python程序
order: 6
---

# 如何使用pycharm调试python程序

使用pycharm的调试功能，可以对程序进行调试，也就debug。 调试功能主要包括设置断点, 单步执行, 查看调用栈信息, 查看变量的值，调试功能让程序在某一行代码上暂停，通过对这一刻的上下文环境进行检查，可以定位程序出问题的原因。

调试程序是程序员的日常，再牛逼的人，也不可能一次性把程序写对，因此，写完一个函数或者一段代码后，你需要测试自己的代码，如果有问题，就必须进行调试，也就是debug。

对于初学者，debug更有意义,通过debug你可以了解程序的执行过程，如何进行循环，为何break,if条件判断是否成立，运行过程中的每一个变量值如何变化，可以非常确定的讲，如果你不会debug，那么，永远都做不好编程。

## 1. 断点

只有在debug状态下，断点才会起作用，所谓断点，就是一处你希望程序暂停的地方。我们希望程序不要那么快的执行完，设置一个断点，程序运行到这里，就会停下来，之后，你就可以单步调试，所谓单步调试，就是你让程序执行一行，它就执行一行。

断点的设置非常容易，以pycharm为例，只需要左键点击代码左侧标识代码行数的区域即可
![pycharm设置断点](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234353763.jpeg)
左键点击红框内的区域，就会出现一个小红点，再次点击，小红点会消失，这就是一处断点

## 2. debug模式运行

想要单步调试程序，需要进入debug模式运行程序

第一种方法，通过Run 找出下拉菜单，然后点击debug
![启动pycharm的debug](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234355366.jpeg)
第二种方法，右键点击代码区域，点击debug
![启动pycharm的debug](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234418187.jpeg)

## 3. debug工具使用

![pycharm的debug界面](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234422064.jpeg)
上图就是进入到debug模式后的界面，有三个重要区域需要你关注

### 3.1 调试区域

![pycharm调试区域](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234354362.jpeg)

常用的功能键有5个

第1个功能键是 step over,你点击一下，程序就会执行一行

第2个功能键是 step into,如果这一行有函数，点击它就会进入到函数中

第3个功能键是 step into my code，如果这一行有函数，既有第三方库的函数，也有自己编写的函数，那么会优先进入到自己的函数里

第4个功能键是step out，就是从函数里退出来

第5个功能键是run to cursor,就是直接运行到下一处断点

## 3.2 调用栈区域

![pycharm调用栈区域](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234354931.jpeg)

先看左侧的两个红框内的按钮

绿色箭头的按钮是resume progrom,是恢复程序执行，如果后续代码没有断点了，程序就会正常执行直到结束。

红方块的按钮是 stop，停止程序。

调用栈区域展示的，是调用栈的信息，通过它就能知道从程序开始运行到当前代码，经过了哪些调用关系

## 3.3 变量查看区域

![pycharm变量查看区域](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250823234512138.jpeg)

变量查看区域，可以查看当前程序里的每一个变量的值，非常的方便，不仅如此，还可以自己添加表达式，查看自己关注的值，比如绿色框内的1%2 就是我通过左侧红色框内的+按钮添加上去的，添加的时候，写完表达式后，要点击回车才能完成添加。

除了这里可以查看变量的值，在程序页面里，debug模式下，鼠标滑动到变量上，也可以查看变量的值

除了pycharm提供的调试工具，还可以使用python提供的pdb，不过是命令行的，不如图形界面友好，还有一个ipdb，比pdb高级一些，有高亮显示，学会了debug，程序的运行结果和自己期望的不一致时，就可以单步跟踪调试啦，再也不用四处求人解答了
